# ADK Custom Tools

Use this reference for creating custom tools in Google's Agent Development Kit. Covers function tools, MCP integration, OpenAPI tools, authentication, and tool optimization patterns.

## When to Use This Reference

- Creating Python/TypeScript/Go/Java function tools for ADK agents
- Integrating MCP (Model Context Protocol) servers with ADK
- Loading tools from OpenAPI/Swagger specifications
- Adding authentication (OAuth2, API keys) to tools
- Requiring user confirmation before tool execution
- Optimizing tool performance

## Function Tools

Function tools are the most common way to give agents capabilities. ADK auto-wraps plain functions, or you can use the `FunctionTool` class for more control.

### Python Function Tools

#### Auto-Wrapped Functions (Simplest)

Pass a plain Python function directly to the `tools` list. ADK wraps it automatically.

```python
from google.adk.agents import Agent

def get_weather(city: str) -> dict:
    """Retrieves current weather for a given city.

    Args:
        city: The name of the city (e.g., "London", "New York").

    Returns:
        dict with temperature and weather condition.
    """
    # Your implementation
    return {"temperature": "72F", "condition": "sunny", "city": city}

agent = Agent(
    name="weather_agent",
    model="gemini-2.5-flash",
    instruction="Use the get_weather tool when users ask about weather.",
    tools=[get_weather]  # Auto-wrapped as FunctionTool
)
```

**Critical**: The LLM uses the function's **docstring** to decide when and how to call the tool. Always write clear, descriptive docstrings. The LLM uses **type hints** to understand parameter types and validate inputs.

#### Docstring Requirements

The docstring is the primary mechanism by which the LLM understands a tool. It must include:

1. **Summary line**: What the tool does (first line)
2. **Args section**: Description of each parameter
3. **Returns section**: What the tool returns

```python
def search_products(
    query: str,
    category: str = "all",
    max_results: int = 10
) -> dict:
    """Search the product catalog for items matching a query.

    Use this tool when the user wants to find products, check inventory,
    or browse the catalog. Do NOT use for pricing questions (use get_price instead).

    Args:
        query: Search keywords describing the desired product.
        category: Product category to filter by. Options: "electronics",
                  "clothing", "home", "all". Defaults to "all".
        max_results: Maximum number of results to return. Defaults to 10.

    Returns:
        dict containing 'products' list and 'total_count' integer.
    """
    # Implementation
    return {"products": [...], "total_count": 42}
```

#### Type Hints and Return Types

ADK uses type hints to generate the tool's JSON schema for the LLM.

**Supported parameter types:**
- `str`, `int`, `float`, `bool` - primitives
- `list[str]`, `list[int]` - typed lists
- `dict` - dictionaries
- `Optional[str]` - optional parameters (with default values)
- Enum types for constrained choices

**Return types:**
- `dict` - recommended (structured data, the LLM parses it well)
- `str` - simple text responses
- `list` - collections
- Primitives (`int`, `float`, `bool`) - simple values

```python
from typing import Optional
from enum import Enum

class Priority(str, Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"

def create_ticket(
    title: str,
    description: str,
    priority: Priority = Priority.MEDIUM,
    assignee: Optional[str] = None
) -> dict:
    """Create a support ticket in the tracking system.

    Args:
        title: Short summary of the issue.
        description: Detailed description of the problem.
        priority: Ticket priority level. Defaults to medium.
        assignee: Username to assign the ticket to. Optional.

    Returns:
        dict with 'ticket_id', 'status', and 'url' keys.
    """
    return {
        "ticket_id": "TICKET-123",
        "status": "created",
        "url": "https://tracker.example.com/TICKET-123"
    }
```

#### Explicit FunctionTool Class

Use `FunctionTool` for more control over tool wrapping.

```python
from google.adk.tools import FunctionTool

def raw_search(query: str, limit: int = 5) -> dict:
    """Search for information."""
    return {"results": [...]}

# Explicit wrapping
search_tool = FunctionTool(func=raw_search)

agent = Agent(
    name="agent",
    model="gemini-2.5-flash",
    tools=[search_tool]
)
```

#### Async Function Tools

ADK natively supports async functions. Use `async def` for I/O-bound operations.

