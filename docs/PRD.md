# AgentFlow

## Product Requirements Document

**Version:** 1.0
**Status:** Draft
**Project Type:** Portfolio / Engineering Project
**Difficulty:** Medium
**Primary Objective:** Demonstrate full-stack AI application engineering, asynchronous job processing, agent orchestration, service integration, and production-oriented development practices.

---

# 1. Product Overview

## 1.1 Product Name

**AgentFlow**

## 1.2 Product Description

AgentFlow is an AI-powered workflow system that accepts a web URL, retrieves and processes its content, performs structured analysis through an AI agent workflow, and presents the resulting information through a web dashboard.

The system separates application logic, AI orchestration, asynchronous processing, persistence, and workflow automation into distinct components.

The primary workflow is:

```text
User
 │
 ▼
Vue.js Frontend
 │
 ▼
Express.js API
 │
 ▼
Redis Job Queue
 │
 ▼
Python Agent Service
 │
 ▼
LangGraph
 ├── Retrieval
 ├── Content Processing
 ├── Analysis
 └── Validation
 │
 ▼
PostgreSQL
 │
 ▼
Express.js API
 │
 ▼
Vue.js Dashboard
```

n8n provides external workflow automation and integration capabilities around the core application.

---

# 2. Problem Statement

AI applications frequently demonstrate only a single model call:

```text
User → Prompt → LLM → Response
```

This does not adequately demonstrate the engineering required to build an AI-powered application.

Real-world AI systems commonly require:

* asynchronous processing,
* persistent data,
* external API integration,
* task queues,
* failure handling,
* workflow orchestration,
* structured model outputs,
* observability,
* authentication,
* API design,
* frontend state management,
* deployment automation.

AgentFlow addresses this by implementing an AI workflow as an actual software system rather than as a standalone chatbot.

---

# 3. Product Goals

## 3.1 Primary Goals

AgentFlow must demonstrate the ability to:

1. Build a REST API using Express.js.
2. Build a modern frontend using Vue.js.
3. Design and operate a PostgreSQL data model.
4. Implement asynchronous jobs using Redis.
5. Build an AI workflow using LangGraph.
6. Integrate an LLM through a provider-independent interface.
7. Integrate external services through APIs.
8. Handle long-running jobs without blocking HTTP requests.
9. Persist workflow state and results.
10. Implement workflow automation using n8n.
11. Containerize the application with Docker.
12. Implement automated testing and CI using GitHub Actions.
13. Deploy a functional production-like version of the system.

## 3.2 Secondary Goals

The project should demonstrate:

* clean service boundaries,
* typed data contracts,
* error handling,
* retry mechanisms,
* idempotency,
* structured logging,
* API documentation,
* configuration management,
* secure secret handling,
* maintainable repository structure.

---

# 4. Non-Goals

The following are explicitly outside the initial scope:

* Training an LLM from scratch.
* Fine-tuning a large language model.
* Building a general-purpose agent platform.
* Building a general-purpose workflow editor.
* Implementing Kubernetes.
* Implementing a vector database.
* Building a production-scale web crawler.
* Supporting dozens of external integrations.
* Building a commercial multi-tenant SaaS platform.
* Implementing autonomous agents capable of unrestricted computer interaction.

These can be future extensions but should not be required for the MVP.

---

# 5. Target Users

## 5.1 Primary User

A developer, researcher, or knowledge worker who wants to analyze information from web sources using an AI-assisted workflow.

## 5.2 Example Use Case

A user wants to analyze an article.

They submit:

```text
https://example.com/article
```

AgentFlow:

1. Creates a processing job.
2. Retrieves the webpage.
3. Extracts relevant content.
4. Cleans the content.
5. Analyzes the content using an AI workflow.
6. Validates the generated result.
7. Stores the result.
8. Displays the result in the dashboard.

---

# 6. Core User Stories

## 6.1 Submit URL

