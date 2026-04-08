# ADK Runtime & Deployment

Use this reference for running and deploying ADK agents in development and production.

## When to Use This Reference

- Starting local development with ADK dev tools (web UI, CLI, API server)
- Running agents programmatically with Runner and InMemoryRunner
- Deploying agents to Cloud Run, GKE, or Vertex AI Agent Engine
- Configuring runtime environment variables and logging
- Understanding the event loop, session management, and resume patterns
- Troubleshooting deployment and runtime issues

## Local Development

ADK provides three built-in ways to run agents locally during development.

### Web UI (adk web)

The web UI provides an interactive browser-based interface for testing agents.

```bash
# Start the web UI (default: http://localhost:8000)
adk web <agents_directory>

# Example: agents are in current directory
adk web .

# Example: agents are in a specific folder
adk web ./my_agents

# Specify a custom port
adk web --port 8080 ./my_agents
```

**Features available:**
- Interactive chat interface with your agent
- Session management (create, switch, view sessions)
- Real-time event streaming (see agent reasoning, tool calls, responses)
- Multi-agent selection (choose which agent to talk to)
- State inspection (view session state in real time)
- Event trace visualization (see the full event chain)
- Artifact viewing

**How it works:**
- Scans the specified directory for agent packages (folders with `__init__.py` exporting `root_agent`)
- Starts a FastAPI server with a built-in React frontend
- Each agent package appears as a selectable option in the UI
- Sessions are stored in-memory by default (lost on restart)

### CLI Chat (adk run)

The CLI provides an interactive terminal chat for quick testing.

```bash
# Start CLI chat with an agent
adk run <agent_directory>

# Example
adk run ./my_agent

# Example with specific agent folder
adk run ./agents/weather_agent
```

**Features:**
- Simple text-based chat in the terminal
- Type messages and see agent responses inline
- Shows tool calls and their results
- Useful for quick smoke testing
- Type `exit` or `Ctrl+C` to quit

### API Server (adk api_server)

The API server exposes your agent as a REST API, suitable for integration with other services.

```bash
# Start the API server
adk api_server <agents_directory>

# With custom port
adk api_server --port 8000 ./my_agents

# Example
adk api_server .
```

**Available endpoints:**

```
POST /run              - Send a message and get a response
POST /run_sse          - Send a message and get Server-Sent Events stream
GET  /list-apps        - List available agent apps
POST /apps/{app}/users/{user}/sessions          - Create a new session
GET  /apps/{app}/users/{user}/sessions          - List sessions
GET  /apps/{app}/users/{user}/sessions/{session} - Get session details
DELETE /apps/{app}/users/{user}/sessions/{session} - Delete session
```

**Request format for /run and /run_sse:**

```json
{
  "app_name": "my_agent",
  "user_id": "user123",
  "session_id": "session456",
  "new_message": {
    "role": "user",
    "parts": [{"text": "Hello, what can you do?"}]
  },
  "streaming": false
}
```

**Response format:**

```json
{
  "events": [
    {
      "author": "my_agent",
      "content": {
        "role": "model",
        "parts": [{"text": "I can help you with..."}]
      },
      "is_final_response": true
    }
  ]
}
```

**SSE streaming (POST /run_sse):**
Returns Server-Sent Events where each event is a JSON object representing an agent event. This is what the web UI uses internally.

```bash
# Example curl request
curl -X POST http://localhost:8000/run_sse \
  -H "Content-Type: application/json" \
  -d '{
    "app_name": "my_agent",
    "user_id": "user1",
    "session_id": "sess1",
    "new_message": {
      "role": "user",
      "parts": [{"text": "Hello"}]
    },
    "streaming": true
  }'
```

**Integration with other services:**
- The API server is a standard FastAPI application
- Can be placed behind a reverse proxy (nginx, Envoy)
- Supports CORS for browser-based clients
- Can serve as the backend for custom frontends
- The same server is used when deploying to Cloud Run or GKE

## Programmatic Execution

For production applications, use the Runner class to execute agents programmatically.

### Runner Class

The `Runner` orchestrates agent execution, managing the interaction between the agent, session service, and artifact service.

```python
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.artifacts import InMemoryArtifactService

# Create services
session_service = InMemorySessionService()
artifact_service = InMemoryArtifactService()  # Optional

# Create runner
runner = Runner(
    agent=root_agent,
    app_name="my_app",
    session_service=session_service,
    artifact_service=artifact_service,  # Optional
)
```