```python
import aiohttp

async def fetch_data(url: str) -> dict:
    """Fetch data from an external API endpoint.

    Args:
        url: The URL to fetch data from.

    Returns:
        dict with 'status' and 'data' keys.
    """
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            data = await response.json()
            return {"status": response.status, "data": data}

agent = Agent(
    name="api_agent",
    model="gemini-2.5-flash",
    tools=[fetch_data]  # Async functions work the same way
)
```

#### Accessing Tool Context

Function tools can receive a `ToolContext` parameter to access session state, artifacts, and agent information.

```python
from google.adk.tools import ToolContext

def save_note(content: str, tool_context: ToolContext) -> dict:
    """Save a note to the user's session.

    Args:
        content: The note content to save.

    Returns:
        dict confirming the note was saved.
    """
    # Access and modify session state
    notes = tool_context.state.get("notes", [])
    notes.append(content)
    tool_context.state["notes"] = notes

    return {"status": "saved", "total_notes": len(notes)}

def read_notes(tool_context: ToolContext) -> dict:
    """Read all saved notes from the current session.

    Returns:
        dict with list of saved notes.
    """
    notes = tool_context.state.get("notes", [])
    return {"notes": notes, "count": len(notes)}
```

**Important**: The `tool_context` parameter is automatically injected by ADK. It is NOT passed by the LLM and should NOT appear in your docstring's Args section. ADK detects parameters named `tool_context` (typed as `ToolContext`) and excludes them from the tool schema sent to the LLM.

#### Working with Artifacts in Tools

```python
from google.adk.tools import ToolContext
from google.genai import types

def save_file(filename: str, content: str, tool_context: ToolContext) -> dict:
    """Save content as a file artifact.

    Args:
        filename: Name for the file.
        content: Text content to save.

    Returns:
        dict confirming the file was saved.
    """
    artifact = types.Part.from_text(text=content)
    version = tool_context.save_artifact(filename=filename, artifact=artifact)
    return {"status": "saved", "filename": filename, "version": version}

def load_file(filename: str, tool_context: ToolContext) -> dict:
    """Load a previously saved file artifact.

    Args:
        filename: Name of the file to load.

    Returns:
        dict with file content.
    """
    artifact = tool_context.load_artifact(filename=filename)
    if artifact is None:
        return {"error": f"File '{filename}' not found"}
    return {"filename": filename, "content": artifact.text}
```

### TypeScript Function Tools

In TypeScript, use `FunctionTool` with Zod schemas for parameter validation.

```typescript
import { z } from 'zod';
import { FunctionTool, LlmAgent } from '@google/adk';

// Define parameter schema with Zod
const weatherParamsSchema = z.object({
    city: z.string().describe('Name of the city to get weather for'),
    units: z.enum(['celsius', 'fahrenheit']).default('celsius')
        .describe('Temperature unit preference'),
});

// Create function tool
const getWeatherTool = new FunctionTool({
    name: 'get_weather',
    description: 'Get current weather conditions for a city. Use when user asks about weather, temperature, or forecasts.',
    parameters: weatherParamsSchema,
    execute: async (params) => {
        const { city, units } = params;
        // Your implementation
        return {
            city,
            temperature: units === 'celsius' ? '22C' : '72F',
            condition: 'sunny',
        };
    },
});

// Use in agent
const agent = new LlmAgent({
    name: 'weather_agent',
    model: 'gemini-2.5-flash',
    instruction: 'Help users check weather conditions.',
    tools: [getWeatherTool],
});
```

#### TypeScript Tool with Complex Schemas

```typescript
import { z } from 'zod';
import { FunctionTool } from '@google/adk';

const searchSchema = z.object({
    query: z.string().describe('Search keywords'),
    filters: z.object({
        category: z.string().optional().describe('Category filter'),
        minPrice: z.number().optional().describe('Minimum price'),
        maxPrice: z.number().optional().describe('Maximum price'),
    }).optional().describe('Optional search filters'),
    page: z.number().default(1).describe('Page number for pagination'),
});

const searchProductsTool = new FunctionTool({
    name: 'search_products',
    description: 'Search the product catalog. Use when user wants to find or browse products.',
    parameters: searchSchema,
    execute: async (params) => {
        // Implementation
        return { products: [], totalCount: 0, page: params.page };
    },
});
```

