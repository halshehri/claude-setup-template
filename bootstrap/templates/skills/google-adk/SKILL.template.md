---
name: google-adk
description: Apply Google ADK patterns when building AI agents. Use when creating agent.py files, configuring multi-agent systems, defining tools, or working with ADK project structure.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, WebFetch
---

# Google ADK Development

Apply these patterns when building agents with Google's Agent Development Kit (ADK).

## Reference index — read on demand

The `reference/` folder beside this SKILL.md contains deep documentation. Read only the file relevant to the current task:

| When you need... | Read |
|---|---|
| Project layout, env vars, first agent | `reference/fundamentals.md` |
| Agent types (LlmAgent, Sequential, Parallel, Loop) | `reference/agents.md` |
| Built-in tools and tool patterns | `reference/tools.md` |
| Building custom Python/TypeScript tools | `reference/custom-tools.md` |
| Sessions, state, memory services | `reference/memory.md` |
| Running agents, deployment, Vertex AI | `reference/runtime-deploy.md` |
| Planners, code execution, callbacks | `reference/advanced.md` |

## Project Structure

Every ADK agent project requires this minimum structure:

```
my-agent/
├── .env              # API keys (GOOGLE_API_KEY or GOOGLE_CLOUD_PROJECT)
├── __init__.py       # Must export root_agent
└── agent.py          # Agent definition with root_agent variable
```

**__init__.py** (required):
```python
from .agent import root_agent
__all__ = ["root_agent"]
```

**.env** (required):
```bash
# Google AI (Gemini) - simplest option
GOOGLE_API_KEY=your_key_here

# OR Vertex AI (production)
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_CLOUD_LOCATION=us-central1
```

## Agent Types

### LlmAgent (alias: Agent)
Dynamic, LLM-powered reasoning. Use for natural language understanding, tool selection, adaptive behavior.

```python
from google.adk.agents import Agent

root_agent = Agent(
    name="my_agent",
    model="gemini-2.5-flash",
    description="Brief purpose (used by parent agents for delegation)",
    instruction="Detailed behavior instructions for the LLM",
    tools=[my_tool_function]
)
```

### SequentialAgent
Execute sub-agents in order. Use for pipelines with defined steps.

```python
from google.adk.agents import SequentialAgent

root_agent = SequentialAgent(
    name="pipeline",
    sub_agents=[step1_agent, step2_agent, step3_agent]
)
```

### ParallelAgent
Execute sub-agents concurrently. Use for independent data gathering.

```python
from google.adk.agents import ParallelAgent

root_agent = ParallelAgent(
    name="gatherer",
    sub_agents=[weather_agent, news_agent, stocks_agent]
)
```

### LoopAgent
Repeat until condition met. Use for iterative processing.

```python
from google.adk.agents import LoopAgent

# Loop stops when any sub-agent sets tool_context.actions.escalate = True,
# or when max_iterations is reached.
root_agent = LoopAgent(
    name="batch_processor",
    sub_agents=[processing_agent],
    max_iterations=100,
)
```

## Agent Configuration

### Instructions with State Variables
```python
agent = Agent(
    name="personalized",
    model="gemini-2.5-flash",
    instruction="""You are assisting {user_name} from {user_location}.
    Preferences: {preferences}
    Use this context for personalized responses."""
)
# {var} = session.state['var'] at runtime
# {var?} = optional (no error if missing)
```

### Structured Output
```python
from pydantic import BaseModel, Field

class ExtractedInfo(BaseModel):
    name: str = Field(description="User's name")
    age: int = Field(description="User's age")

agent = Agent(
    name="extractor",
    model="gemini-2.5-flash",
    instruction="Extract user information as JSON",
    output_schema=ExtractedInfo,
    output_key="extracted_data"  # Store in session.state
)
# NOTE: Cannot use tools when output_schema is set
```

### Planning (Advanced Reasoning)
```python
from google.adk.planners import BuiltInPlanner
from google.genai import types

agent = Agent(
    name="thinker",
    model="gemini-2.5-pro",
    planner=BuiltInPlanner(
        thinking_config=types.ThinkingConfig(
            include_thoughts=True,
            thinking_budget=1024
        )
    ),
    tools=[...]
)
```