**As a user**, I want to submit a URL so that AgentFlow can analyze its contents.

### Acceptance Criteria

* User can enter a URL.
* System validates the URL.
* Invalid URLs are rejected.
* Valid URLs create a job.
* API returns a unique job ID.
* Job initially has `queued` status.

---

## 6.2 Track Processing

**As a user**, I want to see the status of my job so that I know whether processing is complete.

Possible states:

```text
queued
processing
completed
failed
cancelled
```

### Acceptance Criteria

* Dashboard displays current status.
* Status updates without requiring the user to resubmit the URL.
* Failed jobs display an error message.
* Completed jobs provide access to the generated result.

---

## 6.3 View Workflow

**As a user**, I want to see which processing stages have completed.

Example:

```text
✓ URL Validation
✓ Content Retrieval
✓ Content Cleaning
● AI Analysis
○ Validation
○ Persistence
```

### Acceptance Criteria

* Workflow stages are visible.
* Current stage is distinguishable.
* Completed stages are recorded.
* Failed stages display an error state.

---

## 6.4 View Analysis

**As a user**, I want to view the AI-generated analysis in a structured format.

Example:

```text
Title
Summary
Key Points
Entities
Topics
Confidence / Validation Status
Source URL
```

The exact schema may evolve during implementation.

---

## 6.5 Review Source Content

**As a user**, I want to inspect the source information used by the system.

### Acceptance Criteria

* Original URL is displayed.
* Extracted content metadata is displayed.
* Generated analysis is associated with the source.

---

## 6.6 Retry Failed Jobs

**As a user**, I want to retry a failed job without submitting the URL again.

### Acceptance Criteria

* Failed jobs expose a retry action.
* A new processing attempt is created.
* Previous attempt information remains available.
* Retry does not create duplicate persistent results.

---

# 7. Functional Requirements

# 7.1 Frontend

**Technology:** Vue.js + TypeScript

The frontend must provide:

### Dashboard

Displays:

* total jobs,
* queued jobs,
* processing jobs,
* completed jobs,
* failed jobs.

### Job Creation

Provides:

* URL input,
* validation feedback,
* submission button.

### Job List

Displays:

* job ID,
* source URL,
* status,
* creation time,
* completion time.

### Job Detail

Displays:

* source information,
* workflow status,
* processing stages,
* generated analysis,
* errors,
* retry controls.

### Result Viewer

Displays structured AI output rather than only raw LLM text.

---

# 7.2 Express.js API

**Technology:** Node.js + Express.js

The API acts as the application's primary gateway.

Initial endpoints:

```text
POST   /api/jobs
GET    /api/jobs
GET    /api/jobs/:id
GET    /api/jobs/:id/result
POST   /api/jobs/:id/retry
DELETE /api/jobs/:id
GET    /api/health
```

## Job Creation

```http
POST /api/jobs
```

Request:

```json
{
  "url": "https://example.com/article"
}
```

Response:

```json
{
  "id": "job_123",
  "status": "queued"
}
```

---

# 7.3 Python Agent Service

**Technology:** Python + FastAPI + LangGraph

The Python service is responsible for AI workflow execution.

It must not be responsible for frontend concerns or primary application authentication.

Initial endpoint:

```text
POST /execute
```

Example request:

```json
{
  "job_id": "job_123",
  "url": "https://example.com/article"
}
```

The service executes the LangGraph workflow and reports the resulting state.

---

# 8. LangGraph Workflow

The initial workflow should consist of explicit nodes.

```text
START
  │
  ▼
Validate Input
  │
  ▼
Retrieve Content
  │
  ▼
Clean Content
  │
  ▼
Analyze Content
  │
  ▼
Validate Output
  │
  ├── Invalid → Retry / Repair
  │
  └── Valid
       │
       ▼
Persist Result
       │
       ▼
      END
```

## 8.1 Input Validation

Validates:

* URL format,
* supported protocol,
* basic request constraints.

