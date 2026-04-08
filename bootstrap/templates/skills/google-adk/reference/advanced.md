# ADK Advanced Features

Use this reference for advanced ADK capabilities including MCP integration, A2A protocol, streaming (text and bidirectional), artifacts, plugins, and grounding.

## When to Use This Reference

- Integrating MCP (Model Context Protocol) servers with ADK agents
- Setting up agent-to-agent communication via the A2A protocol
- Implementing streaming responses (SSE or bidirectional live audio/video)
- Managing binary data and files with the artifacts system
- Using plugins for agent behavior enhancement (reflect and retry)
- Adding grounding capabilities (Google Search, Vertex AI Search)

## MCP (Model Context Protocol)

### What is MCP

MCP (Model Context Protocol) is an open standard that provides a unified way to connect AI models with external data sources and tools. It defines a client-server architecture where:

- **MCP Servers** expose tools, resources, and prompts via a standardized protocol
- **MCP Clients** connect to servers and make those capabilities available to agents
- **Transport**: Communication happens over stdio (local processes) or SSE/HTTP (remote servers)

ADK provides first-class MCP support through `MCPToolset`, which acts as an MCP client. This lets ADK agents use any MCP-compatible server's tools without writing custom integration code.

### Using MCP Servers with ADK

#### MCPToolset Configuration

```python
from google.adk.tools.mcp_tool import MCPToolset, StdioServerParameters

# Connect to a local MCP server via stdio
tools, exit_stack = await MCPToolset.from_server(
    connection_params=StdioServerParameters(
        command="npx",
        args=["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"],
    )
)

agent = Agent(
    name="file_agent",
    model="gemini-2.5-flash",
    instruction="You can read and write files using the filesystem tools.",
    tools=tools,
)
```

#### StdioServerParameters (Local MCP Servers)

Use this when the MCP server runs as a local subprocess communicating over stdin/stdout.

```python
from google.adk.tools.mcp_tool import MCPToolset, StdioServerParameters

# Filesystem server
filesystem_params = StdioServerParameters(
    command="npx",
    args=["-y", "@modelcontextprotocol/server-filesystem", "/allowed/path"],
)

# SQLite database server
sqlite_params = StdioServerParameters(
    command="npx",
    args=["-y", "@modelcontextprotocol/server-sqlite", "path/to/database.db"],
)

# Python-based MCP server
python_params = StdioServerParameters(
    command="python",
    args=["-m", "my_mcp_server"],
    env={"MY_API_KEY": "..."},  # Optional environment variables
)

# UV-based MCP server
uv_params = StdioServerParameters(
    command="uvx",
    args=["my-mcp-server", "--some-flag"],
)
```

#### SseServerParams (Remote MCP Servers)

Use this when the MCP server is running remotely and communicating over HTTP Server-Sent Events.

```python
from google.adk.tools.mcp_tool import MCPToolset
from google.adk.tools.mcp_tool.mcp_session_manager import SseServerParams

# Remote MCP server via SSE
remote_params = SseServerParams(
    url="http://localhost:8080/sse",
    headers={"Authorization": "Bearer my-token"},  # Optional auth headers
)

tools, exit_stack = await MCPToolset.from_server(
    connection_params=remote_params
)
```

#### Full Agent Example with MCP

```python
import asyncio
from google.adk.agents import Agent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.tools.mcp_tool import MCPToolset, StdioServerParameters
from google.genai import types


async def create_agent():
    """Create an agent with MCP filesystem tools."""
    tools, exit_stack = await MCPToolset.from_server(
        connection_params=StdioServerParameters(
            command="npx",
            args=["-y", "@modelcontextprotocol/server-filesystem", "/tmp/workspace"],
        )
    )

    agent = Agent(
        name="file_manager",
        model="gemini-2.5-flash",
        instruction="""You are a file management assistant.
        Use the available tools to read, write, and manage files.
        Always confirm actions before modifying files.""",
        tools=tools,
    )

    return agent, exit_stack


async def main():
    agent, exit_stack = await create_agent()

    session_service = InMemorySessionService()
    runner = Runner(agent=agent, app_name="file_app", session_service=session_service)

    session = await session_service.create_session(
        app_name="file_app", user_id="user1"
    )

    try:
        message = types.Content(
            role="user", parts=[types.Part(text="List files in the workspace")]
        )
        async for event in runner.run_async(
            user_id="user1", session_id=session.id, new_message=message
        ):
            if event.is_final_response() and event.content:
                print(event.content.parts[0].text)
    finally:
        await exit_stack.aclose()  # Clean up MCP connection


asyncio.run(main())
```

