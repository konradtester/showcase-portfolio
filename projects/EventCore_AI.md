# EventCore AI | Next-Gen Event Management SaaS with AI-Powered Intelligence

## 🚀 Business Overview

EventCore AI is a comprehensive SaaS platform designed to revolutionize event management and coordination. Built for scalability and real-time interaction, it bridges the gap between organizers and attendees through an intuitive interface and advanced AI capabilities. EventCore transforms static event data into dynamic, searchable knowledge bases, enabling automated attendee support and seamless incident management.

**Key Value Propositions:**
- **Scalable Infrastructure:** Designed to handle high-concurrency events with real-time updates via WebSocket mesh.
- **AI-Driven Support:** Built-in RAG system to handle complex attendee queries automatically.
- **Autonomous DevOps Agent:** LangGraph-based agent with Human-in-the-Loop safeguards — exposed via Discord bot for infrastructure management.
- **Secure by Design:** Role-based access control (RBAC) and data anonymization protocols ensure enterprise-grade security.

---

## 🛠️ System Architecture

EventCore AI uses a monorepo with a distributed, event-driven architecture:

- **Web Dashboard (React):** Organizer command center — event creation, attendee management, RAG monitoring.
- **Mobile App (Flutter):** Real-time event navigation, AI assistant chat, SOS/emergency features.
- **Backend (FastAPI):** REST + WebSocket API, PostgreSQL, Docker-hosted on VPS/Linux.
- **AI Agent (LangGraph + Discord):** DevOps assistant with safe/sensitive tool split and HITL approval flow.

---

## 🏗️ Engineering Challenges & Solutions

### 1. Real-Time State Synchronization at Scale

**Challenge:** Hundreds of mobile clients need instant updates (event status, SOS alerts) without overwhelming the database.

**Solution:** WebSocket mesh with a pub/sub broker. This decouples the API from live connections, allowing efficient broadcast using a "Room" pattern — each event is an isolated room. Sub-second synchronization across web and mobile clients.

---

### 2. Securing WebSockets with JWT

**Challenge:** WebSockets don't natively support standard HTTP headers for authentication in all environments.

**Solution:** Custom middleware that validates JWT tokens during the initial handshake and maintains session integrity through heartbeats. Tokens refresh periodically without interrupting the socket connection.

---

### 3. Mitigating LLM Hallucinations in RAG

**Challenge:** AI assistants might provide incorrect data about event schedules or safety protocols.

**Solution:** Strict self-correction loop using LangGraph. The agent verifies retrieved context against the query before generating a response. If no relevant context is found, the agent redirects the user to a human organizer — it never guesses. All responses are anchored to a source document fragment.

---

### 4. Human-in-the-Loop DevOps Agent

**Challenge:** An autonomous agent with access to destructive infrastructure operations (git pull, storage cleanup, Docker management) is a liability without proper safeguards.

**Solution:** Tools split into two categories at compile time:
- **Safe tools** (read-only): Docker logs, NGINX log analysis, server vitals, API tests — run autonomously.
- **Sensitive tools** (destructive): git pull, storage cleanup — require explicit human approval via Discord before execution.

The LangGraph graph compiles with `interrupt_before=["sensitive_tools"]` — a hard architectural constraint, not a runtime check. Exposed via Discord bot with a `tak`/`nie` approval flow. Every decision traced in LangSmith.

---

### 5. High-Performance Vector Search

**Challenge:** Scaling semantic search across thousands of event document segments.

**Solution:** Qdrant indexing with custom payload filtering — queries are isolated per event ID, preventing cross-event data leakage. Sub-100ms response times under load.

---

## 🛠️ Technology Stack

| Layer | Technologies |
|---|---|
| Backend | FastAPI · Python · PostgreSQL · WebSockets · SQLAlchemy · Docker · NGINX |
| Frontend | React · TypeScript · TailwindCSS |
| Mobile | Flutter · Dart |
| AI / Agentic | LangGraph · LangChain · Qdrant (VectorDB) · LangSmith · Gemini / OpenAI APIs |
| DevOps | Docker · VPS/Linux · GitLab CI · SSH-based multi-env management |

---

## 📊 Status

Near App Store / Google Play launch. Payments mocked. Built to solve a real problem — designed for public release.
