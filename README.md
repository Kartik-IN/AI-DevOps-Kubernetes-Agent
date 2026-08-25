# 🤖 AI Kubernetes Troubleshooting Agent

An AI-powered Kubernetes troubleshooting platform designed to investigate cluster failures, analyze Kubernetes evidence, identify likely root causes, and recommend fixes through an intelligent DevOps agent.

> **Project status:** Architecture / HLD phase. The design below defines the target platform and workflow; implementation details will be added as each component is built.

## 🎯 Goal

Build a real, publicly deployable AI-powered Kubernetes troubleshooting platform that can:

- Investigate Kubernetes failures
- Analyze logs, events, and cluster state
- Identify root causes
- Suggest practical fixes
- Store investigation history
- Provide realtime investigation progress
- Be deployed publicly as an application

## 🏗️ High-Level Architecture

```text
┌────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                     │
│                                                            │
│  Pods | Deployments | Services | Events | Logs            │
│                                                            │
│  Failure signals and troubleshooting evidence              │
└────────────────────────────────────────────────────────────┘
                              │
                              │ kubectl / Kubernetes API
                              ▼
┌────────────────────────────────────────────────────────────┐
│                  Investigation Layer                      │
│                                                            │
│  Pod Inspector                                             │
│  Logs Collector                                            │
│  Events Analyzer                                           │
│  Deployment Inspector                                      │
│  Network Inspector                                         │
└────────────────────────────────────────────────────────────┘
                              │
                              │ Structured Investigation Data
                              ▼
┌────────────────────────────────────────────────────────────┐
│                  AI Kubernetes Agent                      │
│                                                            │
│  Prompt Builder → LLM Reasoning → Root Cause Analyzer     │
│                         ↓                                  │
│               Fix Recommendation Engine                    │
│                         ↓                                  │
│                 Confidence Scoring                          │
└────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────┐
│                    InsForge Backend                       │
│                                                            │
│  Authentication | APIs | History | Realtime Updates       │
└────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────┐
│                   Frontend Dashboard                      │
│                                                            │
│  Investigate → Live Progress → Diagnosis → Suggested Fix  │
│                         ↓                                  │
│                  Investigation History                     │
└────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────┐
│                  InsForge Deployment                      │
│                                                            │
│              Public application deployment                 │
└────────────────────────────────────────────────────────────┘
```

## 🔄 End-to-End Workflow

1. The user clicks **Investigate Cluster** in the frontend.
2. The frontend sends an investigation request to the backend.
3. The FastAPI backend acts as the orchestration layer.
4. The user is authenticated through InsForge.
5. The Investigation Layer collects Kubernetes evidence:
   - Pod status
   - Container logs
   - Kubernetes events
   - Deployment and rollout status
   - Services, selectors, and networking information
6. The collected evidence is converted into structured investigation data.
7. The AI Kubernetes Agent builds a prompt containing the relevant evidence.
8. The LLM reasoning layer analyzes the Kubernetes failure using an OpenRouter API key managed through InsForge.
9. The Root Cause Analyzer correlates the available signals and identifies the primary issue.
10. The Fix Recommendation Engine produces suggested remediation steps, including possible `kubectl` commands or YAML changes.
11. A confidence percentage is returned for the diagnosis.
12. The investigation and its result are stored in InsForge.
13. Realtime updates communicate investigation progress to the frontend.
14. The frontend displays the diagnosis, confidence, recommended fix, and investigation history.
15. The complete application can be deployed publicly through InsForge.

## 🔎 Investigation Layer

### 1. Pod Inspector

Responsible for checking pod health and detecting common failure states such as:

- `CrashLoopBackOff`
- `Pending`
- `Error`
- Other unhealthy pod states

### 2. Logs Collector

Collects pod/container logs and captures application or runtime errors that can provide evidence for root-cause analysis.

### 3. Events Analyzer

Reads Kubernetes events to identify problems such as:

- Scheduling failures
- Image pull failures
- Resource-related failures
- Other cluster events associated with the incident

### 4. Deployment Inspector

Inspects deployment status and verifies rollout health to determine whether a workload has successfully reached the desired state.

### 5. Network Inspector

Investigates Kubernetes networking by checking services and validating selectors, with the goal of identifying service discovery and DNS/networking issues.

## 🧠 AI Kubernetes Agent

The AI layer is responsible for turning raw Kubernetes troubleshooting evidence into an actionable diagnosis.

### Prompt Builder

Converts structured investigation data into an LLM-ready troubleshooting prompt.

### LLM Reasoning Layer

Uses the OpenRouter API through a key managed by InsForge. The architecture supports models such as:

- Claude
- GPT
- DeepSeek

### Root Cause Analyzer

Correlates logs, events, pod state, deployment state, and other signals to identify the most likely primary failure.

### Fix Recommendation Engine

Produces practical remediation guidance, including:

- Suggested `kubectl` commands
- Recommended Kubernetes YAML changes
- Configuration corrections