#### TypeScript MCPToolset

```typescript
import { LlmAgent } from '@google/adk';
import { MCPToolset, StdioServerParameters } from '@google/adk';

const [tools, cleanup] = await MCPToolset.fromServer(
    new StdioServerParameters({
        command: 'npx',
        args: ['-y', '@modelcontextprotocol/server-filesystem', '/tmp'],
    })
);

const agent = new LlmAgent({
    name: 'file_agent',
    model: 'gemini-2.5-flash',
    instruction: 'Manage files using the available tools.',
    tools: tools,
});

// Remember to call cleanup() when done
```

#### Filtering MCP Tools

```python
# Only use specific tools from the MCP server
tools, exit_stack = await MCPToolset.from_server(
    connection_params=StdioServerParameters(
        command="npx",
        args=["-y", "@modelcontextprotocol/server-filesystem", "/tmp"],
    ),
    tool_filter=["read_file", "list_directory"],  # Only these tools
)
```

### Common MCP Servers

| Server | Package | Purpose |
|--------|---------|---------|
| Filesystem | `@modelcontextprotocol/server-filesystem` | Read/write local files |
| SQLite | `@modelcontextprotocol/server-sqlite` | Query SQLite databases |
| PostgreSQL | `@modelcontextprotocol/server-postgres` | Query PostgreSQL databases |
| GitHub | `@modelcontextprotocol/server-github` | GitHub API access |
| Google Maps | `@modelcontextprotocol/server-google-maps` | Maps and geocoding |
| Brave Search | `@modelcontextprotocol/server-brave-search` | Web search |
| Memory | `@modelcontextprotocol/server-memory` | Persistent key-value memory |
| Puppeteer | `@modelcontextprotocol/server-puppeteer` | Browser automation |

#### Configuration Examples

```python
# GitHub MCP server
github_tools, _ = await MCPToolset.from_server(
    connection_params=StdioServerParameters(
        command="npx",
        args=["-y", "@modelcontextprotocol/server-github"],
        env={"GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_..."},
    )
)

# PostgreSQL MCP server
postgres_tools, _ = await MCPToolset.from_server(
    connection_params=StdioServerParameters(
        command="npx",
        args=[
            "-y",
            "@modelcontextprotocol/server-postgres",
            "postgresql://user:pass@localhost:5432/mydb",
        ],
    )
)
```

## A2A Protocol (Agent-to-Agent)

### What is A2A

A2A (Agent-to-Agent) is an open protocol by Google that enables communication between AI agents, even when they are built with different frameworks. Key concepts:

- **Agent Card**: A JSON metadata document describing an agent's capabilities, skills, and endpoint (similar to a service manifest)
- **Task**: A unit of work exchanged between agents (with states: submitted, working, completed, failed, canceled)
- **Message**: Communication between agents within a task (with roles: user, agent)
- **Artifact**: Files or data produced by a task
- **Push Notifications**: Webhook-based notifications for long-running tasks

A2A complements MCP: while MCP connects agents to tools/data, A2A connects agents to other agents. Use A2A when you need:
- Cross-framework agent communication (e.g., ADK agent talking to LangChain agent)
- Distributed multi-agent systems across services
- Agent discovery and capability advertisement

### A2A Server (Exposing an ADK Agent)

<!-- VERIFY: A2A server API surface. The ADK docs describe this via
     quickstart guides; the exact helper names and imports should be
     confirmed against https://adk.dev/a2a/ before relying on this
     snippet. The commonly-referenced helper is `to_a2a()`, which wraps
     an ADK agent into an A2A-compatible app. -->

```python
# TODO: Confirm exact import path for to_a2a() in your installed ADK version.
# Example scaffolding:
#
# from google.adk.agents import Agent
# from google.adk.a2a import to_a2a  # <-- verify import
#
# expense_agent = Agent(
#     name="expense_agent",
#     model="gemini-2.5-flash",
#     instruction="You are an expense reporting agent...",
#     tools=[create_expense, get_expenses, approve_expense],
# )
#
# # Wrap the agent as an A2A-compatible ASGI app and serve it.
# a2a_app = to_a2a(expense_agent)
# # Run with uvicorn / any ASGI server:
# #   uvicorn my_module:a2a_app --host 0.0.0.0 --port 8001
```

#### Agent Card Discovery

A2A servers expose their Agent Card at `/.well-known/agent.json`:

```json
{
  "name": "Expense Agent",
  "description": "Handles expense report creation and management",
  "url": "http://localhost:8001",
  "version": "1.0.0",
  "capabilities": {
    "streaming": true,
    "pushNotifications": false
  },
  "skills": [
    {
      "id": "create_expense",
      "name": "Create Expense Report",
      "description": "Creates a new expense report with line items"
    }
  ],
  "authentication": {
    "schemes": ["bearer"]
  }
}
```

### A2A Client (Consuming a Remote Agent)

<!-- VERIFY: A2A client API. The standard pattern is RemoteA2aAgent, which
     behaves as a regular ADK agent and can be placed in `sub_agents=` of
     a parent agent for LLM-driven delegation. Confirm import path against
     https://adk.dev/a2a/ for your ADK version. -->

```python
# Example scaffolding:
#
# from google.adk.agents import Agent
# from google.adk.agents.remote_a2a_agent import RemoteA2aAgent  # <-- verify
#
# expense_agent = RemoteA2aAgent(
#     name="expense_agent",
#     description="Remote expense reporting agent",
#     agent_card="http://localhost:8001/.well-known/agent.json",
# )
#
# orchestrator = Agent(
#     name="orchestrator",
#     model="gemini-2.5-flash",
#     instruction=(
#         "Coordinate business operations. Delegate expense-related "
#         "tasks to the expense_agent."
#     ),
#     sub_agents=[expense_agent],
# )
```

### A2A Patterns

#### Cross-Framework Communication

```
                        A2A Protocol
  ADK Agent  <------------------------->  LangChain Agent
  (Python)           HTTP/JSON              (Python)

  ADK Agent  <------------------------->  CrewAI Agent
  (Python)           HTTP/JSON              (Python)

  ADK Agent  <------------------------->  Custom Agent
  (TypeScript)       HTTP/JSON              (Any language)
```

Any framework that implements the A2A protocol can communicate with ADK agents. The protocol is language- and framework-agnostic.

#### Multi-Service Agent Architecture

```python
# Service A: ADK expense agent (port 8001)
# Service B: LangChain calendar agent (port 8002)
# Service C: Custom approval agent (port 8003)

# Orchestrator discovers and coordinates all agents
# <!-- VERIFY: RemoteA2aAgent import path for your ADK version -->
from google.adk.agents import Agent
from google.adk.agents.remote_a2a_agent import RemoteA2aAgent  # verify

expense_agent = RemoteA2aAgent(
    name="expense_agent",
    description="Handles expense reports and reimbursement",
    agent_card="http://expense-service:8001/.well-known/agent.json",
)
calendar_agent = RemoteA2aAgent(
    name="calendar_agent",
    description="Handles scheduling and availability",
    agent_card="http://calendar-service:8002/.well-known/agent.json",
)
approval_agent = RemoteA2aAgent(
    name="approval_agent",
    description="Handles approval workflows",
    agent_card="http://approval-service:8003/.well-known/agent.json",
)

orchestrator = Agent(
    name="business_orchestrator",
    model="gemini-2.5-flash",
    instruction="""Coordinate business workflows across services.
    Delegate to:
    - expense_agent: expense reports and reimbursement
    - calendar_agent: scheduling and availability
    - approval_agent: approval workflows
    """,
    sub_agents=[expense_agent, calendar_agent, approval_agent],
)
```

## Streaming

### Text Streaming

ADK supports streaming agent responses via Server-Sent Events (SSE). Instead of waiting for the full response, the client receives incremental updates.

#### Server-Side: Streaming Runner

```python
from google.adk.agents import Agent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types

agent = Agent(
    name="streaming_agent",
    model="gemini-2.5-flash",
    instruction="You are a helpful assistant.",
)

session_service = InMemorySessionService()
runner = Runner(
    agent=agent,
    app_name="streaming_app",
    session_service=session_service,
)

# Stream events as they are produced
async def stream_response(user_message: str, session_id: str):
    message = types.Content(
        role="user", parts=[types.Part(text=user_message)]
    )

    async for event in runner.run_async(
        user_id="user1",
        session_id=session_id,
        new_message=message,
    ):
        # Each event can contain partial content
        if event.content and event.content.parts:
            for part in event.content.parts:
                if part.text:
                    print(part.text, end="", flush=True)

        # Check for final response
        if event.is_final_response():
            print("\n[Stream complete]")
```

#### Serving SSE with FastAPI

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from google.adk.agents import Agent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types
import json

app = FastAPI()

agent = Agent(name="assistant", model="gemini-2.5-flash", instruction="Be helpful.")
session_service = InMemorySessionService()
runner = Runner(agent=agent, app_name="sse_app", session_service=session_service)