### InMemoryRunner (Simplified)

For quick prototyping and testing, `InMemoryRunner` wraps Runner with in-memory services:

```python
from google.adk.runners import InMemoryRunner

# Quick setup - no explicit session service needed
runner = InMemoryRunner(
    agent=root_agent,
    app_name="my_app",
)
```

### Session Management

```python
from google.adk.sessions import InMemorySessionService

session_service = InMemorySessionService()

# Create a session
session = await session_service.create_session(
    app_name="my_app",
    user_id="user123",
    session_id="session456",  # Optional - auto-generated if omitted
    state={"initial_key": "initial_value"},  # Optional initial state
)

# Get an existing session
session = await session_service.get_session(
    app_name="my_app",
    user_id="user123",
    session_id="session456",
)

# List sessions for a user
sessions = await session_service.list_sessions(
    app_name="my_app",
    user_id="user123",
)

# Delete a session
await session_service.delete_session(
    app_name="my_app",
    user_id="user123",
    session_id="session456",
)
```

**Production session services:**
- `InMemorySessionService` - Development only (data lost on restart)
- `DatabaseSessionService` - Persistent storage with database backends
- `VertexAiSessionService` - Managed by Vertex AI Agent Engine
- Custom implementations extending `BaseSessionService`

### Event Handling

Events are the communication units in ADK. Every interaction produces a stream of events.

```python
from google.genai import types

# Create user message
user_message = types.Content(
    role="user",
    parts=[types.Part(text="What's the weather in London?")]
)

# Run and process events
async for event in runner.run_async(
    user_id="user123",
    session_id="session456",
    new_message=user_message,
):
    # Event properties
    print(f"Author: {event.author}")         # Which agent produced this
    print(f"Is final: {event.is_final_response()}")  # Last event?

    # Check for text content
    if event.content and event.content.parts:
        for part in event.content.parts:
            if part.text:
                print(f"Text: {part.text}")
            if part.function_call:
                print(f"Tool call: {part.function_call.name}")
            if part.function_response:
                print(f"Tool result: {part.function_response.response}")

    # Check for actions (state changes, transfers)
    if event.actions:
        if event.actions.state_delta:
            print(f"State changes: {event.actions.state_delta}")
        if event.actions.transfer_to_agent:
            print(f"Transfer to: {event.actions.transfer_to_agent}")
```

**Event types you will encounter:**
- **User message echo**: The user's input echoed back
- **Model response**: LLM-generated text or tool calls
- **Tool call**: Agent requesting a tool execution
- **Tool response**: Result from tool execution
- **Final response**: The last event with the agent's answer
- **State delta**: Changes to session state
- **Transfer**: Agent delegation to another agent

### Async Patterns

ADK is built on async/await. Here are the key patterns:

```python
import asyncio
from google.adk.agents import Agent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types

# Setup
agent = Agent(
    name="my_agent",
    model="gemini-2.5-flash",
    instruction="You are a helpful assistant."
)

session_service = InMemorySessionService()
runner = Runner(
    agent=agent,
    app_name="my_app",
    session_service=session_service,
)

async def chat(user_text: str, user_id: str, session_id: str) -> str:
    """Send a message and return the final response text."""
    message = types.Content(
        role="user",
        parts=[types.Part(text=user_text)]
    )

    final_text = ""
    async for event in runner.run_async(
        user_id=user_id,
        session_id=session_id,
        new_message=message,
    ):
        if event.is_final_response() and event.content and event.content.parts:
            final_text = event.content.parts[0].text
    return final_text

async def main():
    # Create session
    session = await session_service.create_session(
        app_name="my_app",
        user_id="user1",
    )

    # Multi-turn conversation
    response1 = await chat("Hello!", "user1", session.id)
    print(f"Agent: {response1}")

    response2 = await chat("What did I just say?", "user1", session.id)
    print(f"Agent: {response2}")

# Run
asyncio.run(main())
```

### Full Working Example