### Go Function Tools

```go
package main

import (
    "context"
    "google.golang.org/adk"
    "google.golang.org/adk/tools"
)

// Define tool function
func getWeather(ctx context.Context, input struct {
    City string `json:"city" description:"Name of the city"`
}) (map[string]any, error) {
    return map[string]any{
        "temperature": "72F",
        "condition":   "sunny",
        "city":        input.City,
    }, nil
}

func main() {
    weatherTool := tools.NewFunctionTool(
        "get_weather",
        "Get current weather for a city",
        getWeather,
    )

    agent := adk.NewLlmAgent(adk.LlmAgentConfig{
        Name:        "weather_agent",
        Model:       "gemini-2.5-flash",
        Instruction: "Help users check weather.",
        Tools:       []adk.Tool{weatherTool},
    })
    // ...
}
```

### Java Function Tools

```java
import com.google.adk.agents.LlmAgent;
import com.google.adk.tools.FunctionTool;

// Define tool with annotation-based approach
public class WeatherTools {

    @Tool(description = "Get current weather for a city")
    public static Map<String, Object> getWeather(
            @Param(description = "Name of the city") String city) {
        Map<String, Object> result = new HashMap<>();
        result.put("temperature", "72F");
        result.put("condition", "sunny");
        result.put("city", city);
        return result;
    }
}

// Use in agent
FunctionTool weatherTool = FunctionTool.create(WeatherTools.class, "getWeather");

LlmAgent agent = LlmAgent.builder()
    .name("weather_agent")
    .model("gemini-2.5-flash")
    .instruction("Help users check weather.")
    .tools(List.of(weatherTool))
    .build();
```

## MCP Tools Integration

### What is MCP (Model Context Protocol)

MCP is an open protocol that standardizes how applications provide tools and context to LLMs. ADK can connect to any MCP-compatible server, instantly giving your agents access to a broad ecosystem of capabilities without writing custom tool code.

**Key benefits:**
- Reuse existing MCP servers (filesystem, database, GitHub, etc.)
- Standardized tool discovery and invocation
- Connect to local (stdio) or remote (SSE/HTTP) servers

### MCPToolset Usage (Python)

#### Connecting to a Stdio MCP Server

```python
from google.adk.agents import Agent
from google.adk.tools.mcp_tool import MCPToolset
from mcp import StdioServerParameters

agent = Agent(
    name="filesystem_agent",
    model="gemini-2.5-flash",
    instruction="""You are a file manager agent.
    Use the available tools to read, write, and manage files.
    Always confirm before modifying or deleting files.""",
    tools=[
        MCPToolset(
            connection_params=StdioServerParameters(
                command="npx",
                args=["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed/dir"],
            )
        )
    ]
)
```

#### Connecting to an SSE MCP Server

```python
from google.adk.tools.mcp_tool import MCPToolset
from google.adk.tools.mcp_tool.mcp_session_manager import SseServerParams

agent = Agent(
    name="data_agent",
    model="gemini-2.5-flash",
    instruction="Query and analyze data using the available tools.",
    tools=[
        MCPToolset(
            connection_params=SseServerParams(
                url="http://localhost:8080/sse",
                headers={"Authorization": "Bearer your-token"},
            )
        )
    ]
)
```

#### Connecting to a Streamable HTTP MCP Server

```python
from google.adk.tools.mcp_tool import MCPToolset
from google.adk.tools.mcp_tool.mcp_session_manager import (
    StreamableHTTPConnectionParams,
)

agent = Agent(
    name="api_agent",
    model="gemini-2.5-flash",
    tools=[
        MCPToolset(
            connection_params=StreamableHTTPConnectionParams(
                url="http://localhost:3000/mcp",
            )
        )
    ]
)
```

#### Filtering MCP Tools

When an MCP server exposes many tools but you only need a subset:

```python
MCPToolset(
    connection_params=StdioServerParameters(
        command="npx",
        args=["-y", "@modelcontextprotocol/server-filesystem", "/tmp"],
    ),
    # Only expose these tools to the agent
    tool_filter=["read_file", "write_file", "list_directory"]
)
```