## 8.2 Content Retrieval

Retrieves the webpage through an HTTP client.

Potential implementation:

* Python HTTP client,
* readability/content extraction library.

Selenium may be introduced later for pages requiring browser rendering, but it should not be required for the initial MVP.

## 8.3 Content Cleaning

Removes:

* navigation,
* advertisements,
* irrelevant markup,
* duplicated content.

Produces normalized text.

## 8.4 AI Analysis

The LLM generates structured information from the cleaned content.

The output should conform to a defined Pydantic schema.

Example:

```json
{
  "title": "...",
  "summary": "...",
  "key_points": [],
  "topics": [],
  "entities": []
}
```

## 8.5 Validation

The system validates:

* schema compliance,
* required fields,
* basic consistency,
* source association.

Invalid output should trigger a controlled repair/retry path.

---

# 9. Redis

Redis is responsible for asynchronous job processing and transient application state.

The architecture should prevent long-running AI processing from blocking the Express.js HTTP process.

```text
HTTP Request
     │
     ▼
Express
     │
     ▼
Redis Queue
     │
     ▼
Worker
     │
     ▼
Python Agent Service
```

Redis may additionally provide:

* job state,
* temporary cache,
* distributed locking where required.

Redis should not be treated as the system's permanent source of truth.

---

# 10. PostgreSQL

PostgreSQL is the persistent system of record.

Initial entities:

```text
users
jobs
job_attempts
workflow_steps
sources
results
```

Possible relationship:

```text
User
 │
 └── Jobs
      │
      ├── Attempts
      │
      ├── Workflow Steps
      │
      └── Result
           │
           └── Source
```

The schema should support multiple processing attempts for the same job.

---

# 11. n8n Integration

n8n is used for external workflow automation rather than replacing the application's internal job system.

Initial automation:

```text
n8n
 │
 ├── Scheduled trigger
 │
 ├── HTTP Request
 │
 └── AgentFlow API
```

Example workflows:

### Daily Health Check

```text
Schedule
 ↓
GET /api/health
 ↓
Evaluate response
 ↓
Notification
```

### Job Completion Notification

```text
AgentFlow Webhook
 ↓
n8n
 ↓
Notification Service
```

Potential integrations:

* Discord,
* Slack,
* email,
* generic HTTP webhook.

Only one external notification integration is required for MVP.

---

# 12. API Integration

The system should demonstrate at least:

* REST API,
* webhook integration,
* service-to-service HTTP communication.

Optional future extensions:

* OAuth 2.0,
* GraphQL,
* external search APIs.

OAuth and GraphQL are not required for the first release.

---

# 13. Authentication

Authentication should not be implemented in the first functional milestone.

For the MVP:

```text
Single-user development environment
```

A later milestone can introduce:

```text
User
 ↓
Authentication
 ↓
Authorization
 ↓
Jobs
```

Potential implementation:

* JWT,
* session-based authentication,
* OAuth provider.

The authentication architecture should avoid coupling the AI service to user authentication.

---

# 14. Observability

The system must provide sufficient information to diagnose failed jobs.

Minimum requirements:

### Application Logs

Each request/job should include:

```text
timestamp
service
job_id
event
level
```

Example:

```text
INFO job_id=job_123 event=workflow_started
INFO job_id=job_123 event=content_retrieved
ERROR job_id=job_123 event=llm_failed
```

### Workflow Tracking

Each job should record:

```text
step
status
started_at
completed_at
error
```

---

# 15. Error Handling

The system must distinguish between:

### Client Errors

Examples:

```text
Invalid URL
Malformed request
Unsupported URL
```

### Processing Errors

Examples:

```text
Website unavailable
Timeout
LLM failure
Malformed model output
```

### Infrastructure Errors

Examples:

```text
Redis unavailable
PostgreSQL unavailable
Agent service unavailable
```

Errors should not expose secrets, API keys, stack traces, or internal infrastructure details to users.