async def event_generator(user_id: str, session_id: str, message_text: str):
    """Generate SSE events from agent runner."""
    message = types.Content(role="user", parts=[types.Part(text=message_text)])

    async for event in runner.run_async(
        user_id=user_id, session_id=session_id, new_message=message
    ):
        event_data = {}

        if event.content and event.content.parts:
            for part in event.content.parts:
                if part.text:
                    event_data["text"] = part.text

        if event.is_final_response():
            event_data["final"] = True

        if event_data:
            yield f"data: {json.dumps(event_data)}\n\n"


@app.post("/chat/{session_id}")
async def chat(session_id: str, message: str):
    return StreamingResponse(
        event_generator("user1", session_id, message),
        media_type="text/event-stream",
    )
```

#### Client-Side SSE Handling (JavaScript)

```javascript
const eventSource = new EventSource('/chat/session123?message=Hello');

eventSource.onmessage = (event) => {
    const data = JSON.parse(event.data);

    if (data.text) {
        // Append incremental text to UI
        document.getElementById('response').textContent += data.text;
    }

    if (data.final) {
        eventSource.close();
    }
};

eventSource.onerror = (error) => {
    console.error('SSE error:', error);
    eventSource.close();
};
```

#### Using `adk api_server` with Streaming

```bash
# Start API server with streaming support
adk api_server --port 8000 .

# The /run_sse endpoint streams events
# POST /apps/{app_name}/users/{user_id}/sessions/{session_id}/run_sse
```

```python
import httpx

async with httpx.AsyncClient() as client:
    async with client.stream(
        "POST",
        "http://localhost:8000/apps/my_app/users/user1/sessions/s1/run_sse",
        json={"message": "Tell me a story"},
    ) as response:
        async for line in response.aiter_lines():
            if line.startswith("data: "):
                data = json.loads(line[6:])
                print(data.get("text", ""), end="")
```

### Bidirectional Streaming (Live)

ADK supports bidirectional (bidi) streaming for real-time voice and video interactions using Gemini's Live API. This enables:
- Live audio input and output
- Real-time video frame processing
- Simultaneous send/receive
- Low-latency conversational AI

#### Configuration

```python
from google.adk.agents import Agent
from google.genai import types

# Agent configured for live/bidi streaming
live_agent = Agent(
    name="voice_assistant",
    model="gemini-2.0-flash-live",  # Live-capable model
    instruction="You are a voice assistant. Respond naturally and conversationally.",
    generate_content_config=types.GenerateContentConfig(
        response_modalities=["AUDIO"],  # Output audio
        speech_config=types.SpeechConfig(
            voice_config=types.VoiceConfig(
                prebuilt_voice_config=types.PrebuiltVoiceConfig(
                    voice_name="Aoede"  # Voice options: Aoede, Charon, Fenrir, Kore, Puck
                )
            )
        ),
    ),
)
```

#### Live Runner (run_live + LiveRequestQueue)

The real ADK streaming API uses `runner.run_live()` (an async generator
of events) together with a `LiveRequestQueue` that you push audio/text
frames into. There is no separate `LiveRunner.connect/send_audio/receive`.

```python
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.agents.live_request_queue import LiveRequestQueue  # verify path
from google.genai import types

session_service = InMemorySessionService()
runner = Runner(
    agent=live_agent,
    app_name="voice_app",
    session_service=session_service,
)

session = await session_service.create_session(
    app_name="voice_app",
    user_id="user1",
    session_id="live_session_1",
)

# Queue for pushing live input (audio/text/video blobs) into the model
live_queue = LiveRequestQueue()

async def pump_audio(audio_chunks):
    for chunk in audio_chunks:
        live_queue.send_realtime(
            types.Blob(data=chunk, mime_type="audio/pcm")
        )
    live_queue.close()

# runner.run_live yields streaming events (partial text, audio, tool calls)
async for event in runner.run_live(
    user_id="user1",
    session_id=session.id,
    live_request_queue=live_queue,
):
    if event.content and event.content.parts:
        for part in event.content.parts:
            if part.text:
                print(part.text, end="", flush=True)
            if getattr(part, "inline_data", None):
                play_audio(part.inline_data.data)
```

<!-- VERIFY: exact import path for LiveRequestQueue and the live_request_queue
     kwarg name on runner.run_live() in your installed ADK version. -->

#### Response Modalities

```python
# Text only (default)
generate_content_config=types.GenerateContentConfig(
    response_modalities=["TEXT"],
)

# Audio only
generate_content_config=types.GenerateContentConfig(
    response_modalities=["AUDIO"],
)