### Code Execution
```python
from google.adk.code_executors import BuiltInCodeExecutor

agent = Agent(
    name="calculator",
    model="gemini-2.5-flash",
    code_executor=BuiltInCodeExecutor(),
    instruction="Write and execute Python code to solve problems."
)
```

### Generation Config
```python
from google.genai import types

agent = Agent(
    model="gemini-2.5-flash",
    generate_content_config=types.GenerateContentConfig(
        temperature=0.2,
        max_output_tokens=1024,
        top_p=0.95
    )
)
```

### Stateless Agent (No History)
```python
agent = Agent(
    name="stateless",
    model="gemini-2.5-flash",
    include_contents='none'  # Each request independent
)
```

## Multi-Agent Patterns

### Delegation (LLM-Driven Routing)
Parent agent decides when to transfer to specialists:

```python
billing_agent = Agent(
    name="billing",
    model="gemini-2.5-flash",
    description="Handles billing questions and payment issues",
    instruction="Answer billing questions..."
)

technical_agent = Agent(
    name="technical",
    model="gemini-2.5-flash",
    description="Handles technical support and troubleshooting",
    instruction="Help with technical issues..."
)

root_agent = Agent(
    name="router",
    model="gemini-2.5-flash",
    instruction="""Route customer inquiries:
    - Billing questions → billing agent
    - Technical issues → technical agent
    - General questions → handle yourself""",
    sub_agents=[billing_agent, technical_agent]
)
```

### Hierarchical (Coordinator + Specialists)
```python
root_agent = Agent(
    name="coordinator",
    model="gemini-2.5-flash",
    instruction="""Coordinate workflow:
    1. Use researcher to gather information
    2. Use writer to create content
    3. Use reviewer to improve quality""",
    sub_agents=[research_agent, writer_agent, reviewer_agent]
)
```

### Router (Intent-Based)
```python
router = Agent(
    name="router",
    model="gemini-2.5-flash",
    instruction="""Analyze request and transfer to:
    - billing_agent for payments, invoices
    - support_agent for technical help
    - sales_agent for product info""",
    sub_agents=[billing_agent, support_agent, sales_agent]
)
```

### Pipeline (Sequential Processing)
```python
root_agent = SequentialAgent(
    name="pipeline",
    sub_agents=[input_validator, data_processor, output_formatter]
)
```

### Gather-Then-Process
```python
gather = ParallelAgent(
    name="gather",
    sub_agents=[source1_agent, source2_agent, source3_agent]
)
process = Agent(
    name="processor",
    model="gemini-2.5-flash",
    instruction="Analyze gathered data from state"
)
root_agent = SequentialAgent(
    name="workflow",
    sub_agents=[gather, process]
)
```

### Global Instructions (Apply to All)
```python
root_agent = Agent(
    name="root",
    model="gemini-2.5-flash",
    global_instruction="""RULES FOR ALL AGENTS:
    - Always be polite and professional
    - Never share personal data
    - If unsure, ask for clarification""",
    instruction="Your specific task...",
    sub_agents=[sub_agent1, sub_agent2]
)
```

## Tool Patterns

### Python Function Tools
```python
def get_weather(city: str) -> dict:
    """Get current weather for a city.

    Args:
        city: Name of the city

    Returns:
        dict with temp and condition
    """
    return {"temp": "72F", "condition": "sunny"}

agent = Agent(
    name="weather_agent",
    model="gemini-2.5-flash",
    tools=[get_weather]  # Auto-wrapped as FunctionTool
)
```

### TypeScript Tools
```typescript
import { z } from 'zod';
import { FunctionTool } from '@google/adk';

const getWeatherTool = new FunctionTool({
    name: 'getWeather',
    description: 'Get current weather for a city',
    parameters: z.object({
        city: z.string().describe('City name'),
    }),
    execute: async (params) => {
        return { temp: "72F", condition: "sunny" };
    },
});
```

