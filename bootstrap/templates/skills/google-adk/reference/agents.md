# ADK Agents

Use this skill for detailed agent configuration, understanding agent types, and building multi-agent systems with Google's Agent Development Kit.

## When to Use This Skill

- Configuring LlmAgent with advanced options
- Choosing between agent types (LLM vs Workflow vs Custom)
- Building multi-agent systems
- Using planning, structured I/O, code execution
- Agent delegation and transfer patterns

## Agent Types Overview

### LlmAgent (Agent)
**Purpose**: Dynamic, LLM-powered reasoning and decision-making
**When to use**:
- Natural language understanding required
- Dynamic tool selection needed
- Adaptive behavior based on context
- Non-deterministic workflows

**Non-deterministic**: Uses LLM to decide next steps

### Workflow Agents
**Purpose**: Deterministic, predictable execution patterns
**When to use**:
- Structured processes with defined steps
- Guaranteed execution order needed
- No LLM reasoning required for flow control

**Deterministic**: Fixed execution patterns

#### Types:
- **SequentialAgent**: Execute sub-agents one after another
- **ParallelAgent**: Execute sub-agents concurrently
- **LoopAgent**: Repeat until termination condition

### Custom Agents
**Purpose**: Specialized logic not covered by standard types
**When to use**:
- Unique control flows
- Custom integrations
- Specialized protocols

## LlmAgent Configuration

### Identity and Purpose

```python
from google.adk.agents import LlmAgent, Agent

agent = LlmAgent(
    # REQUIRED
    name="customer_support",  # Unique identifier
    model="gemini-2.5-flash",  # LLM to use

    # RECOMMENDED (especially for multi-agent)
    description="Handles customer support inquiries about billing",

    # Core behavior definition
    instruction="You are a helpful customer support agent..."
)

# Alias: Agent = LlmAgent
agent = Agent(name="...", model="...", ...)
```

### Instruction Patterns

#### Basic Instruction
```python
instruction = """You are a weather assistant.
When user asks about weather:
1. Extract city name
2. Use get_weather tool
3. Format response clearly
"""
```

#### Instruction with Examples (Few-Shot)
```python
instruction = """You are a data extractor.

Example Input: "John is 25 years old"
Example Output: {"name": "John", "age": 25}

Example Input: "Sarah lives in NYC"
Example Output: {"name": "Sarah", "location": "NYC"}

Extract information from user messages.
"""
```

#### Instruction with State Variables
```python
instruction = """You are a personalized assistant.
User's name: {user_name}
User's location: {user_location}
Preferences: {preferences}

Use this context to provide personalized responses.
"""
# State variables replaced at runtime
# {var} = session.state['var']
# {artifact.name} = artifact content
# {var?} = optional (no error if missing)
```

#### Instruction with Tool Guidance
```python
instruction = """You are a research assistant with three tools:

1. web_search(query): Use for current events, facts, recent information
   - When: User asks "what", "when", "who" questions
   - Example: "What happened yesterday?"

2. calculator(expression): Use for mathematical calculations
   - When: User provides numbers and operations
   - Example: "What is 15% of 250?"

3. database_query(sql): Use for internal data lookup
   - When: User asks about company records
   - Example: "Show sales for Q4"

Always use the most appropriate tool for the task.
"""
```

### Tool Configuration

#### Simple Function Tools (Python)
```python
def get_weather(city: str) -> dict:
    """Retrieves current weather for a city.

    Args:
        city: Name of the city

    Returns:
        dict with 'temp' and 'condition' keys
    """
    # Implementation
    return {"temp": "72F", "condition": "sunny"}

agent = Agent(
    name="weather_agent",
    model="gemini-2.5-flash",
    tools=[get_weather]  # Auto-wrapped as FunctionTool
)
```

#### Complex Function Tools (TypeScript)
```typescript
import {z} from 'zod';
import { FunctionTool } from '@google/adk';

const weatherParamsSchema = z.object({
    city: z.string().describe('City name'),
});

const getWeatherTool = new FunctionTool({
    name: 'getWeather',
    description: 'Get current weather for a city',
    parameters: weatherParamsSchema,
    execute: async (params) => {
        return {temp: "72F", condition: "sunny"};
    },
});

const agent = new LlmAgent({
    tools: [getWeatherTool],
});
```

