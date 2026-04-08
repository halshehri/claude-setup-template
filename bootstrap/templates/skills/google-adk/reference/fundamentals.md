# ADK Fundamentals

Use this skill when building agents with Google's Agent Development Kit (ADK). This covers installation, core concepts, project structure, and getting started patterns.

## When to Use This Skill

- Starting a new ADK project
- Understanding ADK architecture and primitives
- Setting up development environment
- Creating basic agents
- Questions about ADK concepts (Agent, Tool, Session, Memory, etc.)

## Core Concepts

ADK is built around these key primitives:

### Agent Types
1. **LlmAgent** (`Agent`): Uses LLMs for reasoning, dynamic decision-making, and tool usage
2. **Workflow Agents**: Deterministic execution flow controllers
   - `SequentialAgent`: Execute sub-agents in sequence
   - `ParallelAgent`: Execute sub-agents concurrently
   - `LoopAgent`: Repeat execution until condition met
3. **Custom Agents**: Extend `BaseAgent` for specialized logic

### Core Primitives
- **Tool**: Gives agents abilities (API calls, code execution, search, etc.)
- **Session**: Tracks a single conversation context
- **State**: Working memory within a session
- **Memory**: Long-term recall across multiple sessions
- **Event**: Communication unit (user message, agent reply, tool use)
- **Runner**: Orchestrates execution flow and manages sessions
- **Callbacks**: Custom code at specific execution points
- **Artifact**: File/binary data management for sessions

### Key Capabilities
- Multi-agent system design (hierarchical coordination)
- Rich tool ecosystem (function tools, agent tools, built-in tools)
- Flexible orchestration (workflow + LLM-driven routing)
- Native streaming support (text + audio bidirectional)
- Built-in evaluation framework
- Broad LLM support (Gemini, Claude, Vertex AI, Ollama, etc.)

## Project Setup

### Python Setup

```bash
# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
# or .venv\Scripts\activate  # Windows

# Install ADK
pip install google-adk

# Create project structure
mkdir my-adk-agent
cd my-adk-agent
```

**Required project structure:**
```
my-adk-agent/
├── .env              # API keys and config
├── __init__.py       # Package initialization
└── agent.py          # Agent definition
```

**agent.py - Basic structure:**
```python
from google.adk.agents import Agent

# Define your agent(s)
root_agent = Agent(
    name="my_agent",
    model="gemini-2.5-flash",
    description="Brief description of what this agent does",
    instruction="Clear instructions for agent behavior"
)
```

**__init__.py - Package setup:**
```python
from .agent import root_agent

__all__ = ["root_agent"]
```

**.env - Configuration:**
```
# For Google AI (Gemini)
GOOGLE_API_KEY=your_api_key_here

# OR for Vertex AI
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_CLOUD_LOCATION=us-central1
```

### TypeScript Setup

```bash
mkdir my-adk-agent
cd my-adk-agent
npm init -y
npm install @google/adk @google/adk-devtools
npm install -D typescript

# Create tsconfig.json
```

**tsconfig.json:**
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "esModuleInterop": true,
    "strict": true,
    "skipLibCheck": true
  }
}
```

**agent.ts:**
```typescript
import { LlmAgent } from '@google/adk';

export const rootAgent = new LlmAgent({
    name: 'my_agent',
    model: 'gemini-2.5-flash',
    description: 'Brief description',
    instruction: 'Clear instructions for agent behavior',
});
```

### Go Setup

```bash
mkdir my-adk-agent
cd my-adk-agent
go mod init my-adk-agent
go get google.golang.org/adk
```

### Java Setup

**pom.xml:**
```xml
<dependency>
    <groupId>com.google.adk</groupId>
    <artifactId>google-adk</artifactId>
    <version>0.5.0</version>
</dependency>
```

**build.gradle:**
```gradle
dependencies {
    implementation 'com.google.adk:google-adk:0.5.0'
}
```

## Authentication Patterns

### Google AI (Gemini API)
```python
# .env file
GOOGLE_API_KEY=your_key_from_ai_studio

# Get key from: https://aistudio.google.com/app/apikey
```

### Vertex AI
```python
# .env file
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_CLOUD_LOCATION=us-central1

# Authenticate via gcloud
# gcloud auth login
# gcloud auth application-default login
```

## Running Agents

### Three Ways to Run

1. **Web Interface** (Interactive Dev UI)
```bash
adk web <agents_directory>   # e.g., adk web .
# Opens browser at http://localhost:8000
```

2. **Command Line** (Terminal chat)
```bash
adk run <agent_directory>    # e.g., adk run ./my_agent
```

3. **API Server** (Programmatic access)
```bash
adk api_server <agents_directory>   # e.g., adk api_server .
# Starts server at http://localhost:8000
```

### Programmatic Execution (Python)

```python
import asyncio
from google.adk.agents import Agent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types

# Define agent
agent = Agent(
    name="my_agent",
    model="gemini-2.5-flash",
    instruction="You are a helpful assistant."
)

# Setup session management
session_service = InMemorySessionService()
runner = Runner(
    agent=agent,
    app_name="my_app",
    session_service=session_service
)

# Create session
APP_NAME = "my_app"
USER_ID = "user123"
SESSION_ID = "session123"

session = await session_service.create_session(
    app_name=APP_NAME,
    user_id=USER_ID,
    session_id=SESSION_ID
)

# Send message
async def interact():
    user_content = types.Content(
        role='user',
        parts=[types.Part(text="Hello!")]
    )

    async for event in runner.run_async(
        user_id=USER_ID,
        session_id=SESSION_ID,
        new_message=user_content
    ):
        if event.is_final_response() and event.content:
            print(event.content.parts[0].text)

