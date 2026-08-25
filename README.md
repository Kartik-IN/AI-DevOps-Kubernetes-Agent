# 🤖 AI Kubernetes Troubleshooting Agent

An AI-powered Kubernetes troubleshooting platform designed to investigate cluster failures, analyze Kubernetes evidence, identify likely root causes, and recommend fixes through an intelligent DevOps agent.

> **Project status:** Architecture + implementation plan. The repository documentation now captures the planned five-stage build from project foundation through end-to-end testing and deployment. Implementation status will be updated as components are completed and verified.

## 🎯 Goal

Build a real, publicly deployable AI-powered Kubernetes troubleshooting platform that can:

- Investigate Kubernetes failures
- Analyze logs, events, and cluster state
- Identify root causes
- Suggest practical fixes
- Store investigation history
- Provide realtime investigation progress
- Support selecting and investigating Kubernetes clusters available through the user's local kubeconfig
- Be deployed publicly as an application

## 💡 What Makes This Project Different

This is an **on-demand troubleshooting system**, not a Kubernetes controller or operator.

The platform is intended to behave like a DevOps/SRE assistant:

```text
User selects a Kubernetes cluster
        ↓
User clicks "Investigate Cluster"
        ↓
FastAPI orchestrates evidence collection
        ↓
Kubernetes investigation layer
        ↓
AI Kubernetes Agent
        ↓
LLM reasoning through OpenRouter
        ↓
Root cause + suggested fix
        ↓
Diagnosis displayed in dashboard
```

The investigation process is designed in two reasoning stages:

- **Investigation Layer:** behaves like a junior DevOps engineer collecting evidence.
- **AI Agent:** behaves like a senior Kubernetes SRE correlating evidence and reasoning about the incident.

## 🏗️ High-Level Architecture

```text
Frontend
    ↓
FastAPI Backend (Orchestrator)
    ↓
Kubernetes Investigation Layer
    ├── Pod Inspector
    ├── Logs Collector
    ├── Events Analyzer
    ├── Deployment Inspector
    ├── Network Inspector
    └── Kubectl Executor
    ↓
AI Kubernetes Agent
    ├── Prompt Builder
    ├── LLM Client
    ├── Root Cause Analyzer
    ├── Fix Recommendation Engine
    └── Confidence Engine
    ↓
OpenRouter LLM
    (API key supplied through InsForge environment)
    ↓
Root Cause + Suggested Fix
    ↓
InsForge
    ├── Authentication
    ├── Investigation History
    └── Realtime Updates
    ↓
Frontend Diagnosis
```

## ☸️ Multi-Cluster / Kubeconfig Support

A key product requirement is to make the dashboard aware of the Kubernetes clusters configured on the user's local machine.

The intended workflow is:

```text
Local kubeconfig
       ↓
Discover available contexts/clusters
       ↓
Display clusters in dashboard
       ↓
User selects a cluster
       ↓
Backend targets the selected kubeconfig context
       ↓
Run investigation against that cluster
       ↓
Return cluster-specific diagnosis
```

This means the application should **not assume a single hard-coded cluster**. Users should be able to see the available clusters represented by their kubeconfig and explicitly choose which cluster to investigate.

The selected cluster/context should be carried through the investigation request so that pods, logs, events, deployments, services, and networking evidence are collected from the intended cluster.

> **Security note:** kubeconfig files and credentials must never be committed to Git. Cluster access should use the user's configured credentials with appropriate least-privilege permissions.

## 🔎 Kubernetes Investigation Layer

The Investigation Layer is responsible only for evidence collection. It does **not** perform AI reasoning or root-cause analysis.

### Kubectl Executor

A reusable subprocess-based utility will execute supported `kubectl` commands while:

- Capturing stdout and stderr
- Handling command failures gracefully
- Returning structured output
- Logging execution details

Example commands include:

```bash
kubectl get pods -A
kubectl get events -A
kubectl logs <pod-name>
kubectl describe deployment <deployment-name>
kubectl get svc -A
```

The project intentionally uses **kubectl internally instead of the Kubernetes Python SDK** for this investigation layer.

### Pod Inspector

Checks pod health and detects conditions such as:

- `CrashLoopBackOff`
- `ImagePullBackOff`
- `Pending`
- `Error`
- `OOMKilled`
- Stuck `ContainerCreating`

### Logs Collector

Collects concise logs from failed pods and focuses on useful troubleshooting signals such as:

- Exceptions
- Connection failures
- Missing environment variables
- Image/startup failures

The collector should avoid returning thousands of irrelevant log lines.

### Events Analyzer

Reads Kubernetes events and summarizes important findings, including:

- `FailedScheduling`
- `BackOff`
- `FailedMount`
- `FailedPull`
- `ErrImagePull`
- `Unhealthy`

### Deployment Inspector

Checks deployment health using:

- Available replicas
- Unavailable replicas
- Rollout failures
- Deployment conditions

### Network Inspector

Investigates service and networking problems, including:

