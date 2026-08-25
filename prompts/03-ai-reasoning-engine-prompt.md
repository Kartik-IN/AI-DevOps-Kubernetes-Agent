# Phase 03 — AI Reasoning Engine Prompt

## Goal

Turn the Kubernetes evidence collected by Phase 02 into an intelligent diagnosis.

The AI agent should behave like a **Senior Kubernetes SRE** and:

1. Understand Kubernetes failures
2. Correlate logs + events + deployment state
3. Find root cause
4. Suggest fixes
5. Generate a confidence score

## Components

Implement:

- Prompt Builder
- LLM Client
- Root Cause Analyzer
- Fix Recommendation Engine
- Confidence Engine

## Prompt Builder

Build a structured Kubernetes troubleshooting prompt containing:

- Pod Status
- Logs
- Events
- Deployment Health
- Networking Findings

The model should return:

1. Root Cause
2. Explanation
3. Suggested Fix
4. `kubectl` Commands
5. Prevention Recommendation
6. Confidence Score

The prompt should be structured and deterministic, avoiding vague answers.

## LLM Client

Use **OpenRouter**.

Read configuration from:

```env
OPENROUTER_API_KEY=
OPENROUTER_MODEL=
```

Requirements:

- Use HTTPX
- Add timeout handling
- Handle API failures gracefully
- Add retries
- Log errors cleanly
- Never expose secrets

## Root Cause Analyzer

Correlate the investigation payload rather than blindly summarizing a single log line.

Example input:

```json
{
  "pods": {
    "status": "CrashLoopBackOff"
  },
  "logs": {
    "error": "DATABASE_URL missing"
  }
}
```

Expected reasoning:

```text
Root Cause:
Application failed because DATABASE_URL environment variable is missing.
```

## Fix Recommendation Engine

Generate actionable, Kubernetes-specific recommendations.

Example:

```text
Suggested Fix:
Add DATABASE_URL environment variable.

kubectl command:
kubectl edit deployment payment-service
```

Recommendations should be practical, beginner friendly, and explainable.

## Confidence Engine

Return a confidence score such as:

```text
Confidence: 92%
```

Support the score with evidence, for example:

```text
High confidence because:
- Pod state = CrashLoopBackOff
- Logs clearly show missing env var
- Restart count confirms startup failure
```

## FastAPI Integration

Extend:

```http
POST /investigate
```

Flow:

```text
Collect Kubernetes Evidence
        ↓
Send to AI Agent
        ↓
LLM Reasoning
        ↓
Root Cause Analysis
        ↓
Suggested Fix
        ↓
Return Diagnosis
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

## Constraints

Do not implement:

- Authentication
- Investigation history
- Realtime updates
- Frontend changes
- Deployment

Only build the AI reasoning layer and extend existing functionality without breaking it.
