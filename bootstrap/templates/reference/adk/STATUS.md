# ADK Skills Creation Status

## Completed Skills

1. **adk-fundamentals.md** - Core concepts, setup, getting started
2. **adk-agents.md** - Agent configuration, types, multi-agent systems

## Skills to Create

### High Priority (Phase 1)

3. **adk-tools.md** - Built-in tools ecosystem
   - Gemini API tools (Code Execution, Computer use, Google Search)
   - Google Cloud tools (BigQuery, Vertex AI Search, RAG Engine, etc.)
   - Third-party tools (Qdrant, MongoDB, GitHub, etc.)
   - Tool selection strategies, limitations

4. **adk-custom-tools.md** - Creating custom tools
   - Function tools (Python/TypeScript/Go/Java)
   - MCP tools integration
   - OpenAPI tools
   - Authentication patterns
   - Action confirmations
   - Tool performance optimization

5. **adk-runtime-deploy.md** - Running and deploying agents
   - Local development (web UI, CLI, API server)
   - Cloud Run deployment
   - GKE deployment
   - Vertex AI Agent Engine
   - Runtime configuration
   - Event loop and resume patterns

6. **adk-memory.md** - Session, state, and memory
   - Session management
   - State persistence patterns
   - Memory across sessions
   - Context caching and compression
   - Session migration and rewind

### Medium Priority (Phase 2)

7. **adk-observability.md** - Monitoring and evaluation
   - Structured logging
   - Cloud Trace integration
   - BigQuery Agent Analytics
   - Third-party observability (AgentOps, Arize, Phoenix, etc.)
   - Evaluation framework (criteria, user simulation)

8. **adk-callbacks.md** - Lifecycle hooks and control
   - Callback types (before/after model, before/after tool)
   - Callback patterns and best practices
   - Events system
   - Validation and modification patterns

9. **adk-models.md** - LLM provider integrations
   - Gemini configuration
   - Claude (Anthropic)
   - Vertex AI hosted models
   - Ollama, vLLM, LiteLLM
   - Apigee AI Gateway
   - Model switching and selection

10. **adk-advanced.md** - Advanced features
    - MCP (Model Context Protocol)
    - A2A Protocol (Agent2Agent)
    - Bidi-streaming (live voice/video)
    - Streaming tools and configuration
    - Artifacts management
    - Apps (workflow management)
    - Plugins (reflect and retry)
    - Grounding (Google Search, Vertex AI Search)

### Lower Priority (Phase 3)

11. **adk-tutorials.md** - End-to-end examples
    - Multi-tool agent walkthrough
    - Agent team tutorial
    - Streaming agent tutorials
    - Visual Builder guide
    - Real-world use cases

12. **adk-safety.md** - Security and safety
    - Input validation
    - Output filtering
    - Rate limiting
    - Access control
    - Safety guardrails

## URLs to Fetch by Skill

### adk-tools.md
- https://google.github.io/adk-docs/tools/
- https://google.github.io/adk-docs/tools/gemini-api/
- https://google.github.io/adk-docs/tools/google-cloud/
- https://google.github.io/adk-docs/tools/third-party/

### adk-custom-tools.md
- https://google.github.io/adk-docs/tools-custom/function-tools/
- https://google.github.io/adk-docs/tools-custom/mcp-tools/
- https://google.github.io/adk-docs/tools-custom/openapi-tools/
- https://google.github.io/adk-docs/tools-custom/authentication/

### adk-runtime-deploy.md
- https://google.github.io/adk-docs/runtime/
- https://google.github.io/adk-docs/deploy/cloud-run/
- https://google.github.io/adk-docs/deploy/agent-engine/
- https://google.github.io/adk-docs/deploy/gke/

### adk-memory.md
- https://google.github.io/adk-docs/sessions/
- https://google.github.io/adk-docs/sessions/session/
- https://google.github.io/adk-docs/sessions/state/
- https://google.github.io/adk-docs/sessions/memory/
- https://google.github.io/adk-docs/context/

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

### adk-advanced.md
- https://google.github.io/adk-docs/mcp/
- https://google.github.io/adk-docs/a2a/intro/
- https://google.github.io/adk-docs/streaming/
- https://google.github.io/adk-docs/artifacts/