```python
"""Complete working example: Agent with tools, session management, and multi-turn."""
import asyncio
from google.adk.agents import Agent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types

# 1. Define tools
def get_weather(city: str) -> dict:
    """Get the current weather for a city.

    Args:
        city: The name of the city to get weather for.

    Returns:
        Dictionary with temperature and conditions.
    """
    weather_data = {
        "london": {"temp_c": 12, "condition": "cloudy"},
        "tokyo": {"temp_c": 22, "condition": "sunny"},
        "new york": {"temp_c": 18, "condition": "partly cloudy"},
    }
    data = weather_data.get(city.lower(), {"temp_c": 20, "condition": "unknown"})
    return {"city": city, **data}

def convert_temperature(temp_c: float, to_unit: str) -> dict:
    """Convert temperature between Celsius and Fahrenheit.

    Args:
        temp_c: Temperature in Celsius.
        to_unit: Target unit ('fahrenheit' or 'celsius').

    Returns:
        Dictionary with the converted temperature.
    """
    if to_unit.lower() == "fahrenheit":
        return {"temp_f": round(temp_c * 9/5 + 32, 1), "unit": "F"}
    return {"temp_c": temp_c, "unit": "C"}

# 2. Create agent
weather_agent = Agent(
    name="weather_assistant",
    model="gemini-2.5-flash",
    description="A weather assistant that provides weather info and temperature conversions",
    instruction="""You are a weather assistant.
    - Use get_weather to look up current weather
    - Use convert_temperature if the user wants Fahrenheit
    - Always include both the temperature and conditions in your response
    - Be concise but friendly
    """,
    tools=[get_weather, convert_temperature],
)

# 3. Setup runtime
session_service = InMemorySessionService()
runner = Runner(
    agent=weather_agent,
    app_name="weather_app",
    session_service=session_service,
)

# 4. Helper function
async def send_message(user_id: str, session_id: str, text: str) -> str:
    message = types.Content(
        role="user",
        parts=[types.Part(text=text)]
    )
    final_response = ""
    async for event in runner.run_async(
        user_id=user_id,
        session_id=session_id,
        new_message=message,
    ):
        if event.is_final_response() and event.content:
            for part in event.content.parts:
                if part.text:
                    final_response += part.text
    return final_response

# 5. Run conversation
async def main():
    session = await session_service.create_session(
        app_name="weather_app",
        user_id="demo_user",
    )
    sid = session.id

    # Multi-turn
    r1 = await send_message("demo_user", sid, "What's the weather in London?")
    print(f"Agent: {r1}\n")

    r2 = await send_message("demo_user", sid, "Convert that to Fahrenheit")
    print(f"Agent: {r2}\n")

    r3 = await send_message("demo_user", sid, "How about Tokyo?")
    print(f"Agent: {r3}\n")

if __name__ == "__main__":
    asyncio.run(main())
```

## Cloud Run Deployment

Cloud Run is the recommended way to deploy ADK agents as scalable, serverless containers.

### Project Structure

```
my-agent-app/
├── my_agent/
│   ├── __init__.py        # Exports root_agent
│   └── agent.py           # Agent definition
├── .env                   # Environment variables (local only, not in container)
├── Dockerfile             # Container configuration
├── requirements.txt       # Python dependencies
└── app.py                 # Optional: custom FastAPI app
```

### Dockerfile Setup

```dockerfile
# Use Python slim image
FROM python:3.12-slim

# Set working directory
WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy agent code
COPY . .

# Expose port (Cloud Run uses PORT env var)
EXPOSE 8080

# Start the ADK API server
# The agents_dir should point to the parent directory containing agent packages
CMD ["adk", "api_server", "--port", "8080", "."]
```

**Alternative Dockerfile using uvicorn directly:**

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8080

# Using uvicorn for more control
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8080"]
```

**requirements.txt:**

```
google-adk>=0.5.0
google-cloud-aiplatform>=1.74.0
```

### Custom FastAPI App (app.py)

For more control over the server, create a custom FastAPI app:

```python
"""Custom FastAPI server for ADK agent."""
import os
from google.adk.cli import fast_api

# Get the directory containing agent packages
AGENTS_DIR = os.path.dirname(os.path.abspath(__file__))

# Create the FastAPI app using ADK's built-in helper
# This creates the same app as `adk api_server`
app = fast_api.get_fast_api_app(agents_dir=AGENTS_DIR)
```

### Environment Variables for Cloud Run

```bash
# Required for Vertex AI backend
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_CLOUD_LOCATION=us-central1

# OR for Google AI (Gemini API key)
GOOGLE_API_KEY=your-api-key