asyncio.run(interact())
```

## Model Selection

### Common Model Choices

```python
# Fast, efficient (recommended for most use cases)
model="gemini-2.5-flash"

# More capable, slower
model="gemini-2.5-pro"

# Thinking/reasoning model (use gemini-2.5-pro with a planner)
model="gemini-2.5-pro"

# Claude (via Anthropic)
model="claude-sonnet-4-20250514"

# Vertex AI hosted
model="publishers/google/models/gemini-2.0-flash-001"
```

### Model Configuration

```python
from google.genai import types

agent = Agent(
    model="gemini-2.5-flash",
    generate_content_config=types.GenerateContentConfig(
        temperature=0.7,  # Randomness (0=deterministic, 1=creative)
        max_output_tokens=1024,
        top_p=0.95,
        top_k=40
    )
)
```

## Basic Agent Pattern

```python
from google.adk.agents import Agent

# 1. Define tools (if needed)
def get_weather(city: str) -> dict:
    """Get weather for a city."""
    return {"temp": "72F", "condition": "sunny"}

# 2. Create agent
agent = Agent(
    name="weather_agent",
    model="gemini-2.5-flash",
    description="Provides weather information",
    instruction="""You are a weather assistant.
    When user asks about weather:
    1. Use get_weather tool
    2. Format response clearly
    3. Include temperature and conditions
    """,
    tools=[get_weather]  # Automatically wrapped as FunctionTool
)

# 3. Root agent must be named 'root_agent'
root_agent = agent
```

## Common Patterns

### Pattern: Simple Q&A Agent
```python
qa_agent = Agent(
    name="qa_agent",
    model="gemini-2.5-flash",
    instruction="Answer questions concisely and accurately."
)
```

### Pattern: Agent with Multiple Tools
```python
def calculator(expression: str) -> float:
    """Evaluate math expression."""
    return eval(expression)

def search(query: str) -> str:
    """Search the web."""
    return f"Results for: {query}"

multi_tool_agent = Agent(
    name="assistant",
    model="gemini-2.5-flash",
    instruction="""You are a helpful assistant with tools.
    Use calculator for math, search for information.""",
    tools=[calculator, search]
)
```

### Pattern: Structured Output
```python
from pydantic import BaseModel, Field

class UserInfo(BaseModel):
    name: str = Field(description="User's name")
    age: int = Field(description="User's age")

structured_agent = Agent(
    name="info_extractor",
    model="gemini-2.5-flash",
    instruction="Extract user information as JSON",
    output_schema=UserInfo,
    output_key="extracted_info"  # Store in state
)
```

### Pattern: Stateless Agent
```python
stateless_agent = Agent(
    name="stateless",
    model="gemini-2.5-flash",
    instruction="Process each request independently",
    include_contents='none'  # No conversation history
)
```

## Best Practices

### Instructions
- **Be specific**: Clear, unambiguous instructions
- **Use examples**: Few-shot learning improves accuracy
- **Markdown formatting**: Use headers, lists for readability
- **Tool guidance**: Explain when/why to use each tool
- **State variables**: Use `{var}` syntax for dynamic values

### Agent Naming
- **Unique names**: Each agent needs distinct identifier
- **Descriptive**: Reflect agent's purpose (e.g., `billing_agent`)
- **Avoid reserved**: Don't use `user` as agent name
- **Root agent**: Main agent must be named `root_agent`

### Tool Design
- **Single responsibility**: One tool, one clear purpose
- **Good docstrings**: LLM uses these to decide when to call
- **Return objects**: Always return dict/map (except Python can return primitives)
- **Error handling**: Return error info in result, don't just raise

### Session Management
- **InMemorySessionService**: Development and testing
- **Persistent storage**: Production requires durable backend
- **Session IDs**: Use meaningful identifiers for debugging

## Common Pitfalls

### Forgetting root_agent
```python
# Wrong - will fail
my_agent = Agent(name="agent", ...)

# Correct
root_agent = Agent(name="agent", ...)
```

### Missing API key
```python
# Error: No authentication configured
# Fix: Set GOOGLE_API_KEY or use Vertex AI auth
```

### Tools without docstrings
```python
# Wrong - LLM doesn't know when to use
def search(query):
    return results

# Correct
def search(query: str) -> str:
    """Search the web for information about a topic."""
    return results
```

### Not handling tool errors
```python
# Wrong - exceptions break execution
def divide(a: float, b: float) -> float:
    return a / b  # Raises on b=0

# Correct
def divide(a: float, b: float) -> dict:
    """Divide two numbers."""
    if b == 0:
        return {"error": "Cannot divide by zero"}
    return {"result": a / b}
```

## Environment Variables

```bash
# Google AI
GOOGLE_API_KEY=...

# Vertex AI
GOOGLE_CLOUD_PROJECT=...
GOOGLE_CLOUD_LOCATION=...

# Anthropic Claude
ANTHROPIC_API_KEY=...

# OpenAI
OPENAI_API_KEY=...

# Development
ADK_DEBUG=true
ADK_LOG_LEVEL=INFO
```

## Quick Reference Commands

```bash
# Start dev UI
adk web .

# CLI chat
adk run ./my_agent

# API server
adk api_server --port 8000 .

# Check installation
pip show google-adk

# Update
pip install --upgrade google-adk
```

## References

- [ADK Documentation](https://google.github.io/adk-docs)
- [Technical Overview](https://google.github.io/adk-docs/get-started/about/)
- [Python Quickstart](https://google.github.io/adk-docs/get-started/python/)
- [Multi-tool Agent Tutorial](https://google.github.io/adk-docs/get-started/quickstart/)
