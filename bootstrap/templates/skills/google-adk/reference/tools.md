# ADK Tools

Use this reference for understanding and using the built-in tools ecosystem in Google's Agent Development Kit. This covers tool categories, built-in integrations, third-party connectors, and practical usage patterns.

## When to Use This Reference

- Adding built-in tools (Google Search, Code Execution) to agents
- Integrating Google Cloud services (BigQuery, Vertex AI Search, RAG)
- Connecting third-party tools (databases, APIs, vector stores)
- Choosing the right tool type for a use case
- Understanding tool configuration and authentication patterns
- Debugging tool-related issues

## Tool Categories Overview

ADK organizes tools into four categories:

### 1. Function Tools (Custom)
Plain Python/TypeScript functions you write. The LLM calls them based on their name, docstring, and parameter types.

### 2. Gemini API Tools
Built-in capabilities provided by the Gemini model itself:
- **Google Search** - Grounded web search
- **Code Execution** - Run Python code in a sandbox

### 3. Google Cloud Tools
Managed Google Cloud service integrations:
- **Vertex AI Search** - Enterprise search over your data
- **BigQuery** - SQL queries against data warehouses
- **RAG (Retrieval Augmented Generation)** - Vector-based retrieval from Vertex AI

### 4. Third-Party Tools
Community and partner integrations:
- **Vector databases** (Qdrant, Pinecone, Weaviate)
- **Document databases** (MongoDB Atlas)
- **Developer tools** (GitHub, Jira)
- **API platforms** (LangChain tool adapters, CrewAI tool adapters)

## Function Tools (Custom Tools)

Before covering built-in tools, understand how custom function tools work since they are the most common type.

### Python Function Tools

```python
from google.adk.agents import Agent

def get_weather(city: str) -> dict:
    """Retrieves current weather for a specified city.

    Args:
        city: The name of the city (e.g., "San Francisco")

    Returns:
        dict with temperature, condition, and humidity
    """
    # Your implementation here
    return {"temp": "72F", "condition": "sunny", "humidity": "45%"}

def search_products(query: str, max_results: int = 5) -> list:
    """Search product catalog by keyword.

    Args:
        query: Search terms for products
        max_results: Maximum number of results to return (default 5)

    Returns:
        List of matching product dictionaries
    """
    return [{"name": "Widget", "price": 9.99}]

agent = Agent(
    name="assistant",
    model="gemini-2.5-flash",
    instruction="Help users with weather and product questions.",
    tools=[get_weather, search_products]  # Auto-wrapped as FunctionTool
)
```

### Explicit FunctionTool Wrapping

```python
from google.adk.tools import FunctionTool

def my_func(query: str) -> str:
    """Process a query."""
    return f"Processed: {query}"

# Explicit wrapping (usually unnecessary - auto-wrapped when passed to Agent)
tool = FunctionTool(func=my_func)

agent = Agent(
    name="agent",
    model="gemini-2.5-flash",
    tools=[tool]
)
```

### TypeScript Function Tools

```typescript
import { z } from 'zod';
import { FunctionTool, LlmAgent } from '@google/adk';

const getWeatherTool = new FunctionTool({
    name: 'getWeather',
    description: 'Get current weather for a city',
    parameters: z.object({
        city: z.string().describe('City name'),
    }),
    execute: async ({ city }) => {
        return { temp: '72F', condition: 'sunny' };
    },
});

const agent = new LlmAgent({
    name: 'weather_agent',
    model: 'gemini-2.5-flash',
    instruction: 'Help with weather questions.',
    tools: [getWeatherTool],
});
```

## Gemini API Tools

These tools are built into the Gemini model and execute server-side within the Gemini API. They do not require external service setup.

### Google Search Tool

Enables the agent to perform grounded web searches using Google Search. Results include source citations and are factually grounded.

#### Python

```python
from google.adk.agents import Agent
from google.adk.tools import google_search

agent = Agent(
    name="research_agent",
    model="gemini-2.5-flash",
    instruction="""You are a research assistant.
    Use Google Search to find current, accurate information.
    Always cite your sources.""",
    tools=[google_search]
)

root_agent = agent
```

#### TypeScript

```typescript
import { LlmAgent, GoogleSearchTool } from '@google/adk';

const agent = new LlmAgent({
    name: 'research_agent',
    model: 'gemini-2.5-flash',
    instruction: 'You are a research assistant. Use Google Search for current info.',
    tools: [new GoogleSearchTool()],
});
```