# Optional
GOOGLE_CLOUD_LOGGING=true     # Enable Cloud Logging integration
ADK_LOG_LEVEL=INFO            # Logging level
```

### Step-by-Step Deployment

```bash
# 1. Set project variables
export PROJECT_ID="your-project-id"
export REGION="us-central1"
export SERVICE_NAME="my-adk-agent"
export AR_REPO="adk-agents"   # Artifact Registry repo name

# 2. Enable required APIs
gcloud services enable \
    run.googleapis.com \
    cloudbuild.googleapis.com \
    artifactregistry.googleapis.com \
    aiplatform.googleapis.com \
    --project=$PROJECT_ID

# 3. Create Artifact Registry repository (if not exists)
gcloud artifacts repositories create $AR_REPO \
    --repository-format=docker \
    --location=$REGION \
    --project=$PROJECT_ID

# 4. Build and push the container image
gcloud builds submit \
    --tag ${REGION}-docker.pkg.dev/${PROJECT_ID}/${AR_REPO}/${SERVICE_NAME}:latest \
    --project=$PROJECT_ID

# 5. Deploy to Cloud Run
gcloud run deploy $SERVICE_NAME \
    --image ${REGION}-docker.pkg.dev/${PROJECT_ID}/${AR_REPO}/${SERVICE_NAME}:latest \
    --region $REGION \
    --project $PROJECT_ID \
    --platform managed \
    --allow-unauthenticated \
    --memory 1Gi \
    --cpu 1 \
    --timeout 300 \
    --set-env-vars "GOOGLE_CLOUD_PROJECT=${PROJECT_ID},GOOGLE_CLOUD_LOCATION=${REGION}" \
    --min-instances 0 \
    --max-instances 10

# 6. Get the service URL
gcloud run services describe $SERVICE_NAME \
    --region $REGION \
    --project $PROJECT_ID \
    --format 'value(status.url)'
```

### Using a Service Account

```bash
# Create a service account for the agent
gcloud iam service-accounts create adk-agent-sa \
    --display-name="ADK Agent Service Account" \
    --project=$PROJECT_ID

# Grant Vertex AI permissions
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:adk-agent-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
    --role="roles/aiplatform.user"

# Deploy with the service account
gcloud run deploy $SERVICE_NAME \
    --image ${REGION}-docker.pkg.dev/${PROJECT_ID}/${AR_REPO}/${SERVICE_NAME}:latest \
    --region $REGION \
    --service-account adk-agent-sa@${PROJECT_ID}.iam.gserviceaccount.com \
    --set-env-vars "GOOGLE_CLOUD_PROJECT=${PROJECT_ID},GOOGLE_CLOUD_LOCATION=${REGION}"
```

### Scaling Considerations

- **Min instances**: Set `--min-instances 1` to avoid cold starts for latency-sensitive agents
- **Max instances**: Control costs with `--max-instances`; each instance handles one request at a time by default
- **Concurrency**: Set `--concurrency` to handle multiple requests per instance (if agent is stateless or sessions are externalized)
- **Memory**: Agents with large tool sets or context windows may need 2Gi+
- **Timeout**: Set `--timeout` high enough for long-running agent interactions (max 3600s)
- **CPU always allocated**: Use `--cpu-boost` for faster cold starts
- **Session storage**: Use a persistent session service (e.g., Firestore, Cloud SQL) in production -- `InMemorySessionService` does not persist across instances or restarts

### Testing Deployed Agent

```bash
# Get the service URL
SERVICE_URL=$(gcloud run services describe $SERVICE_NAME \
    --region $REGION --format 'value(status.url)')

# Send a request
curl -X POST "${SERVICE_URL}/run" \
  -H "Content-Type: application/json" \
  -d '{
    "app_name": "my_agent",
    "user_id": "test_user",
    "session_id": "test_session",
    "new_message": {
      "role": "user",
      "parts": [{"text": "Hello!"}]
    }
  }'
