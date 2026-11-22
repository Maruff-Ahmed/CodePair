# CodePair --- System Architecture

## 1. Architectural style

Start with a **modular monolith** backed by PostgreSQL, plus an external
code execution provider. A single FastAPI application owns the product's
business domains. This avoids premature microservice complexity while
keeping modules separable.

## 2. Technology choices

  -----------------------------------------------------------------------
  Layer                   Technology              Responsibility
  ----------------------- ----------------------- -----------------------
  Frontend                React + TypeScript      Application UI

  Editor                  Monaco Editor           Coding experience

  Backend                 FastAPI                 REST APIs and business
                                                  logic

  Real-time               FastAPI WebSockets      Room events and editor
                                                  synchronization

  Database                PostgreSQL              Persistent application
                                                  data

  ORM                     SQLAlchemy 2.0          Database access

  Migrations              Alembic                 Versioned schema
                                                  changes

  Execution               Judge0                  Sandboxed code
                                                  execution

  AI                      External LLM API        Post-session feedback
                                                  summary

  Deployment              Docker                  Reproducible
                                                  environments
  -----------------------------------------------------------------------

## 3. Logical components

### Frontend

-   Authentication and profile pages
-   Dashboard and session history
-   Room creation and invitation flow
-   Interview room UI
-   Monaco editor and WebSocket client
-   Question panel, timer, participants, execution output
-   Evaluation and feedback views

### FastAPI backend modules

-   `api`: versioned REST routes
-   `core`: configuration, security, logging
-   `models`: SQLAlchemy models
-   `schemas`: request/response validation
-   `repositories`: database query operations
-   `services`: business use cases
-   `websockets`: connection manager and event handling
-   `integrations`: Judge0 and LLM clients

### External services

Judge0 receives code submissions and returns execution results. The LLM
receives only the minimum session information needed to generate a
report.

## 4. REST and WebSocket responsibilities

REST is used for durable request/response operations: - Register, login,
refresh token, profile - Create/join/leave rooms - Fetch questions and
session history - Submit code execution requests - Submit evaluations
and request reports

WebSockets are used for live room events: - Participant presence -
Editor changes - Cursor positions - Timer state updates - Interview
lifecycle notifications

## 5. High-level flow

1.  User authenticates through REST.
2.  Interviewer creates a room; backend persists it and returns an
    invitation token/link.
3.  Candidate joins after authorization and capacity checks.
4.  Each browser opens an authenticated WebSocket connection.
5.  The server validates room membership and broadcasts room events.
6.  Editor changes carry a revision/sequence number. The server
    maintains the authoritative room state and rejects or reconciles
    stale updates according to the initial synchronization policy.
7.  Code execution is submitted to Judge0 through the backend.
    Credentials and hidden test cases remain server-side.
8.  At interview completion, the backend persists final code and session
    metadata.
9.  Interviewer submits evaluation; AI report generation can use the
    evaluation, question context, execution outcomes, and explicitly
    selected notes.

## 6. Initial synchronization design

Start with a server-authoritative, revision-based synchronization
protocol. Each accepted editor update increments a room revision.
Clients ignore stale updates and request a current snapshot after
reconnecting if their revision is behind.

This is an MVP synchronization strategy, not a full conflict-free
collaborative editing algorithm. Simultaneous edits can still require
conflict handling. Evaluate Yjs/CRDT integration after the basic
two-client flow is stable.

## 7. WebSocket event envelope

Use a consistent JSON envelope, for example:

``` json
{
  "event": "editor:change",
  "room_id": "room-uuid",
  "client_message_id": "client-generated-id",
  "revision": 12,
  "payload": {
    "code": "print('hello')"
  }
}
```

The server must derive the sender identity from the authenticated
connection, not trust a client-supplied user ID. Validate event names,
payload schemas, message sizes, membership, and permissions.

Initial event names: - `room:join` - `room:leave` - `editor:change` -
`editor:update` - `editor:snapshot` - `cursor:update` -
`participant:joined` - `participant:left` - `timer:start` -
`timer:sync` - `interview:end` - `error`

## 8. Reliability and scaling

-   Track connection state separately from durable participant
    membership.
-   Use heartbeat/ping-pong and reconnect behavior.
-   On reconnect, revalidate authorization and send the authoritative
    room snapshot.
-   Persist important lifecycle changes and final session data.
-   Begin with one FastAPI process. In-memory connection managers do not
    synchronize across multiple workers.
-   Before multi-instance deployment, introduce a shared pub/sub layer
    such as Redis and revisit room-state ownership.

## 9. Security boundaries

-   Hash passwords using Argon2id.
-   Use short-lived access tokens and a secure refresh-token strategy.
-   Authenticate WebSockets and authorize every room action.
-   Use unguessable invitation tokens; store a hash of the token where
    practical.
-   Apply request validation, rate limits, CORS restrictions, and
    message-size limits.
-   Keep Judge0 and LLM credentials server-side.
-   Do not expose hidden test cases to candidate-facing endpoints.
-   Never run submitted code inside the FastAPI process.
-   Treat code, execution output, interviewer notes, and LLM output as
    untrusted content.
-   Do not render untrusted output as raw HTML.

## 10. Deployment shape

For local development, use Docker Compose for PostgreSQL and optionally
the application services. The frontend and backend can run independently
during development. Production deployment choices will be confirmed
after the MVP works and provider constraints are checked.