#### Multiple Tools
```python
def search(query: str) -> str:
    """Search the web."""
    return f"Results: {query}"

def calculate(expression: str) -> float:
    """Evaluate math expression."""
    return eval(expression)

def get_time(timezone: str) -> str:
    """Get current time in timezone."""
    return "12:30 PM EST"

multi_tool_agent = Agent(
    name="assistant",
    model="gemini-2.5-flash",
    instruction="Use appropriate tool for each task",
    tools=[search, calculate, get_time]
)
```

## Advanced LlmAgent Features

### Structured Input/Output

#### Input Schema (Enforce Input Format)
```python
from pydantic import BaseModel, Field

class QueryInput(BaseModel):
    city: str = Field(description="City name")
    days: int = Field(description="Number of days", default=7)

agent = Agent(
    name="forecast_agent",
    model="gemini-2.5-flash",
    input_schema=QueryInput,  # User must send JSON
    instruction="""You receive JSON: {"city": "...", "days": ...}
    Provide forecast for the specified city and days."""
)
```

#### Output Schema (Enforce Output Format)
```python
class ForecastOutput(BaseModel):
    city: str
    forecast: list[str] = Field(description="Daily forecasts")
    summary: str

agent = Agent(
    name="forecast_agent",
    model="gemini-2.5-flash",
    output_schema=ForecastOutput,  # Agent must respond in JSON
    instruction="Respond ONLY with JSON matching the schema"
)
# NOTE: Cannot use tools when output_schema is set
```

#### Output Key (Store Result in State)
```python
agent = Agent(
    name="extractor",
    model="gemini-2.5-flash",
    output_key="extracted_data",  # Store final response here
    instruction="Extract data and respond"
)
# Result stored in: session.state['extracted_data']
```

### Generation Control

```python
from google.genai import types

agent = Agent(
    name="agent",
    model="gemini-2.5-flash",
    generate_content_config=types.GenerateContentConfig(
        temperature=0.2,  # Lower = more deterministic
        max_output_tokens=500,  # Limit response length
        top_p=0.95,
        top_k=40,
        safety_settings=[
            types.SafetySetting(
                category=types.HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT,
                threshold=types.HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE,
            )
        ]
    )
)
```

### Context Control

```python
# Include conversation history (default)
agent = Agent(
    name="contextual_agent",
    include_contents='default'
)

# Stateless - no conversation history
agent = Agent(
    name="stateless_agent",
    include_contents='none'  # Each request independent
)
```

### Planning (Advanced Reasoning)

#### Built-in Planner (Gemini Thinking)
```python
from google.adk.planners import BuiltInPlanner
from google.genai import types

agent = Agent(
    name="thinking_agent",
    model="gemini-2.5-pro-preview-03-25",
    planner=BuiltInPlanner(
        thinking_config=types.ThinkingConfig(
            include_thoughts=True,  # Include reasoning in response
            thinking_budget=1024,  # Tokens for thinking
        )
    ),
    tools=[...]
)
```

#### Plan-ReAct Planner
```python
from google.adk.planners import PlanReActPlanner

agent = Agent(
    name="planner_agent",
    model="gemini-2.5-flash",
    planner=PlanReActPlanner(),  # Structured planning output
    tools=[...]
)
# Response format:
# /*PLANNING*/ - Agent creates plan
# /*ACTION*/ - Agent executes
# /*REASONING*/ - Agent explains
# /*FINAL_ANSWER*/ - Final response
```

### Code Execution

```python
from google.adk.code_executors import BuiltInCodeExecutor

code_agent = Agent(
    name="calculator",
    model="gemini-2.5-flash",
    code_executor=BuiltInCodeExecutor(),
    instruction="""You are a calculator.
    Write and execute Python code to solve problems.
    Return only the numerical result."""
)
# Agent can generate and execute code blocks
```

## Workflow Agents

### Sequential Agent

