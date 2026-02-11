# ADK Reference Status

## Completed

1. **adk-fundamentals.md** - Core concepts, setup, getting started
2. **adk-agents.md** - Agent configuration, types, multi-agent systems
3. **adk-tools.md** - Built-in tools (Gemini API, Google Cloud, third-party)
4. **adk-custom-tools.md** - Function tools, MCP integration, OpenAPI tools, authentication
5. **adk-memory.md** - Session, state, memory management
6. **adk-runtime-deploy.md** - Runtime, Cloud Run, GKE, Agent Engine deployment
7. **adk-advanced.md** - MCP, A2A, streaming, artifacts, plugins, grounding

## Remaining

### Medium Priority

8. **adk-observability.md** - Monitoring and evaluation
   - Structured logging, Cloud Trace, BigQuery analytics
   - Third-party observability (AgentOps, Arize, Phoenix)
   - Evaluation framework (criteria, user simulation)

9. **adk-callbacks.md** - Lifecycle hooks and control
   - Callback types (before/after model, before/after tool)
   - Events system, validation and modification patterns

10. **adk-models.md** - LLM provider integrations
    - Gemini, Claude, Vertex AI, Ollama, vLLM, LiteLLM, Apigee

### Lower Priority

11. **adk-tutorials.md** - End-to-end examples and walkthroughs
12. **adk-safety.md** - Input validation, output filtering, rate limiting, guardrails

## Source URLs for Remaining Skills

### adk-observability.md
- https://google.github.io/adk-docs/evaluate/
- https://google.github.io/adk-docs/observability/logging/
- https://google.github.io/adk-docs/observability/cloud-trace/

### adk-callbacks.md
- https://google.github.io/adk-docs/callbacks/
- https://google.github.io/adk-docs/callbacks/types-of-callbacks/
- https://google.github.io/adk-docs/events/

### adk-models.md
- https://google.github.io/adk-docs/agents/models/
- https://google.github.io/adk-docs/agents/models/google-gemini/
- https://google.github.io/adk-docs/agents/models/anthropic/
