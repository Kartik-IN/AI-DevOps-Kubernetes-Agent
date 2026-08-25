# Phase 01 — Project Setup Prompt

## Context

We are building an **AI Kubernetes Troubleshooting Agent**.

Architecture:

```text
Frontend
    ↓
FastAPI Backend (Orchestrator)
    ↓
Kubernetes Investigation Layer
    ↓
AI Kubernetes Agent
    ↓
LLM Reasoning (OpenRouter via InsForge)
    ↓
Root Cause + Suggested Fix
    ↓
Frontend Diagnosis
```

This is an **on-demand troubleshooting system**. We are **not building a Kubernetes controller/operator**.

## Goal

Set up the project foundation:

- FastAPI backend
- Next.js frontend
- Docker setup
- Environment variables
- Basic folder structure
- Health endpoint

Do not implement Kubernetes logic or AI yet.

## Tech Stack

### Backend

- FastAPI
- Python 3.12+
- Uvicorn
- Pydantic
- Loguru
- HTTPX

### Frontend

- Next.js
- TypeScript
- Tailwind CSS
- Axios
- React Query

### Infrastructure

- Docker
- Docker Compose

## Project Structure

```text
ai-kubernetes-agent/
├── backend/
├── frontend/
├── docs/
├── prompts/
├── docker-compose.yml
└── README.md
```

Backend placeholders:

```text
api/
core/
kubernetes/
ai/
services/
models/
```

Frontend placeholders:

```text
components/
services/
hooks/
types/
```

Use placeholder implementations only.

## Backend Requirements

Create a FastAPI application with:

```text
GET /health
```

Expected response:

```json
{
  "status": "healthy",
  "service": "ai-kubernetes-agent"
}
```

Enable:

- CORS
- Logging
- Environment loading

## Frontend Requirements

Create a minimal homepage:

```text
AI Kubernetes Agent

Troubleshoot Kubernetes with AI

[ Investigate Cluster ]

System Status: Ready
```

Use simple professional styling.

## Environment Variables

Backend:

```env
OPENROUTER_API_KEY=
OPENROUTER_MODEL=
KUBECONFIG_PATH=
```

Frontend:

```env
NEXT_PUBLIC_API_BASE_URL=http://localhost:8000
```

## Docker Requirements

Create Dockerfiles for:

- Backend
- Frontend

Create Docker Compose with:

```text
backend → port 8000
frontend → port 3000
```

## Constraints

Do not implement:

- kubectl logic
- AI reasoning
- OpenRouter
- InsForge
- Authentication
- Realtime updates

Only set up the foundation. Keep the code beginner friendly and production-style. Do not break existing code in future implementations; extend the project incrementally.

## Expected Result

The application should run with:

```bash
docker compose up --build
```

Access:

```text
http://localhost:3000
http://localhost:8000/health
```
