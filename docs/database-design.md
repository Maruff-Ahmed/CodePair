# CodePair --- Database Design

## 1. Database conventions

-   PostgreSQL is the relational database.
-   Use UUID primary keys for core entities.
-   Use UTC timestamps (`TIMESTAMPTZ`) for persisted times.
-   Use SQLAlchemy 2.0 models and Alembic migrations.
-   Add indexes to foreign keys and frequently queried fields.
-   Enforce critical invariants in the database where possible and in
    transactional backend logic.
-   Store password hashes only, never plaintext passwords.

## 2. Proposed tables

### `users`

-   `id` UUID primary key
-   `username` unique, not null
-   `email` unique, not null
-   `password_hash` not null
-   `display_name` nullable
-   `created_at`, `updated_at`

### `interview_rooms`

-   `id` UUID primary key
-   `room_code` unique, not null
-   `invitation_token_hash` unique, not null
-   `created_by` UUID foreign key to users
-   `status`: `waiting`, `active`, `closed`, `expired`
-   `expires_at`
-   `created_at`, `updated_at`

### `room_participants`

-   `id` UUID primary key
-   `room_id` UUID foreign key to interview_rooms
-   `user_id` UUID foreign key to users
-   `role`: `interviewer` or `candidate`
-   `joined_at`
-   `left_at` nullable

A participant may have multiple historical membership records if
reconnect/rejoin history is needed. Define a clear active-membership
uniqueness rule and enforce room capacity transactionally.

### `questions`

-   `id` UUID primary key
-   `title`
-   `description`
-   `difficulty`: `easy`, `medium`, `hard`
-   `constraints` text or structured JSON
-   `starter_code` JSON by language, nullable
-   `created_at`, `updated_at`

### `test_cases`

-   `id` UUID primary key
-   `question_id` UUID foreign key to questions
-   `input_data`
-   `expected_output`
-   `is_hidden` boolean
-   `display_order`
-   `created_at`

Hidden test cases must only be retrieved by trusted backend execution
logic, never by candidate-facing question endpoints.

### `interview_sessions`

-   `id` UUID primary key
-   `room_id` UUID foreign key to interview_rooms
-   `question_id` UUID foreign key to questions
-   `status`: `in_progress`, `completed`, `cancelled`
-   `started_at`, `ended_at`
-   `final_code` text
-   `language`
-   `duration_seconds`
-   `created_at`

For the initial product, one room produces at most one interview
session. Enforce this with a unique constraint on `room_id` if that
remains the chosen lifecycle.

### `evaluations`

-   `id` UUID primary key
-   `session_id` UUID unique foreign key to interview_sessions
-   `interviewer_id` UUID foreign key to users
-   `problem_solving_score` integer
-   `code_quality_score` integer
-   `communication_score` integer
-   `comments` text
-   `created_at`, `updated_at`

Define score range (recommended 1--5) with database check constraints
and matching API validation.

### `execution_records`

-   `id` UUID primary key
-   `session_id` UUID foreign key to interview_sessions
-   `language`
-   `judge0_submission_id` nullable
-   `status`
-   `execution_time` nullable
-   `memory_used` nullable
-   `created_at`

Avoid storing unnecessary source code or sensitive execution data in
this table. Final submitted code belongs to the session; temporary
execution payloads should have a retention policy.

### `ai_reports`

-   `id` UUID primary key
-   `session_id` UUID unique foreign key to interview_sessions
-   `strengths` JSONB
-   `weaknesses` JSONB
-   `recommendations` JSONB
-   `model_name` nullable
-   `generated_at`

Store structured, validated output. Treat AI-generated content as
advisory, not as an authoritative hiring decision.

### `code_snapshots` (later enhancement)

-   `id` UUID primary key
-   `session_id` UUID foreign key to interview_sessions
-   `code` text
-   `revision` bigint
-   `created_at`

Snapshots are optional for the MVP. A later replay feature may need
event history rather than snapshots alone.

## 3. Relationships

-   One user can create many interview rooms.
-   Users can participate in many rooms through room_participants.
-   A room has at most two active participants.
-   A room can produce one interview session in the initial design.
-   A question has many test cases.
-   A question can be used in many interview sessions.
-   A session has zero or one evaluation.
-   A session has many execution records.
-   A session has zero or one AI report.
-   A session can have many code snapshots.

## 4. Important constraints and indexes

Recommended constraints: - Unique `users.email` and `users.username`. -
Unique `interview_rooms.room_code`. - Unique
`interview_rooms.invitation_token_hash`. - Check allowed values for
room/session status and participant role. - Check score ranges. - Unique
`evaluations.session_id` for one evaluation per session. - Optional
unique `interview_sessions.room_id` for one session per room.

Recommended indexes: - `room_participants(room_id, left_at)` -
`room_participants(user_id)` -
`interview_rooms(created_by, created_at)` -
`interview_sessions(room_id)` -
`interview_sessions(status, started_at)` -
`execution_records(session_id, created_at)`

Room capacity cannot be safely guaranteed by a simple count check alone
under concurrent joins. Implement a transaction/locking strategy around
room admission and test simultaneous join attempts.

## 5. Design decisions to confirm during implementation

-   Whether room creator can leave and later rejoin as interviewer.
-   Whether a room can be reused for multiple sessions.
-   Whether evaluation scores are 1--5 or 1--10.
-   How long invitation links remain valid.
-   Retention period for code, execution records, and AI reports.
-   Whether AI reports are visible immediately or require interviewer
    approval.
