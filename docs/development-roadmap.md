# CodePair --- Development Roadmap

Target: approximately 6--8 weeks for a focused solo development effort.
The schedule is an estimate and should change based on implementation
and testing results.

## Phase 1 --- Planning and system design

**Status: Conceptual planning complete**

-   Product vision and scope
-   Functional and non-functional requirements
-   Technology selection
-   Architecture
-   Database design
-   API inventory
-   Security boundaries
-   MVP acceptance criteria

Remaining Phase 1 handoff: review and commit the documentation, then
confirm any unresolved schema decisions during implementation.

## Phase 2 --- Project foundation

**Goal:** a reproducible local development environment.

-   Create repository and initial documentation.
-   Set up React + TypeScript frontend.
-   Set up FastAPI backend.
-   Configure PostgreSQL.
-   Configure SQLAlchemy 2.0 and Alembic.
-   Add environment-based settings and `.env.example`.
-   Add backend and frontend health checks.
-   Add basic automated checks and setup instructions.

Exit criteria: frontend loads, backend health endpoint responds, and
backend can connect to PostgreSQL.

## Phase 3 --- Backend foundation

-   User model and migrations.
-   Registration, login, token lifecycle.
-   Password hashing and authorization dependencies.
-   User profile endpoints.
-   Room and participant models.
-   Create/join/leave room APIs.
-   Tests for authentication and room capacity.

Exit criteria: two authorized users can create and join a room through
the API.

## Phase 4 --- Frontend foundation

-   Routing and application layout.
-   Registration and login screens.
-   Authenticated dashboard.
-   Room creation and invitation flow.
-   Join room flow.
-   Interview room layout and loading/error states.

Exit criteria: users can complete the room workflow in the browser.

## Phase 5 --- Real-time collaboration

-   Authenticated WebSocket endpoint.
-   Room connection manager.
-   Presence and disconnect handling.
-   Monaco integration.
-   Editor synchronization with revision numbers.
-   Reconnect and snapshot recovery.
-   Timer synchronization.
-   Two-browser and network interruption tests.

Exit criteria: two browser sessions can reliably collaborate in one
room.

## Phase 6 --- Coding environment

-   Question bank and seed questions.
-   Language selection and starter code.
-   Judge0 integration.
-   Execution status and result display.
-   Hidden test case handling.
-   Execution limits, validation, and failure handling.

Exit criteria: code can be submitted and results returned without
executing untrusted code in the application server.

## Phase 7 --- Interview workflow

-   Start/end session lifecycle.
-   Persist final code and language.
-   Interviewer scorecard.
-   Feedback visibility and permissions.
-   Session history and detail page.

Exit criteria: completed sessions and evaluations can be reviewed by
authorized participants.

## Phase 8 --- AI and analytics

-   Define structured AI input/output schemas.
-   Generate strengths, weaknesses, and recommendations.
-   Validate and persist LLM output.
-   Add session analytics and progress views.
-   Handle provider errors and regeneration.

Exit criteria: an authorized user can request and view a structured AI
report.

## Phase 9 --- Testing, security, and deployment

-   Backend unit and integration tests.
-   Frontend component and workflow tests.
-   Concurrent WebSocket tests.
-   Authorization and room isolation tests.
-   Code execution abuse checks.
-   Docker build and deployment configuration.
-   Production environment variables and CORS.
-   Logging, monitoring basics, README, screenshots, demo walkthrough.

Exit criteria: deployed application passes the core MVP acceptance
criteria.

## MVP acceptance checklist

-   [ ] Two authenticated users join one room.
-   [ ] Both see the same question.
-   [ ] Both can edit shared code.
-   [ ] Editor updates synchronize.
-   [ ] Presence is visible.
-   [ ] Timer state is synchronized.
-   [ ] Code runs through Judge0.
-   [ ] Interviewer submits an evaluation.
-   [ ] Candidate views feedback.
-   [ ] Session history is persisted.

## Later enhancements

-   Yjs/CRDT collaborative editing.
-   Live cursor selections and richer presence.
-   Session replay and event timeline.
-   Collaborative whiteboard.
-   Additional languages and question authoring tools.
-   Redis Pub/Sub for multi-instance WebSocket broadcasting.
-   More advanced analytics.

## Working agreement

For each implementation task: 1. Agree on the expected behavior and
acceptance criteria. 2. Give Antigravity a bounded implementation
prompt. 3. Inspect the diff and test output. 4. Run the relevant
application workflow. 5. Fix issues before moving forward. 6. Commit a
verified milestone.

Maintain `PROJECT_STATUS.md` with completed work, current work, known
issues, and the next task.
