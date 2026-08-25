# Phase 02 — Kubernetes Investigation Engine Prompt

## Goal

Build the **Kubernetes Investigation Layer** as the evidence-gathering stage before AI reasoning.

Architecture:

```text
Frontend
    ↓
FastAPI Backend (Orchestrator)
    ↓
Kubernetes Investigation Layer
        ├── Check Pods
        ├── Read Logs
        ├── Analyze Events
        ├── Inspect Deployments
        └── Check Networking
    ↓
AI Kubernetes Agent
```

The layer should behave like a **junior DevOps engineer collecting evidence**.

## Components

Implement:

- Pod Inspector
- Logs Collector
- Events Analyzer
- Deployment Inspector
- Network Inspector
- Kubectl Executor
- Investigation Service

Use **kubectl internally through Python subprocess**, not the Kubernetes Python SDK.

## Kubectl Executor

Requirements:

- Capture stdout/stderr
- Handle failures gracefully
- Return structured output
- Add logging

Example commands:

```bash
kubectl get pods -A
kubectl get events -A
kubectl logs <pod-name>
kubectl describe deployment <deployment-name>
kubectl get svc -A
```

## Pod Inspector

Detect:

- `CrashLoopBackOff`
- `ImagePullBackOff`
- `Pending`
- `Error`
- `OOMKilled`
- Stuck `ContainerCreating`

Example structure:

```json
{
  "healthy": false,
  "problematic_pods": [
    {
      "name": "payment-service",
      "namespace": "default",
      "status": "CrashLoopBackOff"
    }
  ]
}
```

## Logs Collector

Fetch logs for failed pods and focus on:

- Exceptions
- Connection failures
- Missing environment variables
- Image failures
- Startup errors

Keep logs concise.

## Events Analyzer

Detect and summarize:

- `FailedScheduling`
- `BackOff`
- `FailedMount`
- `FailedPull`
- `ErrImagePull`
- `Unhealthy`

## Deployment Inspector

Check:

- Available replicas
- Unavailable replicas
- Rollout failures
- Deployment conditions

Detect unhealthy deployments.

## Network Inspector

Check:

- Service existence
- Selector mismatch
- Missing endpoints
- DNS-related issues

## Investigation Service

Orchestrate:

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

Return:

```json
{
  "pods": {},
  "logs": {},
  "events": {},
  "deployments": {},
  "network": {}
}
```

## FastAPI API

Create:

```http
POST /investigate
```

At this phase it must only return structured Kubernetes evidence.

Example:

```json
{
  "status": "success",
  "investigation": {
    "pods": {},
    "logs": {},
    "events": {},
    "deployments": {},
    "network": {}
  }
}
```

## Constraints

Do not implement:

- OpenRouter
- LLM reasoning
- Root cause analysis
- Fix recommendation
- InsForge integration
- Authentication
- Realtime updates

Do not use the Kubernetes SDK. Do not break existing functionality.

## Expected Result

`POST /investigate` should collect Kubernetes troubleshooting evidence and return it in a structured payload.
