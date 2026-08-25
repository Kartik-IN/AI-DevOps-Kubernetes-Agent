# Phase 05 — End-to-End Integration, Testing and Deployment Prompt

## Goal

Integrate the complete application, validate real Kubernetes failure scenarios, improve reliability, and prepare the system to feel like a real product.

Expected workflow:

```text
User clicks Investigate
        ↓
Frontend sends API request
        ↓
FastAPI orchestrates investigation
        ↓
Kubernetes evidence collected
        ↓
AI reasoning triggered
        ↓
Root cause generated
        ↓
Suggested fix returned
        ↓
History saved
        ↓
Realtime UI updated
        ↓
User sees diagnosis
```

## End-to-End Integration

Validate the complete frontend → FastAPI → Kubernetes → AI → InsForge flow.

Fix integration issues without unnecessarily changing working components.

## Reliability

Handle:

- `kubectl` failures
- Cluster unreachable
- Missing kubeconfig
- Invalid kubeconfig context
- Insufficient Kubernetes permissions
- OpenRouter failures
- API timeout
- Authentication issues
- No unhealthy resources found
- Empty investigation responses

Example UI message:

```text
Unable to connect to Kubernetes cluster.

Please verify:
- kubeconfig path
- selected cluster/context
- cluster access
- kubectl permissions
```

Do not expose raw stack traces or secrets to users.

## Loading and Empty States

When investigation starts:

```text
Investigating Kubernetes Cluster...
```

During investigation:

```text
✓ Checking Pods
✓ Reading Logs
✓ Analyzing Events
✓ Inspecting Deployments
✓ Checking Networking
✓ AI Reasoning
```

If no issue is found:

```text
No critical Kubernetes issues detected.
Cluster appears healthy.
```

## Real Kubernetes Failure Testing

Create intentional failure scenarios and verify the agent can diagnose them.

### Scenario 1 — CrashLoopBackOff

Example:

```text
Missing environment variable
```

Expected:

```text
Root Cause:
Missing env variable

Suggested Fix:
Add missing Secret/ConfigMap value
```

### Scenario 2 — ImagePullBackOff

Example:

```text
Wrong image tag
```

Expected:

```text
Root Cause:
Invalid image tag

Fix:
Update deployment image
```

### Scenario 3 — OOMKilled

Example:

```text
Low memory limits
```

Expected:

```text
Root Cause:
Container exceeded memory limit

Fix:
Increase memory requests/limits
```

### Scenario 4 — Service Selector Mismatch

Example:

```text
Wrong labels
```

Expected:

```text
Root Cause:
Service selector does not match pod labels

Fix:
Update service selector
```

## Multi-Cluster Testing

Validate that the application can discover the Kubernetes contexts/clusters represented in the local kubeconfig and that a user can select a target cluster before investigation.

For every selected context, verify that the investigation commands are executed against the selected cluster rather than an unintended default context.

## Constraints

Keep the implementation beginner friendly and avoid unnecessary complexity.

Do not break working functionality. Improve reliability incrementally.

## Expected Result

The final product should allow an authenticated user to select a configured Kubernetes cluster, investigate real failures, receive AI-generated diagnosis and remediation guidance, observe progress, and review investigation history.
