# ADK Memory & Sessions

Use this reference for managing sessions, state, and memory in Google's Agent Development Kit. This covers session lifecycle, state scopes, long-term memory, and context management.

## When to Use This Reference

- Managing conversation sessions (create, read, update, delete)
- Persisting state within or across sessions
- Sharing data between agents in multi-agent systems
- Configuring long-term memory across sessions
- Optimizing context window usage (caching, compression)
- Choosing between InMemory vs persistent storage backends
- Using state variables in agent instructions

## Core Concepts

### Session vs State vs Memory

| Concept | Scope | Lifetime | Purpose |
|---------|-------|----------|---------|
| **Session** | Single conversation | Until deleted or expired | Tracks messages, events, and context for one conversation thread |
| **State** | Varies (agent/session/user/app) | Depends on scope | Key-value working memory; passes data between agents and turns |
| **Memory** | Cross-session | Long-term | Recalls information from past sessions for future conversations |

**When to use each:**
- **Session**: Always needed. Every agent interaction happens within a session.
- **State**: When agents need to store/retrieve data during execution, pass data between agents, or persist user preferences.
- **Memory**: When an agent should recall information from previous conversations (e.g., "last time you asked about...").

### How They Relate

```
User
  |
  v
Session (one conversation)
  ├── Events (messages, tool calls, responses)
  ├── State (key-value data, scoped by prefix)
  │     ├── agent:* (current agent only)
  │     ├── (no prefix) session-level
  │     ├── user:* (across sessions for same user)
  │     └── app:* (across all users and sessions)
  └── Memory (loaded from past sessions at start)
```

## Session Management

### Session Object Structure

```python
from google.adk.sessions import Session

# A session contains:
session.id           # Unique session identifier
session.app_name     # Application name
session.user_id      # User identifier
session.state        # Dict[str, Any] - state key-value pairs
session.events       # List[Event] - conversation history
session.last_update_time  # Timestamp of last modification
```

### InMemorySessionService

**Use for**: Development, testing, prototyping. Data lost when process exits.

```python
from google.adk.sessions import InMemorySessionService

session_service = InMemorySessionService()

# Create a session
session = await session_service.create_session(
    app_name="my_app",
    user_id="user_123",
    session_id="session_456",       # Optional: auto-generated if omitted
    state={"language": "en"}        # Optional: initial state
)

# Get an existing session
session = await session_service.get_session(
    app_name="my_app",
    user_id="user_123",
    session_id="session_456"
)

# List all sessions for a user
sessions = await session_service.list_sessions(
    app_name="my_app",
    user_id="user_123"
)

# Delete a session
await session_service.delete_session(
    app_name="my_app",
    user_id="user_123",
    session_id="session_456"
)
```

### Persistent Session Services

#### DatabaseSessionService (SQL databases)

**Use for**: Production with PostgreSQL, MySQL, SQLite, or any SQLAlchemy-supported database.

```python
from google.adk.sessions import DatabaseSessionService

# SQLite (local file)
session_service = DatabaseSessionService(
    db_url="sqlite:///./sessions.db"
)

# PostgreSQL
session_service = DatabaseSessionService(
    db_url="postgresql://user:password@localhost:5432/mydb"
)

# PostgreSQL with async driver
session_service = DatabaseSessionService(
    db_url="postgresql+asyncpg://user:password@localhost:5432/mydb"
)
```

**Install database drivers:**
```bash
# PostgreSQL
pip install google-adk[postgresql]
# or
pip install asyncpg sqlalchemy[asyncio]

# SQLite (included by default)
pip install aiosqlite
```

#### VertexAiSessionService (Managed Cloud)

**Use for**: Production on Google Cloud with fully managed infrastructure.

```python
from google.adk.sessions import VertexAiSessionService

session_service = VertexAiSessionService(
    project="my-gcp-project",
    location="us-central1"
)
```

**Requirements:**
- Google Cloud project with Vertex AI API enabled
- Appropriate IAM permissions
- `google-adk[vertexai]` installed

### Session Lifecycle

#### Full Lifecycle Example