# Both text and audio
generate_content_config=types.GenerateContentConfig(
    response_modalities=["TEXT", "AUDIO"],
)
```

#### Voice Options

Available voices for audio output:
- `Aoede` - Warm, expressive
- `Charon` - Deep, authoritative
- `Fenrir` - Energetic, youthful
- `Kore` - Calm, professional
- `Puck` - Playful, dynamic

### Streaming Events

Events emitted during streaming provide granular control:

```python
async for event in runner.run_async(
    user_id=user_id,
    session_id=session_id,
    new_message=message,
):
    # Event types you may encounter:
    if event.content:
        # Agent is producing content (text, audio, etc.)
        pass

    if event.actions and event.actions.tool_calls:
        # Agent is calling a tool
        for tool_call in event.actions.tool_calls:
            print(f"Calling tool: {tool_call.name}")

    if event.actions and event.actions.transfer_to_agent:
        # Agent is transferring to another agent
        print(f"Transferring to: {event.actions.transfer_to_agent}")

    if event.is_final_response():
        # This is the last event in the stream
        break
```

### Streaming Tools

Tools that produce incremental results can stream their output:

```python
from google.adk.tools import FunctionTool
from typing import AsyncIterator


async def streaming_search(query: str) -> AsyncIterator[dict]:
    """Search that yields results incrementally."""
    for i in range(5):
        yield {"result": f"Result {i+1} for: {query}"}
        await asyncio.sleep(0.5)  # Simulate latency


# Wrap as streaming tool
search_tool = FunctionTool(func=streaming_search)

agent = Agent(
    name="search_agent",
    model="gemini-2.5-flash",
    tools=[search_tool],
)
```

## Artifacts

### What are Artifacts

Artifacts are binary or file-like data associated with a session. They provide a way to:
- Store files generated by agents or tools (PDFs, images, CSVs, etc.)
- Pass binary data between agents in multi-agent systems
- Persist file outputs across interactions within a session
- Version file data (each save creates a new version)

Artifacts are identified by a filename (string) and stored as bytes. Each artifact can have multiple versions (0-indexed).

### Creating Artifacts

#### From Tools (Using ToolContext)

```python
from google.adk.tools import ToolContext


def generate_report(data: str, ctx: ToolContext) -> dict:
    """Generate a report and save as artifact.

    Args:
        data: The data to include in the report.
        ctx: Tool context (automatically injected by ADK).
    """
    # Generate report content
    report_content = f"Report: {data}\nGenerated at: {datetime.now()}"
    report_bytes = report_content.encode("utf-8")

    # Save as artifact
    version = ctx.save_artifact(
        filename="report.txt",
        artifact=types.Part.from_bytes(data=report_bytes, mime_type="text/plain"),
    )

    return {"status": "Report saved", "version": version}


def generate_chart(values: list[float], ctx: ToolContext) -> dict:
    """Generate a chart image and save as artifact."""
    import matplotlib.pyplot as plt
    import io

    fig, ax = plt.subplots()
    ax.bar(range(len(values)), values)

    buf = io.BytesIO()
    fig.savefig(buf, format="png")
    buf.seek(0)

    version = ctx.save_artifact(
        filename="chart.png",
        artifact=types.Part.from_bytes(data=buf.read(), mime_type="image/png"),
    )

    return {"status": "Chart saved", "version": version}


agent = Agent(
    name="report_agent",
    model="gemini-2.5-flash",
    instruction="Generate reports and charts when asked.",
    tools=[generate_report, generate_chart],
)
```

#### From Agents (Using CallbackContext)

```python
from google.adk.agents import Agent


def after_model_callback(callback_context, llm_response):
    """Save agent output as artifact after model generates it."""
    if llm_response and llm_response.content:
        text = llm_response.content.parts[0].text
        if "SAVE_AS_FILE:" in text:
            # Extract content to save
            filename, content = parse_save_directive(text)
            callback_context.save_artifact(
                filename=filename,
                artifact=types.Part.from_bytes(
                    data=content.encode(), mime_type="text/plain"
                ),
            )
    return llm_response


agent = Agent(
    name="writer_agent",
    model="gemini-2.5-flash",
    after_model_callback=after_model_callback,
)
```

### Retrieving Artifacts

#### From Tools

```python
def read_report(filename: str, ctx: ToolContext) -> dict:
    """Read a previously saved artifact.

    Args:
        filename: Name of the artifact file.
        ctx: Tool context.
    """
    # Get latest version
    artifact = ctx.load_artifact(filename=filename)
    if artifact is None:
        return {"error": f"Artifact '{filename}' not found"}

    content = artifact.inline_data.data.decode("utf-8")
    return {"filename": filename, "content": content}