```

## GKE Deployment

Google Kubernetes Engine (GKE) provides more control over infrastructure, networking, and scaling for production agent deployments.

### When to Choose GKE over Cloud Run

- Need persistent connections (WebSockets, gRPC streaming)
- Require GPU access for local model inference
- Need fine-grained network policies
- Want to co-locate agents with other microservices
- Require custom autoscaling policies
- Need to run in a specific VPC

### Kubernetes Configuration

**deployment.yaml:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: adk-agent
  labels:
    app: adk-agent
spec:
  replicas: 2
  selector:
    matchLabels:
      app: adk-agent
  template:
    metadata:
      labels:
        app: adk-agent
    spec:
      serviceAccountName: adk-agent-ksa
      containers:
        - name: adk-agent
          image: us-central1-docker.pkg.dev/PROJECT_ID/adk-agents/my-agent:latest
          ports:
            - containerPort: 8080
              protocol: TCP
          env:
            - name: GOOGLE_CLOUD_PROJECT
              value: "your-project-id"
            - name: GOOGLE_CLOUD_LOCATION
              value: "us-central1"
            - name: PORT
              value: "8080"
          resources:
            requests:
              cpu: "500m"
              memory: "512Mi"
            limits:
              cpu: "1000m"
              memory: "1Gi"
          readinessProbe:
            httpGet:
              path: /list-apps
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /list-apps
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 10
```

**service.yaml:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: adk-agent-service
spec:
  type: ClusterIP
  selector:
    app: adk-agent
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
```

**ingress.yaml (optional, for external access):**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: adk-agent-ingress
  annotations:
    kubernetes.io/ingress.class: "gce"
    kubernetes.io/ingress.global-static-ip-name: "adk-agent-ip"
spec:
  rules:
    - host: agent.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: adk-agent-service
                port:
                  number: 80
```

### Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: adk-agent-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: adk-agent
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

### Workload Identity (GKE Authentication)

```bash
# 1. Create a Kubernetes service account
kubectl create serviceaccount adk-agent-ksa

# 2. Create a Google Cloud service account
gcloud iam service-accounts create adk-agent-gsa \
    --project=$PROJECT_ID

# 3. Grant Vertex AI permissions
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:adk-agent-gsa@${PROJECT_ID}.iam.gserviceaccount.com" \
    --role="roles/aiplatform.user"

# 4. Bind Kubernetes SA to Google Cloud SA (Workload Identity)
gcloud iam service-accounts add-iam-policy-binding \
    adk-agent-gsa@${PROJECT_ID}.iam.gserviceaccount.com \
    --role="roles/iam.workloadIdentityUser" \
    --member="serviceAccount:${PROJECT_ID}.svc.id.goog[default/adk-agent-ksa]"

# 5. Annotate the Kubernetes service account
kubectl annotate serviceaccount adk-agent-ksa \
    iam.gke.io/gcp-service-account=adk-agent-gsa@${PROJECT_ID}.iam.gserviceaccount.com
```

### Step-by-Step GKE Deployment

```bash
# 1. Build and push the container image (same as Cloud Run)
gcloud builds submit \
    --tag ${REGION}-docker.pkg.dev/${PROJECT_ID}/${AR_REPO}/${SERVICE_NAME}:latest

# 2. Get GKE credentials
gcloud container clusters get-credentials CLUSTER_NAME \
    --region $REGION --project $PROJECT_ID

# 3. Apply Kubernetes manifests
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml     # If external access needed
kubectl apply -f hpa.yaml         # If autoscaling needed

# 4. Check deployment status
kubectl get pods -l app=adk-agent
kubectl get service adk-agent-service
kubectl logs -l app=adk-agent --tail=50

# 5. Port-forward for local testing
kubectl port-forward service/adk-agent-service 8080:80
```

### Scaling and Monitoring

- **Pod Disruption Budget**: Ensure availability during updates
  ```yaml
  apiVersion: policy/v1
  kind: PodDisruptionBudget
  metadata:
    name: adk-agent-pdb
  spec:
    minAvailable: 1
    selector:
      matchLabels:
        app: adk-agent
  ```
- **Monitoring**: Use Google Cloud Monitoring + Prometheus annotations
- **Logging**: Structured JSON logs are automatically collected by Cloud Logging on GKE
- **Rolling updates**: Default strategy; set `maxUnavailable: 0` and `maxSurge: 1` for zero-downtime deployments

## Vertex AI Agent Engine

Agent Engine is a fully managed runtime on Vertex AI for deploying, managing, and scaling ADK agents without managing infrastructure.

### What It Is

- Managed hosting for ADK agents on Google Cloud
- Handles scaling, monitoring, and infrastructure automatically
- Integrates natively with Vertex AI services (Gemini, Search, RAG, etc.)
- Provides managed session and state persistence
- Supports evaluation and A/B testing workflows

### Prerequisites