```python
import asyncio
from google.adk.agents import Agent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types

APP_NAME = "my_app"
USER_ID = "user_123"

# 1. Setup
agent = Agent(
    name="assistant",
    model="gemini-2.5-flash",
    instruction="You are a helpful assistant. User's name: {user_name}"
)

session_service = InMemorySessionService()
runner = Runner(
    agent=agent,
    app_name=APP_NAME,
    session_service=session_service
)

async def main():
    # 2. Create session with initial state
    session = await session_service.create_session(
        app_name=APP_NAME,
        user_id=USER_ID,
        state={"user_name": "Alice"}
    )
    session_id = session.id

    # 3. Send messages within the session
    user_msg = types.Content(
        role="user",
        parts=[types.Part(text="Hello!")]
    )

    async for event in runner.run_async(
        user_id=USER_ID,
        session_id=session_id,
        new_message=user_msg
    ):
        if event.is_final_response() and event.content:
            print(event.content.parts[0].text)

    # 4. Retrieve session (includes all events and updated state)
    session = await session_service.get_session(
        app_name=APP_NAME,
        user_id=USER_ID,
        session_id=session_id
    )
    print(f"Events: {len(session.events)}")
    print(f"State: {session.state}")

    # 5. List user's sessions
    all_sessions = await session_service.list_sessions(
        app_name=APP_NAME,
        user_id=USER_ID
    )
    print(f"Total sessions: {len(all_sessions)}")

    # 6. Delete when done
    await session_service.delete_session(
        app_name=APP_NAME,
        user_id=USER_ID,
        session_id=session_id
    )

asyncio.run(main())
```

#### Session IDs and User IDs

```python
# Auto-generated session ID
session = await session_service.create_session(
    app_name="my_app",
    user_id="user_123"
    # session_id is auto-generated (UUID)
)
print(session.id)  # e.g., "a1b2c3d4-..."

# Explicit session ID (useful for resuming)
session = await session_service.create_session(
    app_name="my_app",
    user_id="user_123",
    session_id="order-flow-789"
)

# User ID patterns
# - Use authentication ID for real users
# - Use "test_user" for development
# - Sessions are scoped per (app_name, user_id)
```

## State Management

### State Scopes

State keys use prefixes to control their visibility and persistence scope:

| Prefix | Scope | Visible To | Persists Across |
|--------|-------|-----------|-----------------|
| (none) | Session | All agents in session | Turns in same session |
| `agent:` | Agent | Only the specific agent that wrote it | Turns (same agent, same session) |
| `user:` | User | All sessions for the same user | Sessions (same user, same app) |
| `app:` | App | All users and sessions | Everything (global to the app) |
| `temp:` | Temporary | Current invocation only | Nothing (cleared after each run) |

```python
# Session state (default) - shared across all agents in this session
state["conversation_topic"] = "weather"
state["item_count"] = 5

# Agent-scoped state - only this agent can see it
state["agent:internal_counter"] = 10
state["agent:draft_response"] = "..."

# User-scoped state - persists across sessions for this user
state["user:preferred_language"] = "en"
state["user:name"] = "Alice"

# App-scoped state - global across all users
state["app:total_queries"] = 1000
state["app:system_status"] = "operational"

# Temporary state - gone after this invocation
state["temp:intermediate_result"] = "..."
```

### Reading and Writing State

#### In Tools (via tool_context)

```python
from google.adk.tools import ToolContext

def save_preference(preference: str, value: str, tool_context: ToolContext) -> dict:
    """Save a user preference.

    Args:
        preference: The preference name.
        value: The preference value.
        tool_context: Provided automatically by ADK.
    """
    # Write to user-scoped state (persists across sessions)
    tool_context.state["user:" + preference] = value

    # Read from state
    all_prefs = {
        k.replace("user:", ""): v
        for k, v in tool_context.state.items()
        if k.startswith("user:")
    }

    return {"status": "saved", "all_preferences": all_prefs}


def get_order_status(order_id: str, tool_context: ToolContext) -> dict:
    """Check order status.

    Args:
        order_id: The order ID to check.
        tool_context: Provided automatically by ADK.
    """
    # Read session state
    user_name = tool_context.state.get("user_name", "Unknown")

    # Write session state
    tool_context.state["last_order_checked"] = order_id

    return {"order_id": order_id, "status": "shipped", "customer": user_name}


agent = Agent(
    name="assistant",
    model="gemini-2.5-flash",
    tools=[save_preference, get_order_status]
)
```

**Important:** The `tool_context` parameter is automatically injected by ADK. Do NOT include it in the tool's docstring `Args` section if you want the LLM to ignore it. ADK recognizes the parameter name and type and injects it automatically.

#### In Instructions (Template Variables)

State values are accessible in instructions using `{key}` syntax:

```python
agent = Agent(
    name="personalized_agent",
    model="gemini-2.5-flash",
    instruction="""You are a personal assistant.

User's name: {user_name}
User's language: {user:preferred_language}
Today's special: {app:daily_special}

Greet the user by name and communicate in their preferred language.
Items in cart: {cart_items}
"""
)
# At runtime, {user_name} is replaced with state["user_name"]
# {user:preferred_language} is replaced with state["user:preferred_language"]
```