def read_specific_version(filename: str, version: int, ctx: ToolContext) -> dict:
    """Read a specific version of an artifact."""
    artifact = ctx.load_artifact(filename=filename, version=version)
    if artifact is None:
        return {"error": f"Version {version} of '{filename}' not found"}

    return {"content": artifact.inline_data.data.decode("utf-8")}
```

#### Listing Artifacts

```python
def list_files(ctx: ToolContext) -> dict:
    """List all artifacts in the current session."""
    filenames = ctx.list_artifacts()
    return {"files": filenames}
```

#### Using Artifacts in Instructions

```python
# Reference artifact content in agent instructions via template syntax
agent = Agent(
    name="reviewer",
    model="gemini-2.5-flash",
    instruction="""Review the following document:
    {artifact.report.txt}

    Provide feedback on clarity and accuracy.""",
)
# {artifact.<filename>} is replaced with the artifact's text content at runtime
```

### Artifact Storage

#### InMemoryArtifactService (Development)

```python
from google.adk.artifacts import InMemoryArtifactService

artifact_service = InMemoryArtifactService()

runner = Runner(
    agent=agent,
    app_name="my_app",
    session_service=session_service,
    artifact_service=artifact_service,  # Pass to runner
)
```

**Characteristics:**
- Stored in process memory
- Lost when process restarts
- Fast, no external dependencies
- Suitable for development and testing

#### GcsArtifactService (Production - Google Cloud Storage)

```python
from google.adk.artifacts import GcsArtifactService

artifact_service = GcsArtifactService(
    bucket_name="my-agent-artifacts",
    # Optional: prefix for organization
    # path_prefix="app1/artifacts/"
)

runner = Runner(
    agent=agent,
    app_name="my_app",
    session_service=session_service,
    artifact_service=artifact_service,
)
```

**Characteristics:**
- Persistent storage in Google Cloud Storage
- Survives process restarts
- Scalable and durable
- Requires GCS bucket and credentials
- Suitable for production

#### Artifact Versioning

Every call to `save_artifact` with the same filename creates a new version:

```python
# Version 0
ctx.save_artifact(filename="draft.txt", artifact=part_v0)

# Version 1 (updated)
ctx.save_artifact(filename="draft.txt", artifact=part_v1)

# Version 2 (updated again)
ctx.save_artifact(filename="draft.txt", artifact=part_v2)

# Load latest (version 2)
latest = ctx.load_artifact(filename="draft.txt")

# Load specific version
v0 = ctx.load_artifact(filename="draft.txt", version=0)
v1 = ctx.load_artifact(filename="draft.txt", version=1)
```

## Plugins

### What are Plugins

Plugins are pre-built behavior modifiers that wrap agent execution with additional logic. They can intercept, modify, or retry agent actions without changing the agent's core configuration.

### Reflect and Retry Plugin

Plugins in ADK are registered **once on the Runner**, not on an Agent.
They apply globally to every agent, tool, and LLM call managed by that
runner. There is no `plugins=` argument on `Agent`.

```python
from google.adk.runners import InMemoryRunner
# <!-- VERIFY: exact import path and constructor args for the
#      reflect-and-retry plugin in your ADK version. -->
# from google.adk.plugins.reflect_and_retry_tool_plugin import (
#     ReflectAndRetryToolPlugin,
# )

runner = InMemoryRunner(
    agent=root_agent,
    app_name="resilient_app",
    plugins=[
        # ReflectAndRetryToolPlugin(max_retries=3),
    ],
)
```

**How it works:**
1. Agent calls a tool
2. Tool raises an error or returns an error result
3. Plugin intercepts the error
4. Plugin sends error context back to the LLM with a reflection prompt
5. LLM analyzes what went wrong and generates corrected tool call
6. Tool is called again with new parameters
7. Repeats up to `max_retries` times

### Creating Custom Plugins

Custom plugins subclass `BasePlugin` and are registered on the Runner.

```python
# <!-- VERIFY: exact BasePlugin import path and hook method names
#      for your ADK version. Plugins build on the callback system and
#      are registered at runner construction time, not on the Agent. -->
from google.adk.plugins.base_plugin import BasePlugin
from google.adk.runners import InMemoryRunner


class CountInvocationPlugin(BasePlugin):
    """Example plugin that counts invocations."""

    def __init__(self):
        self.count = 0