#### How It Works
- The LLM decides when to invoke search based on the user query and instructions
- Google Search returns grounded results with citations
- The LLM synthesizes results into a coherent response
- Grounding metadata (sources, URLs) is included in the response

#### Key Considerations
- Requires a Gemini model (will not work with non-Gemini models)
- Search results are grounded (factual, sourced) unlike custom search tools
- Works best with `gemini-2.5-flash` or `gemini-2.5-pro`
- Subject to Google Search API quotas

### Code Execution Tool

Enables the agent to generate and execute Python code in a secure, sandboxed environment within the Gemini API.

#### Python

Code execution is configured at the agent level via `code_executor`,
not as an importable tool.

```python
from google.adk.agents import Agent
from google.adk.code_executors import BuiltInCodeExecutor

agent = Agent(
    name="calculator",
    model="gemini-2.5-flash",
    code_executor=BuiltInCodeExecutor(),
    instruction="""Write and run Python code to solve problems.
    Available libraries: math, numpy-like operations, string processing.
    Always show your code and explain results."""
)

root_agent = agent
```

**Note**: `code_executor` is mutually exclusive with `output_schema`.
An agent cannot both execute code and produce a structured JSON output.

#### How It Works
- Agent generates Python code based on the request
- Code runs in a sandboxed environment (no filesystem, no network)
- Standard library plus limited scientific packages available
- Output (stdout, return values) is captured and returned to the agent
- Agent interprets the output and provides a response

#### Key Considerations
- Sandboxed: no file I/O, no network access, no pip installs
- Limited to standard library and select scientific packages
- Code execution happens server-side in Google's infrastructure
- Great for math, data transformation, string processing, logic problems
- Cannot interact with external APIs or databases

### Combining Google Search with Code Execution

Because code execution is configured via `code_executor` (not as a tool),
you combine it with `google_search` by setting both on the agent:

```python
from google.adk.agents import Agent
from google.adk.tools import google_search
from google.adk.code_executors import BuiltInCodeExecutor

agent = Agent(
    name="research_analyst",
    model="gemini-2.5-flash",
    instruction="""You are a research analyst.
    - Use Google Search to find data and statistics
    - Write and run Python code to perform calculations and analysis
    - Combine research with computation for data-driven answers""",
    tools=[google_search],
    code_executor=BuiltInCodeExecutor(),
)

root_agent = agent
```

## Google Cloud Tools

These tools integrate with managed Google Cloud services. They require a Google Cloud project with appropriate APIs enabled.

### Prerequisites for All Google Cloud Tools

```bash
# Authenticate
gcloud auth login
gcloud auth application-default login

# Set project
gcloud config set project YOUR_PROJECT_ID
```

```python
# .env file
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_CLOUD_LOCATION=us-central1
```

### Vertex AI Search Tool

Searches over your own data using Vertex AI Search (formerly Enterprise Search). Supports structured, unstructured, and website data.

#### Setup

1. Create a Vertex AI Search data store in Google Cloud Console
2. Ingest your documents (PDF, HTML, CSV, BigQuery, etc.)
3. Create a search app linked to the data store

#### Python

```python
from google.adk.agents import Agent
from google.adk.tools.vertex_ai_search_tool import VertexAiSearchTool

# Create the tool with your data store ID
search_tool = VertexAiSearchTool(
    data_store_id="projects/YOUR_PROJECT/locations/global/collections/default_collection/dataStores/YOUR_DATA_STORE_ID"
)

agent = Agent(
    name="knowledge_agent",
    model="gemini-2.5-flash",
    instruction="""You are a knowledge base assistant.
    Use the search tool to find answers from our documentation.
    Always cite the source document in your response.""",
    tools=[search_tool]
)

root_agent = agent
```

#### Key Considerations
- Data must be pre-ingested into a Vertex AI Search data store
- Supports multiple data types: documents, websites, structured data
- Results include source attribution and relevance scores
- Requires `discoveryengine.googleapis.com` API enabled
- Billing applies for Vertex AI Search usage

### BigQuery Tools

Query and analyze data in BigQuery directly from your agent.

#### Python