You can also use a callable filter for dynamic filtering:

```python
def my_tool_filter(tool, server_params):
    """Only allow read operations."""
    return tool.name.startswith("read_") or tool.name.startswith("list_")

MCPToolset(
    connection_params=StdioServerParameters(
        command="npx",
        args=["-y", "@modelcontextprotocol/server-filesystem", "/tmp"],
    ),
    tool_filter=my_tool_filter
)
```

### TypeScript MCP Integration

```typescript
import { LlmAgent, MCPToolset, StdioServerParameters } from '@google/adk';

const agent = new LlmAgent({
    name: 'fs_agent',
    model: 'gemini-2.5-flash',
    instruction: 'Manage files using available tools.',
    tools: [
        new MCPToolset({
            connectionParams: new StdioServerParameters({
                command: 'npx',
                args: ['-y', '@modelcontextprotocol/server-filesystem', '/tmp'],
            }),
        }),
    ],
});
```

### Common MCP Servers

| Server | Command | Purpose |
|--------|---------|---------|
| **Filesystem** | `npx -y @modelcontextprotocol/server-filesystem /path` | Read/write files, list directories |
| **GitHub** | `npx -y @modelcontextprotocol/server-github` | Repos, issues, PRs, code search |
| **PostgreSQL** | `npx -y @modelcontextprotocol/server-postgres postgres://...` | Query PostgreSQL databases |
| **SQLite** | `npx -y @modelcontextprotocol/server-sqlite /path/to/db.sqlite` | Query SQLite databases |
| **Brave Search** | `npx -y @modelcontextprotocol/server-brave-search` | Web search via Brave |
| **Google Maps** | `npx -y @modelcontextprotocol/server-google-maps` | Geocoding, directions, places |
| **Slack** | `npx -y @modelcontextprotocol/server-slack` | Send/read Slack messages |
| **Memory** | `npx -y @modelcontextprotocol/server-memory` | Persistent key-value memory |

**Environment variables** for MCP servers are passed through:

```python
MCPToolset(
    connection_params=StdioServerParameters(
        command="npx",
        args=["-y", "@modelcontextprotocol/server-github"],
        env={"GITHUB_TOKEN": "ghp_your_token_here"}
    )
)
```

### MCP Lifecycle Management

MCP connections are managed automatically by ADK. When using `adk web run` or `adk run`, connections are established on agent load and closed on shutdown.

For programmatic usage, handle the lifecycle explicitly:

```python
from google.adk.tools.mcp_tool import MCPToolset
from mcp import StdioServerParameters

async def main():
    from mcp import StdioServerParameters
    tools, exit_stack = await MCPToolset.from_server(
        connection_params=StdioServerParameters(
            command="npx",
            args=["-y", "@modelcontextprotocol/server-filesystem", "/tmp"],
        )
    )

    agent = Agent(
        name="agent",
        model="gemini-2.5-flash",
        tools=tools
    )

    try:
        # Use agent...
        pass
    finally:
        await exit_stack.aclose()  # Clean up MCP connection
```

## OpenAPI Tools

### Loading Tools from OpenAPI Specs

`OpenAPIToolset` automatically generates tools from an OpenAPI (Swagger) specification. Each API endpoint becomes a callable tool.

```python
import json
from google.adk.tools.openapi_tool import OpenAPIToolset

# Load from a local YAML spec file
with open("path/to/openapi.yaml") as f:
    toolset = OpenAPIToolset(spec_str=f.read(), spec_str_type="yaml")

# Load from a URL (fetch it yourself, then construct)
import httpx
spec_text = httpx.get("https://api.example.com/openapi.json").text
toolset = OpenAPIToolset(spec_str=spec_text, spec_str_type="json")

# Or from an in-memory dict (serialize to a string first)
spec = {
    "openapi": "3.0.0",
    "info": {"title": "Pet Store", "version": "1.0.0"},
    "paths": {
        "/pets": {
            "get": {
                "operationId": "listPets",
                "summary": "List all pets",
                "parameters": [],
                "responses": {},
            }
        }
    },
}
toolset = OpenAPIToolset(spec_str=json.dumps(spec), spec_str_type="json")

agent = Agent(
    name="pet_store_agent",
    model="gemini-2.5-flash",
    instruction="Help users manage pets in the pet store.",
    tools=[toolset]
)
```