```bash
# Enable required APIs
gcloud services enable \
    aiplatform.googleapis.com \
    --project=$PROJECT_ID

# Install required packages
pip install google-adk google-cloud-aiplatform>=1.74.0
```

### Setup and Configuration

```python
import vertexai
from vertexai import agent_engines

# Initialize Vertex AI
vertexai.init(
    project="your-project-id",
    location="us-central1",
)
```

### Deployment Steps

**1. Define your agent (same as normal ADK):**

```python
from google.adk.agents import Agent

def get_weather(city: str) -> dict:
    """Get the current weather for a city."""
    return {"city": city, "temp_c": 20, "condition": "sunny"}

root_agent = Agent(
    name="weather_agent",
    model="gemini-2.5-flash",
    description="A weather information agent",
    instruction="Help users with weather queries using the get_weather tool.",
    tools=[get_weather],
)
```

**2. Create an Agent Engine application:**

```python
from vertexai import agent_engines

# Create the Agent Engine app with your ADK agent
# This packages, uploads, and deploys the agent
remote_app = agent_engines.create(
    agent_engine=root_agent,
    requirements=[
        "google-adk>=0.5.0",
        # Add any other dependencies your agent needs
    ],
    display_name="Weather Agent",
    description="Production weather agent",
    # Optional: specify extra packages
    extra_packages=[],
)

print(f"Resource name: {remote_app.resource_name}")
# Format: projects/{project}/locations/{location}/reasoningEngines/{id}
```

**3. Test the deployed agent:**

```python
# Query the deployed agent
response = remote_app.query(
    user_id="test_user",
    session_id="test_session",
    message="What's the weather in London?",
)
print(response)
```

**4. Using streaming:**

```python
# Stream responses from the deployed agent
for event in remote_app.stream_query(
    user_id="test_user",
    session_id="test_session",
    message="What's the weather in Tokyo?",
):
    print(event)
```

### Management and Monitoring

**List deployed agents:**

```python
# List all Agent Engine apps
apps = agent_engines.list()
for app in apps:
    print(f"{app.display_name}: {app.resource_name}")
```

**Get a specific deployed agent:**

```python
# Retrieve by resource name
remote_app = agent_engines.get(
    "projects/your-project/locations/us-central1/reasoningEngines/12345"
)
```

**Update a deployed agent:**

```python
# Update the agent (redeploy with new code/config)
remote_app = agent_engines.create(
    agent_engine=updated_agent,
    requirements=["google-adk>=0.5.0"],
    display_name="Weather Agent v2",
)
```

**Delete a deployed agent:**

```python
# Delete the Agent Engine app
remote_app.delete()

# Or by resource name
agent_engines.delete(
    "projects/your-project/locations/us-central1/reasoningEngines/12345"
)
```

**Using gcloud CLI:**

```bash
# List deployed agents
gcloud ai reasoning-engines list \
    --project=$PROJECT_ID \
    --region=$REGION

# Describe a specific agent
gcloud ai reasoning-engines describe REASONING_ENGINE_ID \
    --project=$PROJECT_ID \
    --region=$REGION

# Delete a deployed agent
gcloud ai reasoning-engines delete REASONING_ENGINE_ID \
    --project=$PROJECT_ID \
    --region=$REGION
```

### Agent Engine Considerations

- **Session persistence**: Agent Engine manages sessions automatically -- no need for external session storage
- **Scaling**: Automatically scales based on traffic
- **Cold starts**: First request after idle may be slower
- **Dependencies**: All Python dependencies must be specified in `requirements` parameter
- **Region**: Must match the Vertex AI region where models are available
- **Limits**: Check Vertex AI quotas for reasoning engine requests per minute

## Runtime Configuration

### Environment Variables

```bash
# ---- Authentication ----
GOOGLE_API_KEY=...                    # Google AI (Gemini) API key
GOOGLE_CLOUD_PROJECT=...              # GCP project ID (for Vertex AI)
GOOGLE_CLOUD_LOCATION=...             # GCP region (e.g., us-central1)
ANTHROPIC_API_KEY=...                 # For Claude models via Anthropic
OPENAI_API_KEY=...                    # For OpenAI models

# ---- Runtime ----
ADK_LOG_LEVEL=INFO                    # Logging level: DEBUG, INFO, WARNING, ERROR
ADK_DEBUG=true                        # Enable debug mode (verbose logging)
GOOGLE_CLOUD_LOGGING=true             # Route logs to Cloud Logging

# ---- Server ----
PORT=8080                             # Server port (Cloud Run sets this automatically)
HOST=0.0.0.0                          # Server bind address
```