```python
from google.adk.agents import Agent
from google.adk.tools.bigquery import BigQueryCredentialsConfig, BigQueryToolset

# Configure BigQuery access
bq_toolset = BigQueryToolset(
    credentials_config=BigQueryCredentialsConfig(
        project="your-project-id",
        location="US"
    )
)

# Get the tools from the toolset
bq_tools = bq_toolset.get_tools()

agent = Agent(
    name="data_analyst",
    model="gemini-2.5-flash",
    instruction="""You are a data analyst.
    Use BigQuery tools to:
    1. List available datasets and tables
    2. Get table schemas
    3. Write and execute SQL queries
    4. Analyze results

    Always explain your SQL queries before running them.
    Format results as readable tables.""",
    tools=bq_tools
)

root_agent = agent
```

#### Available BigQuery Operations
The BigQuery toolset provides multiple tools:
- **list_datasets**: List available BigQuery datasets
- **list_tables**: List tables in a dataset
- **get_table_schema**: Get column definitions for a table
- **execute_sql**: Run SQL queries and return results

#### Key Considerations
- Requires `bigquery.googleapis.com` API enabled
- User/service account needs appropriate BigQuery IAM roles
- Be cautious with large queries (cost and performance)
- Consider adding row limits in agent instructions
- SQL injection risk: the LLM generates SQL, so validate in production

### Vertex AI RAG (Retrieval Augmented Generation)

Retrieves relevant context from a Vertex AI RAG corpus for grounding agent responses.

#### Setup

1. Create a RAG corpus in Vertex AI
2. Import documents into the corpus
3. Reference the corpus in your tool configuration

#### Python

```python
from google.adk.agents import Agent
from google.adk.tools.retrieval import VertexAiRagRetrieval

# Configure RAG retrieval tool
rag_tool = VertexAiRagRetrieval(
    name="knowledge_base",
    description="Search internal knowledge base for relevant information",
    rag_corpus_name="projects/YOUR_PROJECT/locations/us-central1/ragCorpora/YOUR_CORPUS_ID",
    similarity_top_k=5,  # Number of chunks to retrieve
    vector_distance_threshold=0.7  # Minimum relevance threshold
)

agent = Agent(
    name="rag_agent",
    model="gemini-2.5-flash",
    instruction="""You are an expert assistant.
    Always search the knowledge base before answering.
    Base your responses on retrieved documents.
    If no relevant information is found, say so clearly.""",
    tools=[rag_tool]
)

root_agent = agent
```

#### Key Considerations
- Documents must be pre-imported into a Vertex AI RAG corpus
- Supports PDF, TXT, HTML, and other document formats
- `similarity_top_k` controls how many chunks are retrieved
- `vector_distance_threshold` filters out low-relevance results
- Requires `aiplatform.googleapis.com` API enabled
- Embedding model determines retrieval quality

### Combining Google Cloud Tools

```python
from google.adk.agents import Agent
from google.adk.tools.vertex_ai_search_tool import VertexAiSearchTool
from google.adk.tools.bigquery import BigQueryCredentialsConfig, BigQueryToolset
from google.adk.tools.retrieval import VertexAiRagRetrieval

# Enterprise search for documentation
doc_search = VertexAiSearchTool(
    data_store_id="projects/my-proj/locations/global/collections/default_collection/dataStores/docs-store"
)

# BigQuery for analytics
bq_toolset = BigQueryToolset(
    credentials_config=BigQueryCredentialsConfig(
        project="my-proj", location="US"
    )
)

# RAG for internal knowledge
rag = VertexAiRagRetrieval(
    name="internal_kb",
    description="Search internal knowledge base",
    rag_corpus_name="projects/my-proj/locations/us-central1/ragCorpora/kb-corpus",
    similarity_top_k=3
)

agent = Agent(
    name="enterprise_assistant",
    model="gemini-2.5-flash",
    instruction="""You are an enterprise assistant with access to:
    - Document search: For product docs and manuals
    - BigQuery: For data analysis and metrics
    - Knowledge base: For internal policies and procedures

    Choose the appropriate tool based on the question type.""",
    tools=[doc_search, rag] + bq_toolset.get_tools()
)

root_agent = agent
```

## Third-Party Tools

ADK supports integration with third-party services through dedicated tool packages and adapter patterns.

### LangChain Tool Adapter

Use any LangChain tool within an ADK agent.

```python
from google.adk.agents import Agent
from google.adk.tools.langchain_tool import LangchainTool
from langchain_community.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper

# Wrap a LangChain tool for ADK
wikipedia_tool = LangchainTool(
    tool=WikipediaQueryRun(api_wrapper=WikipediaAPIWrapper())
)

agent = Agent(
    name="wiki_agent",
    model="gemini-2.5-flash",
    instruction="Use Wikipedia to answer factual questions.",
    tools=[wikipedia_tool]
)

root_agent = agent
```