### OpenAPI Configuration

#### Filtering Operations

<!-- VERIFY: OpenAPIToolset operation filtering and base URL override -->
<!-- TODO: Confirm the exact kwargs (operation filter, base URL) on the
     real OpenAPIToolset constructor. The real API takes spec_str +
     spec_str_type; additional filtering/base-url kwargs need to be
     verified against the ADK docs before documenting here. -->

```python
# Example scaffolding (verify kwargs):
# toolset = OpenAPIToolset(
#     spec_str=open("openapi.yaml").read(),
#     spec_str_type="yaml",
# )
```

### Authentication with OpenAPI Tools

```python
from google.adk.tools.openapi_tool import OpenAPIToolset
from google.adk.auth import AuthCredential, AuthCredentialTypes, HttpAuth, HttpCredentials

# API key authentication
api_key_auth = AuthCredential(
    auth_type=AuthCredentialTypes.HTTP,
    http=HttpAuth(
        credentials=HttpCredentials(
            token="your-api-key"
        ),
        scheme="bearer"
    )
)

with open("openapi.yaml") as f:
    toolset = OpenAPIToolset(
        spec_str=f.read(),
        spec_str_type="yaml",
        auth_credential=api_key_auth,
    )

agent = Agent(
    name="api_agent",
    model="gemini-2.5-flash",
    tools=[toolset]
)
```

### REST API Tool (Low-Level)

For single API calls without a full OpenAPI spec:

```python
from google.adk.tools.openapi_tool.rest_api_tool import RestApiTool

tool = RestApiTool(
    name="get_user",
    description="Fetch user profile by ID",
    method="GET",
    url="https://api.example.com/users/{user_id}",
    headers={"Authorization": "Bearer token123"}
)

agent = Agent(
    name="api_agent",
    model="gemini-2.5-flash",
    tools=[tool]
)
```

## Authentication Patterns

ADK provides a structured authentication framework for tools that access protected resources.

### Core Authentication Types

```python
from google.adk.auth import (
    AuthCredential,
    AuthCredentialTypes,
    OAuth2Auth,
    HttpAuth,
    HttpCredentials,
    AuthScheme,
)
```

### API Key Authentication

```python
# HTTP Bearer token
credential = AuthCredential(
    auth_type=AuthCredentialTypes.HTTP,
    http=HttpAuth(
        credentials=HttpCredentials(token="your-api-key"),
        scheme="bearer"
    )
)

# API key in header
credential = AuthCredential(
    auth_type=AuthCredentialTypes.API_KEY,
    api_key="your-api-key",
    api_key_name="X-API-Key",  # Header name
    api_key_location="header"  # "header" or "query"
)
```

### OAuth2 Authentication

#### Authorization Code Flow

```python
from google.adk.auth import AuthCredential, AuthCredentialTypes, OAuth2Auth

oauth_credential = AuthCredential(
    auth_type=AuthCredentialTypes.OAUTH2,
    oauth2=OAuth2Auth(
        client_id="your-client-id",
        client_secret="your-client-secret",
        auth_uri="https://accounts.google.com/o/oauth2/auth",
        token_uri="https://oauth2.googleapis.com/token",
        scopes=["https://www.googleapis.com/auth/calendar.readonly"],
        redirect_uri="http://localhost:8080/callback",
    )
)
```

#### Using OAuth2 with Tools

ADK handles the OAuth2 flow through tool context. When a tool requires authentication, it can trigger an auth request:

```python
from google.adk.tools import ToolContext
from google.adk.auth import AuthCredential, AuthCredentialTypes, OAuth2Auth

# Define the credential configuration
OAUTH_CONFIG = AuthCredential(
    auth_type=AuthCredentialTypes.OAUTH2,
    oauth2=OAuth2Auth(
        client_id="your-client-id",
        client_secret="your-client-secret",
        auth_uri="https://provider.com/oauth2/auth",
        token_uri="https://provider.com/oauth2/token",
        scopes=["read", "write"],
    )
)

def access_protected_resource(query: str, tool_context: ToolContext) -> dict:
    """Access a protected API resource.

    Args:
        query: The query to send to the protected API.

    Returns:
        dict with API response data.
    """
    # Check if we have a valid credential
    cred = tool_context.get_auth_credential(OAUTH_CONFIG)

    if cred is None:
        # Request authentication - this pauses tool execution
        # and prompts the user to authenticate
        tool_context.request_credential(OAUTH_CONFIG)
        return {"status": "awaiting_authentication"}

    # Use the credential
    token = cred.oauth2.access_token
    # Make authenticated API call...
    return {"data": "protected resource data"}
```