### Confidence Scoring

Provides a confidence percentage alongside the diagnosis so users can understand how strongly the available evidence supports the proposed root cause.

## ☁️ InsForge Backend

InsForge provides the backend capabilities required by the platform:

- **Authentication** — user login and identity
- **API layer** — trigger investigations and return AI analysis
- **Investigation history** — persist previous incidents and root-cause reports
- **Realtime updates** — communicate live investigation progress

Example progress flow:

```text
✓ Checking pods
✓ Reading logs
✓ Analyzing events
✓ Finding root cause
```

## 🖥️ Frontend Dashboard

The dashboard is intended to provide a simple interface for operating the troubleshooting agent.

Example incident view:

```text
Incident: Payment Service Failure

Status: Investigating...

✓ Pods Checked
✓ Events Analyzed
✓ Logs Processed

Root Cause: ImagePullBackOff

Suggested Fix:
Update invalid image tag
```

The frontend will allow users to:

- Trigger an investigation
- Monitor realtime progress
- View the identified root cause
- Review confidence scoring
- Read suggested fixes
- Browse previous investigations

## 🧪 Example Failure Flow

**Incident:** Payment service unavailable

The agent investigates the workload and collects Kubernetes evidence:

```text
✓ Pod Status Checked
✓ Logs Collected
✓ Events Analyzed
```

The AI analysis may produce:

```text
Detected Problem:
CrashLoopBackOff

Root Cause:
DATABASE_URL environment variable missing

Confidence:
94%

Suggested Fix:
Update deployment.yaml and add secret reference

Prevention:
Add startup validation checks
```

The example demonstrates the intended flow from Kubernetes symptoms → evidence collection → AI reasoning → root cause → remediation → prevention guidance.

## ☸️ Supported Kubernetes Problems

The initial architecture targets the following troubleshooting scenarios:

| Problem | Investigation Focus |
|---|---|
| `CrashLoopBackOff` | Pod state, logs, configuration and startup failures |
| `ImagePullBackOff` | Image name/tag and Kubernetes events |
| `OOMKilled` | Container memory usage and resource configuration |
| Pending Pods | Scheduling, resources and cluster conditions |
| Resource Exhaustion | CPU/memory availability and workload requests/limits |
| Deployment Rollout Failures | Deployment status and rollout health |
| Service Selector Mismatch | Service selectors and workload labels |
| DNS Resolution Problems | Service discovery and Kubernetes networking |
| Readiness/Liveness Probe Failures | Probe configuration and application health |
| Networking Issues | Services, selectors and network-related evidence |

## 🛠️ Planned Technology Stack

| Layer | Technology / Component |
|---|---|
| Orchestration Backend | FastAPI |
| Container Platform | Kubernetes |
| Cluster Interaction | `kubectl` / Kubernetes API |
| AI Gateway | OpenRouter |
| AI Models | Claude / GPT / DeepSeek-compatible models |
| Backend Platform | InsForge |
| Frontend | Dashboard application |
| Deployment | InsForge public deployment |

> Technologies marked in the architecture are part of the target design. The README will be updated as implementation is completed and verified.

## 📁 Documentation

- [High-Level Design](docs/HLD.md) — detailed architecture and system responsibilities

## 🚧 Project Roadmap

- [ ] Build Kubernetes investigation layer
- [ ] Implement pod inspection
- [ ] Implement log collection
- [ ] Implement Kubernetes event analysis
- [ ] Implement deployment inspection
- [ ] Implement service/network inspection
- [ ] Build FastAPI orchestration API
- [ ] Integrate OpenRouter LLM reasoning
- [ ] Implement root-cause analysis workflow
- [ ] Implement fix recommendation engine
- [ ] Add confidence scoring
- [ ] Add InsForge authentication
- [ ] Store investigation history
- [ ] Add realtime investigation updates
- [ ] Build frontend dashboard
- [ ] Deploy the complete application publicly
- [ ] Add automated tests and production hardening

## 🔐 Security Considerations

The platform is designed with the following security considerations:

- Keep OpenRouter/API credentials outside source control.
- Use authenticated backend APIs for investigation requests.
- Apply least-privilege access when connecting the agent to Kubernetes.
- Treat AI-generated remediation commands as recommendations that should be reviewed before execution.
- Avoid exposing sensitive cluster logs or credentials through the frontend.

## 🔮 Future Improvements

Potential future capabilities include:

- Automated remediation with explicit approval gates
- More Kubernetes resource inspectors
- Historical incident correlation
- Better confidence scoring based on evidence quality
- Prevention recommendations based on recurring failures
- Automated test scenarios for common Kubernetes failures
- Multi-cluster investigation support
- Production-grade observability and audit logging

## 👨‍💻 Author

**Kartik Kale**

GitHub: [Kartik-IN](https://github.com/Kartik-IN)

---

> This project is being developed as a hands-on AI + DevOps + Kubernetes engineering project focused on applying LLM reasoning to real-world infrastructure troubleshooting.