---

# 16. Retry Strategy

Retryable failures should use bounded retries.

Example:

```text
Attempt 1
   ↓
Failure
   ↓
Wait
   ↓
Attempt 2
   ↓
Failure
   ↓
Wait
   ↓
Attempt 3
   ↓
Failed
```

The retry mechanism should distinguish transient failures from permanent failures.

---

# 17. Idempotency

Job processing should be designed to avoid duplicate results when requests are repeated.

Example:

```text
job_123
   │
   ├── attempt_1
   ├── attempt_2
   └── result
```

The same job should not accidentally generate multiple conflicting final results because of a worker restart or duplicate queue delivery.

---

# 18. Security Requirements

Minimum requirements:

* Secrets stored in environment variables.
* `.env` excluded from Git.
* Input validation.
* URL validation.
* Request size limits.
* API rate limiting where appropriate.
* CORS configuration.
* HTTP security headers.
* No API keys in frontend code.
* No sensitive information in logs.

Potential SSRF protection should be considered because the application accepts user-provided URLs.

The system should prevent requests to inappropriate internal/private network destinations in a production deployment.

---

# 19. Docker

Each major runtime component should have a container:

```text
frontend
backend
agent
worker
```

Infrastructure:

```text
postgres
redis
```

Local development should be reproducible through Docker Compose.

Target:

```bash
docker compose up
```

should start the required development environment.

---

# 20. CI/CD

**Platform:** GitHub Actions

The CI pipeline should execute on:

* pull requests,
* pushes to `main`.

Pipeline:

```text
Checkout
   │
   ├── Backend lint
   ├── Backend tests
   │
   ├── Frontend lint
   ├── Frontend tests
   │
   ├── Python lint
   └── Python tests
```

Later:

```text
Tests
 ↓
Docker Build
 ↓
Container Registry
 ↓
Deployment
```

---

# 21. Deployment

The initial deployment should prioritize simplicity and free/low-cost infrastructure.

Target architecture:

```text
Frontend
   │
   ▼
Backend API
   │
   ├── PostgreSQL
   ├── Redis
   └── Agent Service
```

The exact cloud provider should be selected based on available free tiers and current project requirements.

Cloud deployment is a later milestone and should not block local development.

---

# 22. Non-Functional Requirements

## Performance

The HTTP API should return job creation responses immediately rather than waiting for AI processing.

Target behavior:

```text
POST /jobs
    ↓
create job
    ↓
return job ID
```

rather than:

```text
POST /jobs
    ↓
retrieve webpage
    ↓
run LLM
    ↓
validate
    ↓
return response
```

## Reliability

A temporary LLM or website failure should not crash the entire API service.

## Maintainability

Services should have clearly defined responsibilities.

## Extensibility

The architecture should allow additional agent nodes and external integrations without rewriting the core API.

## Portability

The application should run locally using Docker Compose.

---

# 23. MVP Scope

The MVP is complete when a user can:

```text
1. Open the Vue application.
2. Submit a URL.
3. Receive a job ID.
4. See the job enter "queued".
5. See the job become "processing".
6. Watch workflow progress.
7. See the webpage processed.
8. Receive structured AI analysis.
9. View the final result.
10. Retry a failed job.
```

Required technical components:

```text
Vue.js
Express.js
Python
FastAPI
LangGraph
PostgreSQL
Redis
Docker
GitHub Actions
```

n8n should have at least one working integration before the MVP is considered complete.

---

# 24. Post-MVP Features

Potential extensions:

### Phase 2

* Authentication
* Multiple users
* Job history
* Advanced filtering
* Webhook subscriptions
* Discord/Slack notifications

### Phase 3

* Multiple LLM providers
* Provider fallback
* Agent evaluation
* Prompt versioning
* Token/cost tracking
* Model comparison

### Phase 4

* RAG
* Vector database
* Semantic search
* Document ingestion
* Persistent knowledge base