#### Client Credentials Flow (Service-to-Service)

```python
oauth_credential = AuthCredential(
    auth_type=AuthCredentialTypes.OAUTH2,
    oauth2=OAuth2Auth(
        client_id="service-client-id",
        client_secret="service-client-secret",
        token_uri="https://auth.example.com/oauth/token",
        grant_type="client_credentials",
        scopes=["api.read"],
    )
)
```

### Service Account Authentication

For Google Cloud services:

```python
from google.adk.auth import AuthCredential, AuthCredentialTypes, ServiceAccountCredential

service_account_cred = AuthCredential(
    auth_type=AuthCredentialTypes.SERVICE_ACCOUNT,
    service_account=ServiceAccountCredential(
        service_account_email="my-sa@project.iam.gserviceaccount.com",
        scopes=["https://www.googleapis.com/auth/cloud-platform"],
    )
)
```

### Credential Management in Tools

#### Storing Credentials in State

```python
def login(username: str, password: str, tool_context: ToolContext) -> dict:
    """Authenticate user and store session token.

    Args:
        username: User's username.
        password: User's password.

    Returns:
        dict with authentication status.
    """
    # Authenticate with external service
    token = external_auth(username, password)

    # Store in session state for other tools to use
    tool_context.state["auth_token"] = token

    return {"status": "authenticated"}

def fetch_data(query: str, tool_context: ToolContext) -> dict:
    """Fetch data from authenticated API.

    Args:
        query: Data query.

    Returns:
        dict with query results.
    """
    token = tool_context.state.get("auth_token")
    if not token:
        return {"error": "Not authenticated. Please login first."}

    # Use token for API call
    return {"results": [...]}
```

## Action Confirmations

Require explicit user confirmation before executing sensitive tool operations. This is implemented through callbacks.

### Before-Tool Callback for Confirmation

```python
from google.adk.agents import Agent
from google.adk.events import Event
from google.genai import types

def require_confirmation(
    tool_name: str,
    tool_args: dict,
    tool_context
) -> dict | None:
    """Before-tool callback that requires user confirmation for destructive actions."""
    destructive_tools = ["delete_file", "send_email", "make_payment"]

    if tool_name in destructive_tools:
        # Return a message asking for confirmation
        # The framework will pause and ask the user
        return {
            "pending_confirmation": True,
            "message": f"About to execute {tool_name} with args: {tool_args}. Confirm?"
        }
    return None  # Allow execution without confirmation

agent = Agent(
    name="careful_agent",
    model="gemini-2.5-flash",
    instruction="""You are a careful assistant.
    For destructive operations, always explain what you're about to do.""",
    tools=[delete_file, send_email, make_payment],
    before_tool_callback=require_confirmation
)
```

### Tool-Level Confirmation Pattern

Build confirmation into the tool itself:

```python
def delete_record(record_id: str, confirmed: bool = False) -> dict:
    """Delete a record from the database.

    Args:
        record_id: The ID of the record to delete.
        confirmed: Must be True to actually delete. Defaults to False.

    Returns:
        dict with deletion status or confirmation prompt.
    """
    if not confirmed:
        return {
            "status": "pending",
            "message": f"Are you sure you want to delete record {record_id}? "
                       f"Call again with confirmed=True to proceed."
        }

    # Actually delete
    return {"status": "deleted", "record_id": record_id}
```

## Tool Performance

### Optimization Strategies

#### 1. Keep Tools Focused

```python
# Bad: One tool that does everything
def do_everything(action: str, data: str) -> dict:
    """Performs various actions based on the action parameter."""
    if action == "search": ...
    elif action == "create": ...
    elif action == "delete": ...

# Good: Separate focused tools
def search_items(query: str) -> dict:
    """Search for items matching a query."""
    ...

def create_item(name: str, data: str) -> dict:
    """Create a new item."""
    ...

def delete_item(item_id: str) -> dict:
    """Delete an item by ID."""
    ...
```

