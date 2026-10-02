# AgentFlow

AgentFlow is an AI-powered workflow system for extracting, processing, and summarizing information from web sources.

## Status

🚧 Work in Progress

## Architecture

- Frontend: Vue.js
- API: Express.js
- Agent Service: Python + LangGraph
- Database: PostgreSQL
- Queue / Cache: Redis
- Automation: n8n
- CI/CD: GitHub Actions
- Containerization: Docker

## System

Vue.js
  ↓
Express.js API
  ↓
Redis
  ↓
Python Agent Service
  ↓
LangGraph
  ↓
LLM + Tools
  ↓
PostgreSQL