- Service existence
- Selector mismatch
- Missing endpoints
- DNS-related issues

### Investigation Service

Orchestrates the investigation components in a predictable sequence:

```text
Check Pods
    ↓
Collect Logs
    ↓
Analyze Events
    ↓
Inspect Deployments
    ↓
Check Networking
```

The resulting evidence is returned as a structured investigation payload:

```json
{
  "pods": {},
  "logs": {},
  "events": {},
  "deployments": {},
  "network": {}
}
```

## 🧠 AI Kubernetes Agent

The AI layer consumes the investigation payload and behaves like a **Senior Kubernetes SRE**.

### Prompt Builder

Builds a deterministic Kubernetes troubleshooting prompt containing:

- Pod status
- Logs
- Events
- Deployment health
- Networking findings

The requested diagnosis format contains:

1. Root Cause
2. Explanation
3. Suggested Fix
4. `kubectl` Commands
5. Prevention Recommendation
6. Confidence Score

### LLM Client

The planned LLM gateway is **OpenRouter**.

Configuration is supplied through environment variables:

```env
OPENROUTER_API_KEY=
OPENROUTER_MODEL=
```

The client uses HTTPX and is designed to include timeout handling, retries, clean error handling, and secure logging without exposing secrets.

### Root Cause Analyzer

The AI should correlate multiple evidence sources rather than blindly repeating a single log line.

Example:

```text
Pod state: CrashLoopBackOff
Logs: DATABASE_URL missing
Restart count: increasing

→ Root Cause:
Application failed because DATABASE_URL environment variable is missing.
```

### Fix Recommendation Engine

Produces practical Kubernetes-specific remediation, such as:

```text
Suggested Fix:
Add the missing DATABASE_URL environment variable.

kubectl command:
kubectl edit deployment payment-service
```

Recommendations should remain beginner friendly and explainable.

### Confidence Engine

Returns a confidence score supported by evidence.

Example:

```text
Confidence: 92%

High confidence because:
- Pod state = CrashLoopBackOff
- Logs clearly show missing environment variable
- Restart behavior confirms startup failure
```

## ☁️ InsForge Backend

InsForge is planned for application-level backend capabilities:

- **Authentication** — login and user sessions
- **Investigation History** — persist previous incidents
- **Realtime Updates** — publish investigation progress

FastAPI remains the primary orchestration layer for Kubernetes investigation and AI reasoning.

## 🖥️ Frontend Dashboard

The dashboard is intentionally minimal and professional.

### Main experience

```text
AI Kubernetes Agent

Select Kubernetes Cluster
[ cluster-a ▼ ]

[ Investigate Cluster ]

Investigation Status
✓ Checking Pods
✓ Reading Logs
✓ Analyzing Events
✓ Inspecting Deployments
✓ Checking Networking
✓ AI Reasoning
✓ Root Cause Found

Diagnosis

Root Cause:
CrashLoopBackOff

Fix:
Update environment variable

Confidence:
92%

Recent Investigations
---------------------
ImagePullBackOff
OOMKilled
```

The dashboard should handle:

- Login/session state
- Cluster selection
- Investigation loading state
- Realtime progress
- Diagnosis display
- API failures
- Empty results
- Timeouts
- Investigation history

## 🔌 API Surface

### Health

```http
GET /health
```

Expected response:

```json
{
  "status": "healthy",
  "service": "ai-kubernetes-agent"
}
```

### Investigation

```http
POST /investigate
```

The request should identify the selected Kubernetes cluster/context and trigger:

```text
Kubernetes Evidence Collection
        ↓
AI Reasoning
        ↓
Diagnosis
```

Example response:

```json
{
  "status": "success",
  "diagnosis": {
    "root_cause": "DATABASE_URL missing",
    "explanation": "Application cannot connect to DB.",
    "fix": "Add missing environment variable.",
    "kubectl_command": "kubectl edit deployment payment-service",
    "confidence": 92
  }
}
```

## 🛠️ Technology Stack

| Layer | Technology / Component |
|---|---|
| Backend | FastAPI |
| Runtime | Python 3.12+ |
| Backend Server | Uvicorn |
| Validation | Pydantic |
| Logging | Loguru |
| HTTP Client | HTTPX |
| Frontend | Next.js |
| Frontend Language | TypeScript |
| Styling | Tailwind CSS |
| API Client | Axios |
| Data Fetching | React Query |
| Containerization | Docker |
| Local Orchestration | Docker Compose |
| Kubernetes Interaction | `kubectl` / subprocess |
| AI Gateway | OpenRouter |
| Backend Platform | InsForge |

## 📁 Project Structure

Target monorepo structure:

```text
ai-kubernetes-agent/
│
├── backend/
│   ├── api/
│   ├── core/
│   ├── kubernetes/
│   ├── ai/
│   ├── services/
│   └── models/
│
├── frontend/
│   ├── components/
│   ├── services/
│   ├── hooks/
│   └── types/
│
├── docs/
│   └── HLD.md
│
├── prompts/
├── docker-compose.yml
└── README.md
```