#### 2. Limit Tool Count

Too many tools confuse the LLM and increase token usage. Guidelines:
- **5-10 tools**: Optimal for most agents
- **10-20 tools**: Works but may need stronger instructions
- **20+ tools**: Consider splitting into sub-agents

```python
# Instead of one agent with 30 tools, use multi-agent delegation
billing_agent = Agent(
    name="billing",
    description="Handles billing operations",
    tools=[get_invoice, create_payment, refund, ...]  # 5-7 tools
)

support_agent = Agent(
    name="support",
    description="Handles technical support",
    tools=[search_docs, create_ticket, escalate, ...]  # 5-7 tools
)

root_agent = Agent(
    name="router",
    model="gemini-2.5-flash",
    tools=[billing_agent, support_agent]  # Delegates, not overloaded
)
```

#### 3. Use Async for I/O-Bound Tools

```python
import aiohttp
import asyncio

# Bad: Blocking I/O
def fetch_data_sync(url: str) -> dict:
    """Fetch data (blocks the event loop)."""
    import requests
    return requests.get(url).json()

# Good: Non-blocking async I/O
async def fetch_data(url: str) -> dict:
    """Fetch data from URL asynchronously."""
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            return await resp.json()
```

#### 4. Return Concise Results

The LLM processes everything you return. Large responses waste tokens and slow processing.

```python
# Bad: Returns entire database record with 50 fields
def get_user(user_id: str) -> dict:
    """Get user info."""
    return database.get_full_record(user_id)  # 50+ fields

# Good: Return only relevant fields
def get_user(user_id: str) -> dict:
    """Get user's basic information.

    Args:
        user_id: The user's ID.

    Returns:
        dict with name, email, and account status.
    """
    record = database.get_full_record(user_id)
    return {
        "name": record["name"],
        "email": record["email"],
        "status": record["account_status"]
    }
```

### Caching Patterns

```python
from functools import lru_cache
import time

# Simple in-memory cache for expensive lookups
@lru_cache(maxsize=100)
def get_product_details(product_id: str) -> dict:
    """Get product details (cached)."""
    return expensive_api_call(product_id)

# Time-based cache using state
def get_exchange_rate(
    from_currency: str,
    to_currency: str,
    tool_context: ToolContext
) -> dict:
    """Get current exchange rate between currencies.

    Args:
        from_currency: Source currency code (e.g., "USD").
        to_currency: Target currency code (e.g., "EUR").

    Returns:
        dict with exchange rate and timestamp.
    """
    cache_key = f"rate_{from_currency}_{to_currency}"
    cached = tool_context.state.get(cache_key)

    if cached and time.time() - cached["timestamp"] < 300:  # 5 min cache
        return cached

    # Fetch fresh rate
    rate = fetch_exchange_rate(from_currency, to_currency)
    result = {
        "from": from_currency,
        "to": to_currency,
        "rate": rate,
        "timestamp": time.time()
    }
    tool_context.state[cache_key] = result
    return result
```

## Best Practices

### Docstrings Are Your Interface
The LLM reads your docstrings to decide when and how to call tools. Invest heavily in clear, accurate docstrings. Include:
- What the tool does
- When to use it (and when NOT to)
- What each parameter means
- What the return value contains

### Always Return Structured Data
Return `dict` with descriptive keys. Avoid returning raw strings or unstructured data that the LLM must parse.

### Handle Errors Gracefully
Return error information in the result instead of raising exceptions. Unhandled exceptions break the agent loop.

```python
def risky_operation(input_data: str) -> dict:
    """Perform a risky operation.

    Args:
        input_data: Input to process.

    Returns:
        dict with result or error information.
    """
    try:
        result = process(input_data)
        return {"status": "success", "result": result}
    except ValueError as e:
        return {"status": "error", "message": f"Invalid input: {e}"}
    except ConnectionError:
        return {"status": "error", "message": "Service unavailable, try again later"}
```