```python
from google.adk.agents import SequentialAgent, Agent

# Define sub-agents
greeting_agent = Agent(
    name="greeter",
    model="gemini-2.5-flash",
    instruction="Greet the user warmly"
)

main_agent = Agent(
    name="main",
    model="gemini-2.5-flash",
    instruction="Handle the user's request",
    tools=[...]
)

farewell_agent = Agent(
    name="farewell",
    model="gemini-2.5-flash",
    instruction="Say goodbye politely"
)

# Sequential workflow
root_agent = SequentialAgent(
    name="workflow",
    agents=[greeting_agent, main_agent, farewell_agent]
)
# Executes: greeting -> main -> farewell (in order)
```

### Parallel Agent

```python
from google.adk.agents import ParallelAgent, Agent

weather_agent = Agent(
    name="weather",
    model="gemini-2.5-flash",
    instruction="Get weather"
)

news_agent = Agent(
    name="news",
    model="gemini-2.5-flash",
    instruction="Get news"
)

stocks_agent = Agent(
    name="stocks",
    model="gemini-2.5-flash",
    instruction="Get stock prices"
)

root_agent = ParallelAgent(
    name="dashboard",
    agents=[weather_agent, news_agent, stocks_agent]
)
# Executes: All three agents concurrently
```

### Loop Agent

```python
from google.adk.agents import LoopAgent, Agent

processing_agent = Agent(
    name="processor",
    model="gemini-2.5-flash",
    instruction="Process one item and store result in state['processed']"
)

def should_continue(state: dict) -> bool:
    """Loop until all items processed."""
    return len(state.get('items', [])) > 0

root_agent = LoopAgent(
    name="batch_processor",
    agent=processing_agent,
    max_iterations=100,
    termination_condition=should_continue
)
```

## Multi-Agent Systems

### Pattern: Agent Delegation (LLM-Driven Transfer)

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

# Main agent can transfer to specialists
root_agent = Agent(
    name="router",
    model="gemini-2.5-flash",
    description="Routes customer inquiries to appropriate specialist",
    instruction="""You are a support router.
    - For billing questions, transfer to billing agent
    - For technical issues, transfer to technical agent
    - For general questions, handle yourself
    """,
    tools=[billing_agent, technical_agent]  # Sub-agents as tools
)
# LLM decides when to transfer based on instruction
```

### Pattern: Explicit Agent Tool Invocation

```python
from google.adk.tools import AgentTool

specialist_agent = Agent(
    name="specialist",
    model="gemini-2.5-flash",
    instruction="Provide specialized analysis"
)

# Wrap agent as explicit tool
specialist_tool = AgentTool(agent=specialist_agent)

main_agent = Agent(
    name="main",
    model="gemini-2.5-flash",
    instruction="Use specialist_agent tool for complex analysis",
    tools=[specialist_tool]
)
```

### Pattern: Hierarchical Multi-Agent

```python
# Leaf agents (specialists)
research_agent = Agent(
    name="researcher",
    model="gemini-2.5-flash",
    description="Searches and gathers information",
    tools=[search_tool]
)

writer_agent = Agent(
    name="writer",
    model="gemini-2.5-flash",
    description="Creates written content",
    tools=[document_tool]
)

reviewer_agent = Agent(
    name="reviewer",
    model="gemini-2.5-flash",
    description="Reviews and improves content"
)

# Coordinator agent
root_agent = Agent(
    name="coordinator",
    model="gemini-2.5-flash",
    description="Coordinates research, writing, and review workflow",
    instruction="""You coordinate a content creation workflow:
    1. Use researcher to gather information
    2. Use writer to create content
    3. Use reviewer to improve quality
    """,
    tools=[research_agent, writer_agent, reviewer_agent]
)
```

### Global Instructions (Apply to All Agents)

```python
root_agent = Agent(
    name="root",
    model="gemini-2.5-flash",
    global_instruction="""IMPORTANT RULES FOR ALL AGENTS:
    - Always be polite and professional
    - Never share personal data
    - If unsure, ask for clarification
    - Log all tool usage
    """,
    instruction="Your specific task...",
    tools=[sub_agent1, sub_agent2]
)
# global_instruction applied to entire agent hierarchy
```

### Transfer Control

```python
agent = Agent(
    name="specialized",
    model="gemini-2.5-flash",
    disallow_transfer_to_parent=True,  # Can't transfer up
    disallow_transfer_to_peers=True,   # Can't transfer sideways
    # Agent must handle all requests or fail
)
```

## Common Patterns

### Pattern: Router Agent
```python
"""Route to specialized sub-agents based on intent"""
router = Agent(
    name="router",
    model="gemini-2.5-flash",
    description="Routes requests to appropriate handler",
    instruction="""Analyze request and transfer to:
    - billing_agent for payments, invoices
    - support_agent for technical help
    - sales_agent for product info
    """,
    tools=[billing_agent, support_agent, sales_agent]
)
```

### Pattern: Pipeline Agent
```python
"""Sequential processing pipeline"""
root_agent = SequentialAgent(
    name="pipeline",
    agents=[
        input_validator,
        data_processor,
        output_formatter
    ]
)
```

### Pattern: Gather-Then-Process
```python
"""Collect data in parallel, then process"""
gather_agent = ParallelAgent(
    name="gather",
    agents=[source1_agent, source2_agent, source3_agent]
)