#### Available LangChain Tools (Common Examples)

```python
# Wikipedia
from langchain_community.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper

# Tavily Search
from langchain_community.tools.tavily_search import TavilySearchResults

# Python REPL
from langchain_experimental.tools import PythonREPLTool

# Requests (HTTP)
from langchain_community.tools.requests.tool import RequestsGetTool

# Any LangChain tool can be wrapped:
from google.adk.tools.langchain_tool import LangchainTool
adk_tool = LangchainTool(tool=any_langchain_tool)
```

### CrewAI Tool Adapter

Use CrewAI tools within an ADK agent.

```python
from google.adk.agents import Agent
from google.adk.tools.crewai_tool import CrewaiTool
from crewai_tools import SerperDevTool, ScrapeWebsiteTool

# Wrap CrewAI tools for ADK
search_tool = CrewaiTool(tool=SerperDevTool())
scrape_tool = CrewaiTool(tool=ScrapeWebsiteTool())

agent = Agent(
    name="web_research_agent",
    model="gemini-2.5-flash",
    instruction="Search the web and scrape pages for information.",
    tools=[search_tool, scrape_tool]
)

root_agent = agent
```

### MCP (Model Context Protocol) Tools

Connect to any MCP-compatible server to use its tools.

```python
from google.adk.agents import Agent
from google.adk.tools.mcp_tool import MCPToolset
from google.adk.tools.mcp_tool.mcp_session_manager import SseServerParams
from mcp import StdioServerParameters

# Connect to an MCP server via SSE
mcp_tools, exit_stack = await MCPToolset.from_server(
    connection_params=SseServerParams(
        url="http://localhost:3000/sse"
    )
)

agent = Agent(
    name="mcp_agent",
    model="gemini-2.5-flash",
    instruction="Use available tools to help the user.",
    tools=mcp_tools
)

root_agent = agent

# IMPORTANT: Clean up when done
# await exit_stack.aclose()
```

#### MCP via Stdio (Local Process)

```python
from google.adk.tools.mcp_tool import MCPToolset
from mcp import StdioServerParameters

mcp_tools, exit_stack = await MCPToolset.from_server(
    connection_params=StdioServerParameters(
        command="npx",
        args=["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
    )
)

agent = Agent(
    name="filesystem_agent",
    model="gemini-2.5-flash",
    instruction="Help the user manage files.",
    tools=mcp_tools
)
```

### Application Integration Tools

#### REST API Tool

```python
from google.adk.tools.application_integration_tool import ApplicationIntegrationToolset

# Connect to an Application Integration
integration_toolset = ApplicationIntegrationToolset(
    project="your-project-id",
    location="us-central1",
    integration="your-integration-name",
    trigger="api_trigger/your_trigger"
)

agent = Agent(
    name="integration_agent",
    model="gemini-2.5-flash",
    instruction="Use the integration tools to process requests.",
    tools=integration_toolset.get_tools()
)
```

### OpenAPI Tool

Generate tools from an OpenAPI (Swagger) specification.

```python
from google.adk.tools.openapi_tool import OpenAPIToolset

# Load from an OpenAPI spec
openapi_toolset = OpenAPIToolset(
    spec_str=open("openapi.yaml").read(),
    spec_str_type="yaml"
)

# Or load from a URL
openapi_toolset = OpenAPIToolset(
    spec_url="https://api.example.com/openapi.json"
)

agent = Agent(
    name="api_agent",
    model="gemini-2.5-flash",
    instruction="Use the API tools to fulfill user requests.",
    tools=openapi_toolset.get_tools()
)
```

## Tool Selection Strategies

### Decision Matrix

| Need | Tool Type | Example |
|------|-----------|---------|
| Current web information | `google_search` | News, facts, real-time data |
| Math/data computation | `BuiltInCodeExecutor` (agent `code_executor=`) | Calculations, data transforms |
| Your own documents | `VertexAiSearchTool` | Product docs, manuals |
| Database analytics | `BigQueryToolset` | Sales reports, metrics |
| Vector similarity search | `VertexAiRagRetrieval` | Semantic document search |
| External APIs | `OpenAPIToolset` | REST API integration |
| LangChain ecosystem | `LangchainTool` | Wikipedia, Tavily, etc. |
| MCP servers | `MCPToolset` | Any MCP-compatible service |
| Custom business logic | Function tools | Your own Python functions |