### Multiple Tools with Guidance
```python
def search(query: str) -> str:
    """Search the web for information."""
    return f"Results: {query}"

def calculate(expression: str) -> float:
    """Evaluate a mathematical expression."""
    return eval(expression)

agent = Agent(
    name="assistant",
    model="gemini-2.5-flash",
    instruction="""You have tools:
    - search: Use for current info, facts, recent events
    - calculate: Use for mathematical operations
    Always use the appropriate tool for each task.""",
    tools=[search, calculate]
)
```

### Agent as Tool
```python
from google.adk.tools import AgentTool

specialist = Agent(name="specialist", model="gemini-2.5-flash", ...)
specialist_tool = AgentTool(agent=specialist)

main_agent = Agent(
    name="main",
    model="gemini-2.5-flash",
    tools=[specialist_tool]
)
```

## Running Agents

```bash
# Web UI (interactive development)
adk web .

# CLI chat (terminal)
adk run ./my_agent

# API server (programmatic access)
adk api_server --port 8000 .
```

### Programmatic Execution
```python
import asyncio
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types

session_service = InMemorySessionService()
runner = Runner(agent=root_agent, app_name="my_app", session_service=session_service)

async def interact(message: str):
    session = await session_service.create_session(
        app_name="my_app", user_id="user1", session_id="session1"
    )
    user_content = types.Content(
        role='user', parts=[types.Part(text=message)]
    )
    async for event in runner.run_async(
        user_id="user1", session_id="session1", new_message=user_content
    ):
        if event.is_final_response() and event.content:
            print(event.content.parts[0].text)

asyncio.run(interact("Hello!"))
```

## Model Selection

```python
# Fast, efficient (recommended default)
model="gemini-2.5-flash"

# More capable, slower
model="gemini-2.5-pro"

# Claude via Anthropic
model="claude-sonnet-4-20250514"

# Vertex AI hosted
model="publishers/google/models/gemini-2.0-flash-001"
```

## Best Practices

- **Root agent required**: Main agent variable must be named `root_agent`
- **Descriptive names**: Use purpose-based names (`billing_agent`, not `agent1`)
- **Clear descriptions**: Help parent agents decide when to delegate
- **Good docstrings on tools**: LLM uses docstrings to decide when to call tools
- **Return dicts from tools**: Always return structured data, not raw exceptions
- **Handle errors in tools**: Return `{"error": "message"}` instead of raising
- **Flat hierarchies**: Keep agent trees 2-3 levels deep max
- **Single responsibility**: Each agent has one clear, focused purpose
- **State for data passing**: Use `output_key` and session state between agents
- **InMemorySessionService for dev**: Use persistent storage for production

## Anti-Patterns

- Forgetting to name the variable `root_agent`
- Missing `__init__.py` or not exporting `root_agent`
- Tools without docstrings (LLM can't decide when to use them)
- Using `output_schema` and `tools` together (not supported)
- Deep agent hierarchies (5+ levels) - causes confusion and latency
- Vague agent descriptions like "helps with stuff"
- Swallowing tool errors instead of returning structured error info
- Not setting `GOOGLE_API_KEY` or Vertex AI credentials in `.env`

## Environment Variables

```bash
# Google AI (simplest)
GOOGLE_API_KEY=...

# Vertex AI (production)
GOOGLE_CLOUD_PROJECT=...
GOOGLE_CLOUD_LOCATION=...

# Other LLM providers
ANTHROPIC_API_KEY=...
OPENAI_API_KEY=...

# Development
ADK_DEBUG=true
ADK_LOG_LEVEL=INFO
```

## References

- [ADK Documentation](https://google.github.io/adk-docs)
- [Python Quickstart](https://google.github.io/adk-docs/get-started/python/)
- [Agent Types](https://google.github.io/adk-docs/agents/)
- [Multi-Agent Systems](https://google.github.io/adk-docs/agents/multi-agents/)
- Deep reference: `reference/` folder beside this SKILL.md