process_agent = Agent(
    name="processor",
    model="gemini-2.5-flash",
    instruction="Analyze gathered data from state"
)

root_agent = SequentialAgent(
    name="workflow",
    agents=[gather_agent, process_agent]
)
```

### Pattern: Try-Fallback
```python
"""Try primary agent, fallback to secondary"""
def fallback_logic(state):
    if 'primary_result' in state and state['primary_result']:
        return False  # Success, don't continue
    return True  # Failed, try fallback

root_agent = LoopAgent(
    name="try_fallback",
    agent=SequentialAgent(
        name="attempts",
        agents=[primary_agent, fallback_agent]
    ),
    max_iterations=1,
    termination_condition=fallback_logic
)
```

## Best Practices

### Agent Design
- **Single responsibility**: Each agent has clear, focused purpose
- **Descriptive names**: Reflect agent's role (billing_agent, not agent1)
- **Clear descriptions**: Help parent agents decide when to delegate
- **Specific instructions**: Detailed guidance produces better results

### Multi-Agent Architecture
- **Hierarchical design**: Clear parent-child relationships
- **Minimize depth**: Prefer flat hierarchies (2-3 levels max)
- **State management**: Use state to pass data between agents
- **Avoid circular references**: Agent A -> B -> A creates loops

### Tool Usage
- **Tool selection**: Give agents only tools they need
- **Tool descriptions**: Clear docstrings help LLM choose correctly
- **Error handling**: Tools should return structured errors, not raise
- **Tool chaining**: Agent can use multiple tools in sequence

### Instructions
- **Be explicit**: LLM needs clear guidance for complex tasks
- **Provide examples**: Few-shot learning improves accuracy
- **Update iteratively**: Refine instructions based on behavior
- **Use markdown**: Headers and lists improve readability

## Common Pitfalls

### Ambiguous agent descriptions
```python
# Bad - too vague
agent = Agent(
    name="helper",
    description="Helps with stuff"
)

# Good - specific
agent = Agent(
    name="billing_helper",
    description="Answers questions about invoices, payments, and refunds"
)
```

### No tool guidance in instructions
```python
# Bad - agent doesn't know when to use tools
instruction = "You have access to search and calculator tools"

# Good - clear tool usage guidance
instruction = """You have tools:
- search: Use for current info and facts
- calculator: Use for mathematical operations
Always use the appropriate tool."""
```

### Output schema with tools
```python
# Error: Can't use both
agent = Agent(
    output_schema=MySchema,  # Forces JSON output
    tools=[my_tool]  # Can't be used!
)
# Pick one: structured output OR tool usage
```

### Missing root_agent
```python
# Wrong - not named root_agent
my_main_agent = Agent(...)

# Correct
root_agent = Agent(...)
# or
root_agent = my_main_agent
```

### Deep agent hierarchies
```python
# Bad - 5 levels deep
root -> coordinator -> router -> specialist -> handler

# Better - 2-3 levels
root -> specialists (billing, support, sales)
```

## References

- [Agents Overview](https://google.github.io/adk-docs/agents/)
- [LLM Agents](https://google.github.io/adk-docs/agents/llm-agents/)
- [Workflow Agents](https://google.github.io/adk-docs/agents/workflow-agents/)
- [Multi-Agent Systems](https://google.github.io/adk-docs/agents/multi-agents/)
- [Agent Config](https://google.github.io/adk-docs/agents/config/)