**Optional variables** (no error if missing):
```python
instruction = """
User's name: {user_name}
Loyalty tier: {user:loyalty_tier?}
"""
# {user:loyalty_tier?} - the ? makes it optional; empty string if not set
```

#### Between Agents (Multi-Agent State Sharing)

```python
from google.adk.agents import SequentialAgent, Agent

# Agent 1: Extract data and save to state
extractor = Agent(
    name="extractor",
    model="gemini-2.5-flash",
    instruction="Extract the user's name and age from the message.",
    output_key="extracted_data"  # Saves output to state["extracted_data"]
)

# Agent 2: Use extracted data from state
formatter = Agent(
    name="formatter",
    model="gemini-2.5-flash",
    instruction="""Format a greeting using the extracted data.
Extracted info: {extracted_data}
"""
)

# Pipeline: extractor -> formatter
root_agent = SequentialAgent(
    name="pipeline",
    agents=[extractor, formatter]
)
```

#### The output_key Pattern

`output_key` stores the agent's final text response into state:

```python
# Simple output_key
summarizer = Agent(
    name="summarizer",
    model="gemini-2.5-flash",
    instruction="Summarize the given text concisely.",
    output_key="summary"
)
# After execution: state["summary"] = "The agent's text response..."

# With output_schema (structured output stored as JSON string)
from pydantic import BaseModel

class Analysis(BaseModel):
    sentiment: str
    confidence: float
    keywords: list[str]

analyzer = Agent(
    name="analyzer",
    model="gemini-2.5-flash",
    instruction="Analyze the sentiment of the text. Respond as JSON.",
    output_schema=Analysis,
    output_key="analysis_result"
)
# After execution: state["analysis_result"] = '{"sentiment": "positive", ...}'
```

**Scoped output_key:**
```python
# Store in agent scope (only this agent can read it)
agent = Agent(
    name="internal",
    output_key="agent:my_result"
)

# Store in user scope (persists across sessions)
agent = Agent(
    name="profiler",
    output_key="user:profile_summary"
)
```

### State Persistence

**When state is saved:**
- State changes made in tools (via `tool_context.state`) are persisted after the tool completes
- State changes from `output_key` are persisted after the agent produces its response
- With `InMemorySessionService`, state exists only in memory
- With `DatabaseSessionService` or `VertexAiSessionService`, state is durably persisted

**State serialization:**
- State values must be JSON-serializable (strings, numbers, booleans, lists, dicts)
- Complex objects should be serialized to JSON strings or dicts
- Avoid storing large binary data in state; use Artifacts instead

```python
# Good - JSON-serializable values
state["count"] = 42
state["tags"] = ["important", "urgent"]
state["config"] = {"theme": "dark", "lang": "en"}

# Bad - not serializable
state["model"] = some_ml_model  # Will fail
state["connection"] = db_conn   # Will fail

# For complex data, serialize explicitly
import json
state["complex_data"] = json.dumps(my_object.__dict__)
```

## Memory (Long-Term)

### What Memory Provides

Memory enables agents to recall information from **previous sessions**. Without memory, each new session starts fresh with no knowledge of past interactions.

**Use cases:**
- "Remember that I prefer dark mode" (recalled in future sessions)
- "Last time we discussed the Q4 report" (cross-session context)
- Building user profiles over time
- Continuity across conversation threads

**How it works:**
1. After a session ends, the memory service processes and stores session content
2. When a new session starts, relevant memories are retrieved and injected into context
3. The agent sees these memories as additional context alongside the conversation

### Memory Service Types

#### InMemoryMemoryService

**Use for**: Development and testing. Stores memories in process memory.

```python
from google.adk.memory import InMemoryMemoryService

memory_service = InMemoryMemoryService()
```

#### VertexAiRagMemoryService

**Use for**: Production on Google Cloud. Uses Vertex AI RAG Engine for semantic search over memories.

```python
from google.adk.memory import VertexAiRagMemoryService

memory_service = VertexAiRagMemoryService(
    rag_corpus="projects/my-project/locations/us-central1/ragCorpora/my-corpus"
)
```

**Setup requirements:**
1. Create a RAG corpus in Vertex AI
2. Enable the Vertex AI API
3. Set appropriate IAM permissions

### Memory Configuration

#### Integrating Memory with Runner