### When to Use Built-in vs Custom

**Use Gemini API tools when:**
- You need web search with built-in grounding and citations
- You need sandboxed code execution without infrastructure
- You want zero-setup, model-native capabilities

**Use Google Cloud tools when:**
- You have data in Google Cloud (BigQuery, GCS, Vertex AI)
- You need enterprise-grade search over your documents
- You need managed RAG with automatic chunking and embedding

**Use third-party tools when:**
- You need specific external service integration
- You already use LangChain/CrewAI tools you want to reuse
- You need MCP-compatible tool servers

**Use custom function tools when:**
- You need specific business logic
- You are calling internal APIs
- You need full control over behavior and error handling

### Performance Considerations

- **Gemini API tools** (Search, Code Exec) add latency to the Gemini API call itself
- **Google Cloud tools** depend on the underlying service latency (BigQuery queries can be slow)
- **Third-party tools** add network hops and depend on external service performance
- **Function tools** are fastest since they run in your process
- **Limit tool count**: Agents with too many tools (>10-15) may struggle to choose correctly
- **Use descriptions wisely**: Clear tool descriptions reduce incorrect tool selection

## Best Practices

### Tool Documentation

```python
# GOOD: Detailed docstring with clear purpose, args, and returns
def search_orders(
    customer_id: str,
    status: str = "all",
    limit: int = 10
) -> dict:
    """Search customer orders by ID and optional status filter.

    Use this tool when the user asks about their orders, order history,
    or order status. Do NOT use for product searches.

    Args:
        customer_id: The unique customer identifier (e.g., "CUST-12345")
        status: Filter by status: "pending", "shipped", "delivered", "all"
        limit: Maximum orders to return (default 10, max 100)

    Returns:
        dict with 'orders' list and 'total_count' integer
    """
    # Implementation
    return {"orders": [], "total_count": 0}
```

### Error Handling in Tools

```python
def call_external_api(endpoint: str, params: dict) -> dict:
    """Call an external API endpoint.

    Args:
        endpoint: API endpoint path
        params: Query parameters
    """
    try:
        response = requests.get(f"https://api.example.com/{endpoint}", params=params)
        response.raise_for_status()
        return {"success": True, "data": response.json()}
    except requests.ConnectionError:
        return {"success": False, "error": "Could not connect to API. Try again later."}
    except requests.HTTPError as e:
        return {"success": False, "error": f"API returned error: {e.response.status_code}"}
    except Exception as e:
        return {"success": False, "error": f"Unexpected error: {str(e)}"}
```

### Tool Grouping with Toolsets

```python
# Group related tools into a toolset for organized management
from google.adk.tools import FunctionTool

def list_users() -> list:
    """List all users in the system."""
    return []

def get_user(user_id: str) -> dict:
    """Get user details by ID."""
    return {}

def update_user(user_id: str, name: str = None, email: str = None) -> dict:
    """Update user details."""
    return {}

# Group related tools
user_management_tools = [list_users, get_user, update_user]

# Use in agent
agent = Agent(
    name="admin_agent",
    model="gemini-2.5-flash",
    instruction="Manage user accounts using the available tools.",
    tools=user_management_tools
)
```

### Agent-as-Tool Pattern

```python
from google.adk.agents import Agent
from google.adk.tools import AgentTool

# Specialist agent
analysis_agent = Agent(
    name="analyst",
    model="gemini-2.5-pro",  # Use a more capable model for analysis
    instruction="Perform deep analysis of the provided data."
)

# Wrap as tool for another agent
analysis_tool = AgentTool(agent=analysis_agent)

# Main agent uses specialist as a tool
main_agent = Agent(
    name="coordinator",
    model="gemini-2.5-flash",
    instruction="""Handle user requests.
    For complex analysis, delegate to the analyst tool.""",
    tools=[analysis_tool, google_search]
)
```

### Authentication for Tools

```python
# For tools that need API keys, use environment variables
import os

def search_external(query: str) -> dict:
    """Search external service."""
    api_key = os.environ.get("EXTERNAL_API_KEY")
    if not api_key:
        return {"error": "API key not configured"}
    # Use the key...
    return {"results": []}

# .env file
# EXTERNAL_API_KEY=your_key_here
```

## Common Pitfalls

### Using google_search with non-Gemini models