### Logging Configuration

ADK uses Python's standard `logging` module. Configure it as needed:

```python
import logging

# Set ADK log level
logging.getLogger("google.adk").setLevel(logging.DEBUG)

# Log all events (very verbose)
logging.getLogger("google.adk.runners").setLevel(logging.DEBUG)

# Log only tool calls
logging.getLogger("google.adk.tools").setLevel(logging.DEBUG)
```

**Structured logging for Cloud Run / GKE:**

```python
import logging
import json
import sys

class JsonFormatter(logging.Formatter):
    def format(self, record):
        log_obj = {
            "severity": record.levelname,
            "message": record.getMessage(),
            "module": record.module,
            "timestamp": self.formatTime(record),
        }
        return json.dumps(log_obj)

handler = logging.StreamHandler(sys.stdout)
handler.setFormatter(JsonFormatter())
logging.getLogger().addHandler(handler)
logging.getLogger().setLevel(logging.INFO)
```

### Debug Mode

Enable debug mode for development to see detailed event traces:

```bash
# Via environment variable
ADK_DEBUG=true adk web .

# Or set log level
ADK_LOG_LEVEL=DEBUG adk web .
```

In code:

```python
import logging
logging.basicConfig(level=logging.DEBUG)
# This will print all ADK internal events, LLM calls, tool invocations, etc.
```

## Event Loop and Resume Patterns

### How Events Work

The ADK event loop follows this cycle:

1. **User sends message** -> Creates a `Content` with `role="user"`
2. **Runner passes to agent** -> Agent receives message + session context
3. **Agent calls LLM** -> LLM produces response (text, tool calls, or transfer)
4. **Events are yielded** -> Each step produces an `Event` object
5. **Tool execution** -> If LLM requested tools, Runner executes them
6. **Loop continues** -> Steps 3-5 repeat until LLM produces final response
7. **Final response** -> Event with `is_final_response() == True`

```
User Message -> Runner -> Agent -> LLM
                                    |
                          [text response] -> Final Event
                          [tool call] -> Execute Tool -> Back to LLM
                          [transfer] -> Switch Agent -> Back to LLM
```

### Resuming Sessions

Sessions maintain full conversation history. Resume a conversation by using the same session ID:

```python
# First interaction
session = await session_service.create_session(
    app_name="my_app",
    user_id="user1",
)
sid = session.id

# Send first message
async for event in runner.run_async(
    user_id="user1", session_id=sid,
    new_message=types.Content(role="user", parts=[types.Part(text="My name is Alice")])
):
    pass  # Process events

# Later... resume the same session
# The agent remembers the conversation context
async for event in runner.run_async(
    user_id="user1", session_id=sid,
    new_message=types.Content(role="user", parts=[types.Part(text="What's my name?")])
):
    if event.is_final_response() and event.content:
        print(event.content.parts[0].text)  # "Your name is Alice"
```

### Error Recovery

Handle errors gracefully during agent execution:

```python
async def safe_run(runner, user_id, session_id, message):
    """Run agent with error handling."""
    try:
        results = []
        async for event in runner.run_async(
            user_id=user_id,
            session_id=session_id,
            new_message=message,
        ):
            results.append(event)
            if event.is_final_response():
                return event
        return results[-1] if results else None

    except Exception as e:
        # Log the error
        print(f"Agent execution error: {e}")

        # Option 1: Return error message to user
        return {"error": str(e)}

        # Option 2: Retry with a new session
        # new_session = await session_service.create_session(...)
        # return await safe_run(runner, user_id, new_session.id, message)
```

**Tool-level error handling (preferred):**

```python
def risky_tool(query: str) -> dict:
    """A tool that might fail."""
    try:
        result = external_api_call(query)
        return {"status": "success", "data": result}
    except TimeoutError:
        return {"status": "error", "message": "API timed out, please try again"}
    except Exception as e:
        return {"status": "error", "message": f"Unexpected error: {str(e)}"}
```

## Best Practices

### Local Development
- Use `adk web` for interactive development and debugging -- the event trace visualization is invaluable
- Use `adk run` for quick smoke tests from the terminal
- Use `adk api_server` when building a custom frontend or integrating with other services
- Always test with `InMemorySessionService` locally before deploying with persistent storage