## 🧪 Real Kubernetes Failure Testing

The project is designed to validate the agent against intentional failure scenarios.

### Scenario 1 — CrashLoopBackOff

Example cause: missing environment variable.

Expected diagnosis:

```text
Root Cause:
Missing environment variable

Suggested Fix:
Add the missing Secret/ConfigMap value
```

### Scenario 2 — ImagePullBackOff

Example cause: incorrect image tag.

Expected diagnosis:

```text
Root Cause:
Invalid image tag

Suggested Fix:
Update the deployment image
```

### Scenario 3 — OOMKilled

Example cause: low memory limits.

Expected diagnosis:

```text
Root Cause:
Container exceeded memory limit

Suggested Fix:
Increase memory requests/limits
```

### Scenario 4 — Service Selector Mismatch

Example cause: incorrect labels.

Expected diagnosis:

```text
Root Cause:
Service selector does not match pod labels

Suggested Fix:
Update the Service selector
```

## 🛡️ Reliability and Error Handling

The application should provide clear, beginner-friendly handling for:

- `kubectl` failures
- Cluster unreachable errors
- Missing kubeconfig
- Invalid kubeconfig context
- Insufficient Kubernetes permissions
- OpenRouter failures
- API timeouts
- Authentication failures
- No unhealthy resources
- Empty investigation results

Example user-facing message:

```text
Unable to connect to Kubernetes cluster.

Please verify:
- kubeconfig path
- selected cluster/context
- cluster access
- kubectl permissions
```

Stack traces and secrets should not be exposed through the UI.

## 🚧 Five-Stage Implementation Plan

### Phase 01 — Project Foundation

Build the foundation only:

- FastAPI backend
- Next.js frontend
- Dockerfiles
- Docker Compose
- Environment variables
- Modular folder structure
- `/health` endpoint
- Minimal dashboard

No Kubernetes logic, AI, authentication, realtime updates, or InsForge integration in this phase.

### Phase 02 — Kubernetes Investigation Engine

Build the evidence-gathering layer:

- Kubectl Executor
- Pod Inspector
- Logs Collector
- Events Analyzer
- Deployment Inspector
- Network Inspector
- Investigation Service
- `POST /investigate`

No AI reasoning yet.

### Phase 03 — AI Reasoning Engine

Add:

- Prompt Builder
- OpenRouter LLM Client
- Root Cause Analyzer
- Fix Recommendation Engine
- Confidence Engine

Extend `POST /investigate` from evidence-only output to diagnosis output.

### Phase 04 — Dashboard, Authentication and API Integration

Add:

- InsForge authentication
- Protected dashboard
- Cluster selection
- Realtime investigation progress
- Investigation history
- Frontend → FastAPI integration
- Diagnosis UI

### Phase 05 — End-to-End Integration, Testing and Deployment

Validate:

- Full frontend → backend → Kubernetes → AI → InsForge flow
- Error handling
- Loading and empty states
- Real Kubernetes failure scenarios
- Multi-cluster/kubeconfig selection
- Reliability
- Production deployment

## 🗺️ Implementation Roadmap

- [ ] Phase 01 — Project foundation
- [ ] Phase 02 — Kubernetes investigation engine
- [ ] Phase 03 — AI reasoning engine
- [ ] Phase 04 — Dashboard + authentication + API integration
- [ ] Phase 05 — End-to-end integration and real failure testing
- [ ] Discover clusters/contexts from local kubeconfig
- [ ] Allow users to select a target cluster
- [ ] Run investigations against the selected cluster
- [ ] Add automated tests
- [ ] Deploy the complete application publicly

## 🔐 Security Considerations

- Never commit `.env` files, kubeconfig files, tokens, or API keys.
- Keep OpenRouter credentials outside source control.
- Use least-privilege Kubernetes permissions.
- Validate the selected kubeconfig context before investigation.
- Treat AI-generated commands as recommendations requiring review before execution.
- Avoid exposing sensitive cluster logs or credentials in the frontend.

## 📚 Documentation

- [High-Level Design](docs/HLD.md) — architecture and system responsibilities
- Implementation prompts are maintained under [`prompts/`](prompts/) as the source specification for each development phase.

## 🔮 Future Improvements

Potential future capabilities include:

- Automated remediation with explicit approval gates
- More Kubernetes resource inspectors
- Historical incident correlation
- Improved evidence-based confidence scoring
- Prevention recommendations based on recurring failures
- Automated test scenarios for common Kubernetes failures
- Multi-cluster investigation at larger scale
- Production-grade observability and audit logging

## 👨‍💻 Author

**Kartik Kale**

GitHub: [Kartik-IN](https://github.com/Kartik-IN)

---

> This project is being developed as a hands-on AI + DevOps + Kubernetes engineering project focused on applying LLM reasoning to real-world infrastructure troubleshooting.
