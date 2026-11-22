# CodePair

**Real-Time Collaborative Mock Interview Platform**

CodePair is a full-stack web application where an interviewer and a
candidate share a live coding room, solve programming problems
collaboratively, execute code in a sandbox, and review structured
interview feedback.

## Technology stack

-   Frontend: React + TypeScript
-   Backend: Python + FastAPI
-   Database: PostgreSQL
-   ORM and migrations: SQLAlchemy 2.0 + Alembic
-   Editor: Monaco Editor
-   Real-time communication: WebSockets
-   Code execution: Judge0
-   AI feedback: LLM API (planned)
-   Deployment: Docker

## Architecture

CodePair starts as a modular monolith: one FastAPI application owns
authentication, rooms, interview sessions, questions, evaluations,
analytics, and WebSocket coordination. Code execution is delegated to
Judge0 rather than running untrusted code inside the application server.

## Documentation

-   [Requirements](docs/requirements.md)
-   [Architecture](docs/architecture.md)
-   [Database design](docs/database-design.md)
-   [API specification](docs/api-specification.md)
-   [Development roadmap](docs/development-roadmap.md)

## Initial product boundary

The first release targets two authenticated participants per room. It
prioritizes a reliable shared editor, room lifecycle, timer, sandboxed
code execution, evaluation, and session history. Yjs collaboration,
session replay, whiteboard, and advanced analytics are later
enhancements.

## Development status

Phase 1 conceptual planning is complete. Phase 2 will establish the
repository, frontend, backend, database connection, migrations,
environment configuration, and health checks.