### Use Type Hints Everywhere
Type hints generate the JSON schema that the LLM uses. Missing hints mean missing schema, which means the LLM guesses.

### Provide Default Values for Optional Parameters
This reduces friction for the LLM and makes tools easier to call.

### Test Tools Independently
Test your tool functions as regular Python/TypeScript functions before integrating them into agents.

### Prefer Agent Delegation Over Tool Overload
If an agent needs more than 10-15 tools, split into sub-agents with focused toolsets.

## Common Pitfalls

### Missing or Poor Docstrings
```python
# WRONG: LLM has no idea what this does
def process(x):
    return do_stuff(x)

# RIGHT: LLM knows exactly when and how to use this
def process_order(order_id: str) -> dict:
    """Process a pending customer order, charging payment and initiating shipment.

    Args:
        order_id: The unique order identifier (e.g., "ORD-12345").

    Returns:
        dict with 'status', 'tracking_number', and 'estimated_delivery'.
    """
    return do_stuff(order_id)
```

### Raising Exceptions Instead of Returning Errors
```python
# WRONG: Breaks the agent loop
def divide(a: float, b: float) -> float:
    return a / b  # ZeroDivisionError kills the agent

# RIGHT: Returns error info the LLM can work with
def divide(a: float, b: float) -> dict:
    """Divide two numbers.

    Args:
        a: Numerator.
        b: Denominator (must not be zero).

    Returns:
        dict with 'result' or 'error'.
    """
    if b == 0:
        return {"error": "Cannot divide by zero"}
    return {"result": a / b}
```

### Including tool_context in Docstring Args
```python
# WRONG: LLM tries to pass tool_context as a parameter
def my_tool(query: str, tool_context: ToolContext) -> dict:
    """Search for data.

    Args:
        query: Search query.
        tool_context: The tool context.  # <-- WRONG, remove this
    """

# RIGHT: Omit tool_context from docstring
def my_tool(query: str, tool_context: ToolContext) -> dict:
    """Search for data.

    Args:
        query: Search query.
    """
```

### MCP Server Not Found / Connection Errors
```python
# WRONG: npx not installed or MCP server package not available
MCPToolset(
    connection_params=StdioServerParameters(
        command="npx",
        args=["-y", "@modelcontextprotocol/server-filesystem", "/tmp"],
    )
)

# FIX: Ensure Node.js and npx are installed
# npm install -g npx  (or use full path to npx)
# Verify: npx --version
```

### OpenAPI Spec Mismatches
```python
# WRONG: Spec references localhost but agent runs in cloud
with open("openapi.yaml") as f:
    toolset = OpenAPIToolset(spec_str=f.read(), spec_str_type="yaml")
# Spec has: servers: [{url: "http://localhost:3000"}]

# RIGHT: Rewrite the `servers` entry in the spec before constructing
# the toolset, or edit the YAML to point at the production URL.
# <!-- VERIFY: whether OpenAPIToolset exposes a base_url kwarg -->
import yaml
with open("openapi.yaml") as f:
    spec = yaml.safe_load(f)
spec["servers"] = [{"url": "https://api.production.example.com"}]
toolset = OpenAPIToolset(
    spec_str=yaml.safe_dump(spec),
    spec_str_type="yaml",
)
```

### Too Many Tools Confusing the LLM
```python
# WRONG: Agent has 30 tools, LLM picks wrong ones
agent = Agent(tools=[tool1, tool2, ..., tool30])

# RIGHT: Split into focused sub-agents
agent = Agent(
    tools=[billing_sub_agent, support_sub_agent, data_sub_agent]
)
```

## References

- [Function Tools](https://google.github.io/adk-docs/tools-custom/function-tools/)
- [MCP Tools](https://google.github.io/adk-docs/tools-custom/mcp-tools/)
- [OpenAPI Tools](https://google.github.io/adk-docs/tools-custom/openapi-tools/)
- [Authentication](https://google.github.io/adk-docs/tools-custom/authentication/)
- [Tools Overview](https://google.github.io/adk-docs/tools/)
- [ADK Documentation](https://google.github.io/adk-docs)
- [MCP Protocol Specification](https://modelcontextprotocol.io/)
- [MCP Server Registry](https://github.com/modelcontextprotocol/servers)