runner = InMemoryRunner(
    agent=root_agent,
    app_name="logged_app",
    plugins=[CountInvocationPlugin()],
)
```

## Grounding

### What is Grounding

Grounding connects LLM responses to real-world, up-to-date information sources. This reduces hallucination and ensures factual accuracy. ADK supports two grounding mechanisms:

1. **Google Search Grounding**: Uses Google Search results to ground responses
2. **Vertex AI Search Grounding**: Uses your own data indexed in Vertex AI Search

### Google Search Grounding

```python
from google.adk.agents import Agent
from google.adk.tools import google_search

# Using Google Search as a grounding tool
agent = Agent(
    name="grounded_agent",
    model="gemini-2.5-flash",
    instruction="""You are a research assistant.
    Use Google Search to find current, accurate information.
    Always cite your sources.""",
    tools=[google_search],
)
```

#### Google Search as Grounding (Gemini Built-in)

```python
from google.genai import types

agent = Agent(
    name="search_grounded",
    model="gemini-2.5-flash",
    instruction="Answer questions using current information from Google Search.",
    generate_content_config=types.GenerateContentConfig(
        tools=[
            types.Tool(
                google_search=types.GoogleSearch()
            )
        ]
    ),
)
```

#### Dynamic Grounding with Threshold

```python
from google.genai import types

agent = Agent(
    name="dynamic_grounded",
    model="gemini-2.5-flash",
    instruction="Answer questions accurately. Use search when needed.",
    generate_content_config=types.GenerateContentConfig(
        tools=[
            types.Tool(
                google_search=types.GoogleSearchRetrieval(
                    dynamic_retrieval_config=types.DynamicRetrievalConfig(
                        mode=types.DynamicRetrievalConfigMode.MODE_DYNAMIC,
                        dynamic_threshold=0.5,  # 0=always search, 1=never search
                    )
                )
            )
        ]
    ),
)
# With dynamic threshold:
# - Lower values (0.0-0.3): Search more aggressively
# - Higher values (0.7-1.0): Search only when very uncertain
# - 0.5: Balanced default
```

### Vertex AI Search Grounding

Ground responses using your own data indexed in Vertex AI Search (Enterprise Search).

```python
from google.genai import types

agent = Agent(
    name="enterprise_grounded",
    model="gemini-2.5-flash",
    instruction="""Answer questions using the company knowledge base.
    Cite specific documents when possible.""",
    generate_content_config=types.GenerateContentConfig(
        tools=[
            types.Tool(
                retrieval=types.Retrieval(
                    vertex_ai_search=types.VertexAISearch(
                        datastore=f"projects/{PROJECT_ID}/locations/{LOCATION}/collections/default_collection/dataStores/{DATASTORE_ID}"
                    )
                )
            )
        ]
    ),
)
```

#### Setting Up Vertex AI Search

1. Create a Vertex AI Search data store in Google Cloud Console
2. Ingest your data (documents, websites, structured data)
3. Get the data store resource path
4. Configure the agent with the data store path

```python
# Full resource path format:
# projects/{project}/locations/{location}/collections/default_collection/dataStores/{datastore_id}

PROJECT_ID = "my-gcp-project"
LOCATION = "global"  # or specific region
DATASTORE_ID = "my-company-docs_1234567890"

datastore_path = (
    f"projects/{PROJECT_ID}/locations/{LOCATION}"
    f"/collections/default_collection/dataStores/{DATASTORE_ID}"
)
```

### Combining Grounding Sources

```python
# Agent with both search and enterprise grounding
agent = Agent(
    name="hybrid_grounded",
    model="gemini-2.5-flash",
    instruction="""Answer questions using:
    1. Company knowledge base for internal information
    2. Google Search for public/current information
    """,
    generate_content_config=types.GenerateContentConfig(
        tools=[
            types.Tool(google_search=types.GoogleSearch()),
            types.Tool(
                retrieval=types.Retrieval(
                    vertex_ai_search=types.VertexAISearch(
                        datastore=datastore_path
                    )
                )
            ),
        ]
    ),
)
```

## Best Practices

### MCP Integration
- **Cleanup connections**: Always use `exit_stack.aclose()` or context managers to close MCP connections
- **Filter tools**: Use `tool_filter` to only expose tools the agent needs
- **Handle server failures**: MCP servers can crash; wrap in try/except and implement reconnection
- **Prefer stdio for local**: Stdio is simpler and more reliable for local servers
- **Prefer SSE for remote**: SSE handles network concerns better for distributed setups

### A2A Protocol
- **Agent Cards matter**: Write clear, descriptive agent cards -- other agents use them to decide when to delegate
- **Version your agents**: Include version in agent cards to manage compatibility
- **Handle task states**: Tasks can be submitted, working, completed, failed, or canceled
- **Use streaming for long tasks**: Enable streaming in A2A for long-running operations

### Streaming
- **Prefer streaming for UX**: Users perceive faster responses with streaming text
- **Handle partial content**: UI must gracefully handle incomplete text chunks
- **Bidi requires live model**: Use `gemini-2.0-flash-live` or similar live-capable model for bidirectional
- **Buffer audio carefully**: Audio streaming requires proper buffering to avoid choppy playback

### Artifacts
- **Use descriptive filenames**: `quarterly-report-2025-q1.pdf` not `file1.pdf`
- **Check for existence**: Always handle the case where `load_artifact` returns `None`
- **Version intentionally**: Each save creates a new version; design for this
- **Choose storage wisely**: InMemory for dev, GCS for production

### Grounding
- **Tune dynamic threshold**: Start at 0.5, adjust based on your use case
- **Cite sources**: Instruct agents to reference grounding sources in responses
- **Vertex AI Search for private data**: Never use Google Search for proprietary information
- **Index quality matters**: Grounding is only as good as the indexed data

## Common Pitfalls

### MCP connection not cleaned up
```python
# Wrong - connection leak
tools, exit_stack = await MCPToolset.from_server(...)
agent = Agent(tools=tools)
# exit_stack never closed!