### Phase 5

* Distributed workers
* Horizontal scaling
* Kubernetes
* MLflow
* Advanced observability

These features belong to later projects or later versions rather than the initial Medium-project MVP.

---

# 25. Technology Stack

| Layer               | Technology                            |
| ------------------- | ------------------------------------- |
| Frontend            | Vue.js + TypeScript                   |
| Frontend State      | Pinia                                 |
| Frontend Routing    | Vue Router                            |
| Backend             | Node.js + Express.js                  |
| AI Service          | Python + FastAPI                      |
| Agent Framework     | LangGraph                             |
| LLM Integration     | Provider abstraction                  |
| Database            | PostgreSQL                            |
| Queue / Cache       | Redis                                 |
| Automation          | n8n                                   |
| Containerization    | Docker                                |
| Local Orchestration | Docker Compose                        |
| CI/CD               | GitHub Actions                        |
| API Style           | REST                                  |
| Data Validation     | Pydantic / schema validation          |
| Testing             | Vitest + Node test framework + Pytest |

---

# 26. Repository Architecture

```text
agentflow/
│
├── backend/
│   ├── src/
│   │   ├── config/
│   │   ├── controllers/
│   │   ├── middleware/
│   │   ├── routes/
│   │   ├── services/
│   │   ├── workers/
│   │   └── app.js
│   └── tests/
│
├── frontend/
│   └── src/
│       ├── components/
│       ├── views/
│       ├── services/
│       ├── stores/
│       └── router/
│
├── agent/
│   ├── src/
│   │   ├── agents/
│   │   ├── graph/
│   │   ├── tools/
│   │   └── main.py
│   └── tests/
│
├── n8n/
│   └── workflows/
│
├── docs/
│   ├── architecture.md
│   └── api.md
│
├── .github/
│   └── workflows/
│
├── docker-compose.yml
├── .gitignore
├── README.md
└── LICENSE
```

---

# 27. Success Criteria

The project is successful when the repository demonstrates all of the following:

### Software Engineering

* [ ] REST API
* [ ] Modular backend
* [ ] PostgreSQL schema
* [ ] Redis-based asynchronous processing
* [ ] Error handling
* [ ] Retry mechanism
* [ ] Idempotent job processing
* [ ] Automated tests

### AI Engineering

* [ ] LangGraph workflow
* [ ] Multiple workflow nodes
* [ ] Structured LLM output
* [ ] Tool integration
* [ ] Validation/repair path
* [ ] Provider abstraction

### Full Stack

* [ ] Vue.js frontend
* [ ] API integration
* [ ] Job dashboard
* [ ] Workflow visualization
* [ ] Result viewer

### Integration

* [ ] n8n workflow
* [ ] Webhook integration
* [ ] Service-to-service API communication

### DevOps

* [ ] Docker
* [ ] Docker Compose
* [ ] GitHub Actions
* [ ] Automated tests
* [ ] Deployment

---

# 28. Portfolio Demonstration

The project should be presented as an engineering system rather than merely an "AI summarizer."

The portfolio should emphasize:

```text
AI Application
       +
Backend Engineering
       +
Distributed Job Processing
       +
Agent Orchestration
       +
Database Engineering
       +
Workflow Automation
       +
DevOps
```

The most important architectural demonstration is:

```text
                 ┌──────────────┐
                 │   Vue.js     │
                 └──────┬───────┘
                        │
                       REST
                        │
                 ┌──────▼───────┐
                 │  Express.js  │
                 └──┬────┬───┬──┘
                    │    │   │
              ┌─────┘    │   └────────┐
              ▼           ▼            ▼
        PostgreSQL      Redis        n8n
                          │
                          ▼
                   Python Worker
                          │
                          ▼
                      LangGraph
                     /    |    \
                    ▼     ▼     ▼
                Retrieve Analyze Validate
                          │
                          ▼
                         LLM
```