### Deployment Strategy
- Start with **Cloud Run** for simplest deployment path and automatic scaling
- Move to **GKE** when you need WebSocket support, GPU access, or complex networking
- Use **Agent Engine** when you want fully managed infrastructure and native Vertex AI integration
- Always use a **service account** with minimal permissions (principle of least privilege)

### Session Management
- Use `InMemorySessionService` only for development and testing
- In production, use persistent session storage (database-backed or Agent Engine managed)
- Generate meaningful session IDs that help with debugging (e.g., include user ID and timestamp)
- Clean up old sessions periodically to avoid storage bloat

### Performance
- Set appropriate `max_output_tokens` to control response time and cost
- Use `include_contents='none'` for stateless agents that do not need history
- For high-traffic deployments, set Cloud Run min-instances >= 1 to avoid cold starts
- Monitor LLM token usage to optimize costs

### Security
- Never hardcode API keys in source code or Dockerfiles
- Use Secret Manager or environment variables for credentials
- Use Workload Identity on GKE instead of service account key files
- Set `--no-allow-unauthenticated` on Cloud Run for private agents
- Validate and sanitize all user inputs in tools

## Common Pitfalls

### InMemorySessionService in production
```python
# WRONG - sessions lost on restart/scale
session_service = InMemorySessionService()  # Only for dev!

# RIGHT - use persistent storage in production
from google.adk.sessions import DatabaseSessionService
session_service = DatabaseSessionService(db_url="postgresql://...")
```

### Missing PORT environment variable on Cloud Run
```dockerfile
# WRONG - hardcoded port may not match Cloud Run's PORT env var
CMD ["adk", "api_server", "--port", "3000", "."]

# RIGHT - use the PORT env var that Cloud Run provides (defaults to 8080)
CMD ["adk", "api_server", "--port", "8080", "."]
```

### Forgetting to export root_agent in __init__.py
```python
# __init__.py - WRONG: missing export
from .agent import my_agent

# __init__.py - RIGHT: export root_agent
from .agent import root_agent
```

### Not awaiting async session operations
```python
# WRONG - missing await
session = session_service.create_session(app_name="app", user_id="u1")

# RIGHT
session = await session_service.create_session(app_name="app", user_id="u1")
```

### Not handling streaming events properly
```python
# WRONG - only checking text, missing tool calls
async for event in runner.run_async(...):
    print(event.content.parts[0].text)  # Crashes on tool call events

# RIGHT - check event type
async for event in runner.run_async(...):
    if event.is_final_response() and event.content and event.content.parts:
        for part in event.content.parts:
            if part.text:
                print(part.text)
```

### Agent Engine dependency issues
```python
# WRONG - missing dependencies
remote_app = agent_engines.create(
    agent_engine=root_agent,
    requirements=["google-adk"],  # Missing version pin and other deps
)

# RIGHT - pin versions and include all dependencies
remote_app = agent_engines.create(
    agent_engine=root_agent,
    requirements=[
        "google-adk>=0.5.0",
        "requests>=2.31.0",
        "beautifulsoup4>=4.12.0",
        # Include ALL packages your tools import
    ],
)
```

### Dockerfile not copying .env (and should not)
```dockerfile
# WRONG - never bake secrets into images
COPY .env /app/.env

# RIGHT - pass via environment variables at deploy time
# gcloud run deploy ... --set-env-vars "KEY=VALUE"
# or use Secret Manager
```

## References

- [ADK Runtime Documentation](https://google.github.io/adk-docs/runtime/)
- [Cloud Run Deployment](https://google.github.io/adk-docs/deploy/cloud-run/)
- [GKE Deployment](https://google.github.io/adk-docs/deploy/gke/)
- [Vertex AI Agent Engine](https://google.github.io/adk-docs/deploy/agent-engine/)
- [Sessions and State](https://google.github.io/adk-docs/sessions/)
- [Events System](https://google.github.io/adk-docs/events/)
- [ADK GitHub Repository](https://github.com/google/adk-python)
- [Cloud Run Documentation](https://cloud.google.com/run/docs)
- [GKE Documentation](https://cloud.google.com/kubernetes-engine/docs)
- [Vertex AI Documentation](https://cloud.google.com/vertex-ai/docs)