```python
# WRONG: google_search only works with Gemini models
from google.adk.tools import google_search

agent = Agent(
    name="agent",
    model="claude-sonnet-4-20250514",  # Not Gemini!
    tools=[google_search]  # Will fail
)

# CORRECT: Use a custom search tool for non-Gemini models
def web_search(query: str) -> str:
    """Search the web."""
    # Use your own search API (Tavily, Serper, etc.)
    return results

agent = Agent(
    name="agent",
    model="claude-sonnet-4-20250514",
    tools=[web_search]
)
```

### Missing tool docstrings

```python
# WRONG: LLM has no idea what this tool does or when to use it
def process(x):
    return do_something(x)

# CORRECT: Clear docstring with types
def process_order(order_id: str) -> dict:
    """Process a pending order and mark it as fulfilled.

    Args:
        order_id: The order ID to process (e.g., "ORD-12345")

    Returns:
        dict with processing status and confirmation number
    """
    return {"status": "fulfilled", "confirmation": "CONF-789"}
```

### Too many tools on one agent

```python
# WRONG: Agent overwhelmed with choices
agent = Agent(
    name="do_everything",
    model="gemini-2.5-flash",
    tools=[tool1, tool2, tool3, ..., tool25]  # Too many!
)

# CORRECT: Split into specialized sub-agents
search_agent = Agent(
    name="searcher",
    model="gemini-2.5-flash",
    description="Handles all search operations",
    tools=[web_search, doc_search, product_search]
)

data_agent = Agent(
    name="analyst",
    model="gemini-2.5-flash",
    description="Handles data analysis",
    tools=[bq_tools, calculator, chart_tool]
)

root_agent = Agent(
    name="router",
    model="gemini-2.5-flash",
    instruction="Route to searcher for lookups, analyst for data tasks.",
    tools=[search_agent, data_agent]
)
```

### Not handling async tool cleanup

```python
# WRONG: MCP connection never cleaned up
mcp_tools, exit_stack = await MCPToolset.from_server(
    connection_params=SseServerParams(url="http://localhost:3000/sse")
)
agent = Agent(tools=mcp_tools)
# exit_stack is leaked!

# CORRECT: Use async context manager or explicit cleanup
async def create_agent():
    from google.adk.tools.mcp_tool.mcp_session_manager import SseServerParams
    mcp_tools, exit_stack = await MCPToolset.from_server(
        connection_params=SseServerParams(url="http://localhost:3000/sse")
    )
    try:
        agent = Agent(
            name="mcp_agent",
            model="gemini-2.5-flash",
            tools=mcp_tools
        )
        # Use agent...
    finally:
        await exit_stack.aclose()
```

### Forgetting BigQuery permissions

```python
# Error: 403 Access Denied
# Fix: Grant the service account or user these IAM roles:
# - roles/bigquery.dataViewer (read tables)
# - roles/bigquery.jobUser (run queries)
# - roles/bigquery.user (list datasets/tables)
```

### Returning non-serializable objects from tools

```python
# WRONG: Returns objects the LLM can't parse
def get_data() -> object:
    return SomeComplexObject()

# CORRECT: Return serializable types (dict, list, str, int, float, bool)
def get_data() -> dict:
    """Get data as a dictionary."""
    obj = SomeComplexObject()
    return {"field1": obj.field1, "field2": str(obj.field2)}
```

### Code execution tool for external operations

```python
# WRONG: Trying to use code execution for web requests
from google.adk.code_executors import BuiltInCodeExecutor
agent = Agent(
    model="gemini-2.5-flash",
    code_executor=BuiltInCodeExecutor(),
    instruction="Use code execution to call external APIs"
    # Code execution is SANDBOXED - no network access!
)

# CORRECT: Use a custom function tool for external operations
def call_api(url: str) -> dict:
    """Call an external API."""
    response = requests.get(url)
    return response.json()

agent = Agent(
    model="gemini-2.5-flash",
    tools=[call_api]
)
```

## References

- [Tools Overview](https://google.github.io/adk-docs/tools/)
- [Gemini API Tools](https://google.github.io/adk-docs/tools/gemini-api/)
- [Google Cloud Tools](https://google.github.io/adk-docs/tools/google-cloud/)
- [Third-Party Tools](https://google.github.io/adk-docs/tools/third-party/)
- [Function Tools](https://google.github.io/adk-docs/tools/function-tools/)
- [MCP Tools](https://google.github.io/adk-docs/tools/mcp-tools/)
- [OpenAPI Tools](https://google.github.io/adk-docs/tools/openapi-tools/)
- [ADK Documentation](https://google.github.io/adk-docs)
