# Phase 04 — Dashboard, Authentication and API Integration Prompt

## Goal

Turn the backend into a real application experience where users can:

```text
Login
  ↓
Open Dashboard
  ↓
Select Kubernetes Cluster
  ↓
Click Investigate
  ↓
See investigation progress
  ↓
Receive diagnosis
  ↓
View investigation history
```

Use InsForge for:

- Authentication
- Investigation History
- Realtime Updates

## Components

Implement:

- Minimal Dashboard
- Authentication with InsForge
- Realtime Investigation Progress
- Investigation History
- Frontend → FastAPI Integration
- Local kubeconfig cluster/context selection

## Authentication

Requirements:

- Login support
- Protected dashboard
- User session handling

Only authenticated users can:

- Trigger investigation
- View history
- See diagnosis

Keep authentication minimal and clean.

## Cluster Selection

The dashboard should display Kubernetes clusters/contexts available through the user's local kubeconfig.

Expected flow:

```text
Local kubeconfig
      ↓
Discover contexts/clusters
      ↓
Display available clusters
      ↓
User selects a cluster
      ↓
Investigation targets selected context
```

Do not hard-code a single cluster. The selected cluster/context must be associated with the investigation request.

## Investigation Dashboard

Header:

```text
AI Kubernetes Agent
```

Main CTA:

```text
[ Investigate Cluster ]
```

Progress:

```text
✓ Checking Pods
✓ Reading Logs
✓ Analyzing Events
✓ Inspecting Deployments
✓ Checking Networking
✓ AI Reasoning
✓ Root Cause Found
```

Progress should update while the backend investigation runs using InsForge realtime capabilities.

## Root Cause Card

Display:

```text
Root Cause
Explanation
Suggested Fix
kubectl Command
Confidence Score
```

Example:

```text
Root Cause:
DATABASE_URL missing

Explanation:
Application failed during startup.

Suggested Fix:
Add missing environment variable.

Command:
kubectl edit deployment payment-service

Confidence:
92%
```

## Investigation History

Save investigations using InsForge.

Store:

- Timestamp
- Root Cause
- Namespace
- Confidence
- Status

Display recent investigations in a simple table/list.

## Frontend API Integration

Frontend calls:

```http
POST /investigate
```

Flow:

```text
User clicks button
        ↓
Frontend API call
        ↓
Backend investigation
        ↓
Realtime progress updates
        ↓
Diagnosis returned
        ↓
UI updates
```

Handle:

- Loading state
- API failures
- Empty response
- Timeouts

## Constraints

Do not change working backend investigation or AI reasoning logic.

Do not overengineer the UI. Do not add charts or complex state management.

Use InsForge only for:

- Authentication
- Realtime updates
- Investigation history

FastAPI remains the orchestrator.

## Expected Result

Users can log in, select a configured Kubernetes cluster, trigger an investigation, watch realtime progress, receive a diagnosis, and view previous investigations.