# Correct - clean up in finally
try:
    tools, exit_stack = await MCPToolset.from_server(...)
    agent = Agent(tools=tools)
    # ... use agent ...
finally:
    await exit_stack.aclose()
```

### Using non-live model for bidi streaming
```python
# Wrong - standard model does not support bidi
agent = Agent(
    model="gemini-2.5-flash",  # Not a live model
    generate_content_config=types.GenerateContentConfig(
        response_modalities=["AUDIO"],
    ),
)

# Correct - use live model
agent = Agent(
    model="gemini-2.0-flash-live",  # Live-capable model
    generate_content_config=types.GenerateContentConfig(
        response_modalities=["AUDIO"],
    ),
)
```

### Artifacts without artifact_service
```python
# Wrong - no artifact service configured
runner = Runner(
    agent=agent,
    app_name="app",
    session_service=session_service,
    # Missing artifact_service!
)
# Tool calls to save_artifact will fail

# Correct - include artifact service
runner = Runner(
    agent=agent,
    app_name="app",
    session_service=session_service,
    artifact_service=InMemoryArtifactService(),  # Required for artifacts
)
```

### A2A agent card missing skills
```python
# Wrong - no skills advertised
agent_card = {
    "name": "My Agent",
    "url": "http://localhost:8001",
}
# Other agents won't know what this agent can do

# Correct - descriptive skills
agent_card = {
    "name": "My Agent",
    "url": "http://localhost:8001",
    "skills": [
        {
            "id": "data_analysis",
            "name": "Analyze Data",
            "description": "Performs statistical analysis on CSV and JSON datasets",
        }
    ],
}
```

### Grounding tool conflict with agent tools
```python
# Potentially problematic - mixing grounding with many tools
agent = Agent(
    model="gemini-2.5-flash",
    tools=[tool1, tool2, tool3, tool4, tool5],
    generate_content_config=types.GenerateContentConfig(
        tools=[types.Tool(google_search=types.GoogleSearch())]
    ),
)
# Too many tools + grounding can confuse the model

# Better - dedicated grounded agent in a multi-agent setup
grounded_agent = Agent(
    name="researcher",
    model="gemini-2.5-flash",
    description="Searches for current information using Google Search",
    generate_content_config=types.GenerateContentConfig(
        tools=[types.Tool(google_search=types.GoogleSearch())]
    ),
)

worker_agent = Agent(
    name="worker",
    model="gemini-2.5-flash",
    tools=[tool1, tool2, tool3],
)

root_agent = Agent(
    name="coordinator",
    model="gemini-2.5-flash",
    tools=[grounded_agent, worker_agent],
)
```

## References

- [MCP Documentation](https://google.github.io/adk-docs/mcp/)
- [MCP Tools (Custom Tools)](https://google.github.io/adk-docs/tools-custom/mcp-tools/)
- [A2A Introduction](https://google.github.io/adk-docs/a2a/intro/)
- [A2A Protocol Specification](https://google.github.io/A2A/)
- [Streaming Documentation](https://google.github.io/adk-docs/streaming/)
- [Artifacts Documentation](https://google.github.io/adk-docs/artifacts/)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [ADK Main Documentation](https://google.github.io/adk-docs/)