```python
from google.adk.agents import Agent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.memory import InMemoryMemoryService

agent = Agent(
    name="assistant",
    model="gemini-2.5-flash",
    instruction="You are a helpful assistant. Use any memories of past conversations to personalize responses."
)

session_service = InMemorySessionService()
memory_service = InMemoryMemoryService()

runner = Runner(
    agent=agent,
    app_name="my_app",
    session_service=session_service,
    memory_service=memory_service  # Enable long-term memory
)
```

#### Storing Memories (After Session Ends)

```python
# Memories are created from completed sessions
# Call add_session_to_memory to process a session into memory

await memory_service.add_session_to_memory(session)
```

The memory service processes the session's events and extracts key information that can be retrieved later.

#### Retrieving Memories (Automatic)

When a `memory_service` is configured on the Runner, memories are automatically retrieved at the start of each new session based on the user's query. The Runner:

1. Takes the user's first message
2. Searches memory for relevant past interactions
3. Injects matching memories into the agent's context
4. The agent can reference this information naturally

### Memory Patterns

#### Pattern: User Preference Learning

```python
def save_user_preference(category: str, preference: str, tool_context: ToolContext) -> dict:
    """Save a user preference for future sessions.

    Args:
        category: Category of preference (e.g., 'food', 'style').
        preference: The specific preference value.
    """
    # Save to user-scoped state for immediate use
    tool_context.state[f"user:pref_{category}"] = preference
    return {"status": "saved", "category": category, "preference": preference}

agent = Agent(
    name="assistant",
    model="gemini-2.5-flash",
    instruction="""You are a personal assistant.
    When a user tells you their preferences, save them using the save_user_preference tool.
    Reference past preferences from memory when making suggestions.""",
    tools=[save_user_preference]
)
```

#### Pattern: Conversation Continuity

```python
agent = Agent(
    name="project_assistant",
    model="gemini-2.5-flash",
    instruction="""You are a project management assistant.

    If memory from past sessions is available, reference it to maintain continuity:
    - Recall previous project updates
    - Remember action items from past conversations
    - Track progress over time

    Always acknowledge what you remember from previous interactions.
    """
)

# With memory_service configured, the agent can say:
# "Last time we spoke, you mentioned the API redesign was at 60%.
#  How is that progressing?"
```

## Context Management

### include_contents Option

Controls how much conversation history the agent receives:

```python
# Default: Include full conversation history
agent = Agent(
    name="contextual",
    model="gemini-2.5-flash",
    include_contents='default'  # All prior messages included
)

# Stateless: No conversation history
agent = Agent(
    name="stateless",
    model="gemini-2.5-flash",
    include_contents='none'  # Each turn is independent
)
```

**When to use `'none'`:**
- Stateless processing (e.g., classification, extraction)
- Cost optimization (fewer input tokens)
- Privacy (no prior context leakage)
- Sub-agents that process single inputs in a pipeline

### Context and Token Management

#### Managing Long Conversations

Long conversations accumulate tokens. Strategies to manage this:

```python
# 1. Stateless sub-agents for heavy processing
processor = Agent(
    name="processor",
    model="gemini-2.5-flash",
    include_contents='none',  # Don't load history
    instruction="Process only the current input."
)

# 2. Use output_key to summarize and carry forward
summarizer = Agent(
    name="summarizer",
    model="gemini-2.5-flash",
    instruction="Summarize the conversation so far in 2-3 sentences.",
    output_key="conversation_summary"
)

# 3. Reference summary instead of full history
main_agent = Agent(
    name="main",
    model="gemini-2.5-flash",
    instruction="""You are a helpful assistant.
Previous conversation summary: {conversation_summary}
Use this context to maintain continuity."""
)
```

#### Token-Efficient Patterns

```python
# Pattern: Periodic summarization in a loop
from google.adk.agents import SequentialAgent, LoopAgent

# Summarize every N turns to keep context manageable
summarize_step = Agent(
    name="summarizer",
    model="gemini-2.5-flash",
    include_contents='none',
    instruction="Summarize the key points from: {recent_messages}",
    output_key="running_summary"
)

work_step = Agent(
    name="worker",
    model="gemini-2.5-flash",
    instruction="""Context so far: {running_summary}
Handle the user's current request.""",
    output_key="recent_messages"
)

pipeline = SequentialAgent(
    name="efficient_pipeline",
    agents=[summarize_step, work_step]
)
```

#### Artifacts for Large Data

For binary or large data, use Artifacts instead of state:

```python
from google.adk.tools import ToolContext
from google.genai import types

def save_document(content: str, filename: str, tool_context: ToolContext) -> dict:
    """Save a document as an artifact.

    Args:
        content: The document content.
        filename: Name for the file.
    """
    artifact = types.Part.from_text(text=content)
    version = tool_context.save_artifact(filename=filename, artifact=artifact)
    return {"status": "saved", "filename": filename, "version": version}

def load_document(filename: str, tool_context: ToolContext) -> dict:
    """Load a document artifact.

    Args:
        filename: Name of the file to load.
    """
    artifact = tool_context.load_artifact(filename=filename)
    if artifact:
        return {"content": artifact.text}
    return {"error": "Document not found"}
```

## Best Practices

### Session Management
- Use `InMemorySessionService` only for development; always use a persistent service in production
- Generate meaningful session IDs that aid debugging (e.g., include user context or flow type)
- Clean up old sessions to avoid unbounded storage growth
- Scope sessions per user with unique `user_id` values

### State Design
- Use the narrowest scope possible: `agent:` for internal data, session for shared data
- Use `user:` scope for preferences that should persist across conversations
- Use `app:` scope sparingly, only for truly global data
- Keep state values small and JSON-serializable
- Use `output_key` to pass results between agents in pipelines
- Prefer structured state keys: `user:pref_language`, `agent:step_count`

### Memory
- Configure memory services early in project lifecycle
- Write clear, informative agent responses that create useful memories
- Use `user:` state for structured preferences; use memory for unstructured recall
- Test memory retrieval relevance during development
- Consider privacy implications of stored memories

### Context Optimization
- Set `include_contents='none'` for stateless processing agents
- Use `output_key` + template variables instead of passing full conversation history
- Summarize periodically in long-running conversations
- Store large data as artifacts, not in state

## Common Pitfalls

### Using InMemorySessionService in production
```python
# Wrong - data lost on restart
session_service = InMemorySessionService()  # Fine for dev, not prod

# Correct - use persistent storage
session_service = DatabaseSessionService(
    db_url="postgresql://user:pass@host/db"
)
```

### Forgetting state scope prefixes
```python
# Wrong - saves to session scope, lost when session ends
tool_context.state["preferred_language"] = "en"

# Correct - saves to user scope, persists across sessions
tool_context.state["user:preferred_language"] = "en"
```

### Non-serializable state values
```python
# Wrong - will fail on persistent backends
tool_context.state["data"] = datetime.now()      # Not JSON-serializable
tool_context.state["data"] = custom_object        # Not JSON-serializable

# Correct - use serializable types
tool_context.state["data"] = datetime.now().isoformat()  # String
tool_context.state["data"] = {"key": "value"}            # Dict
```

### Overloading state with large values
```python
# Wrong - large data in state degrades performance
tool_context.state["full_document"] = very_long_string  # Sent every turn

# Correct - use artifacts for large data
tool_context.save_artifact(filename="document.txt", artifact=part)
```

### Missing output_key for agent pipelines
```python
# Wrong - agent 2 can't access agent 1's output
agent1 = Agent(name="step1", instruction="Analyze the data")
agent2 = Agent(name="step2", instruction="Use analysis: {analysis}")
# {analysis} is never set!

# Correct - output_key bridges the agents
agent1 = Agent(name="step1", instruction="Analyze the data", output_key="analysis")
agent2 = Agent(name="step2", instruction="Use analysis: {analysis}")
```

### Forgetting memory_service in Runner
```python
# Wrong - memory is never loaded or stored
runner = Runner(
    agent=agent,
    app_name="my_app",
    session_service=session_service
    # memory_service missing!
)

# Correct
runner = Runner(
    agent=agent,
    app_name="my_app",
    session_service=session_service,
    memory_service=memory_service
)
```

### Reading agent-scoped state from another agent
```python
# Agent A writes
tool_context.state["agent:secret"] = "data"

# Agent B tries to read - will NOT see it
value = tool_context.state.get("agent:secret")  # None

# Fix: Use session-scoped state for shared data
tool_context.state["shared_data"] = "data"  # All agents can read
```

## References

- [Sessions Overview](https://google.github.io/adk-docs/sessions/)
- [Session Management](https://google.github.io/adk-docs/sessions/session/)
- [State Management](https://google.github.io/adk-docs/sessions/state/)
- [Memory](https://google.github.io/adk-docs/sessions/memory/)
- [Context](https://google.github.io/adk-docs/context/)
- [Artifacts](https://google.github.io/adk-docs/artifacts/)
- [ADK Documentation](https://google.github.io/adk-docs)
