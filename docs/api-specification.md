# CodePair --- API Specification (Initial)

Base path: `/api/v1`

This is an initial endpoint inventory. Request/response schemas, error
codes, pagination, and authentication details will be finalized as each
module is implemented.

## Authentication

  Method   Endpoint           Purpose
  -------- ------------------ --------------------------------
  POST     `/auth/register`   Register a user
  POST     `/auth/login`      Authenticate and issue tokens
  POST     `/auth/refresh`    Refresh access token
  POST     `/auth/logout`     Revoke refresh session/token
  GET      `/auth/me`         Get current authenticated user

## Users

  Method   Endpoint      Purpose
  -------- ------------- --------------------
  GET      `/users/me`   Get own profile
  PATCH    `/users/me`   Update own profile

## Rooms

  Method   Endpoint                   Purpose
  -------- -------------------------- ------------------
  POST     `/rooms`                   Create room
  GET      `/rooms/{room_id}`         Get room details
  POST     `/rooms/{room_id}/join`    Join room
  POST     `/rooms/{room_id}/leave`   Leave room
  POST     `/rooms/{room_id}/start`   Start interview
  POST     `/rooms/{room_id}/end`     End interview

Room join should use an invitation token or a carefully designed invite
URL flow. Do not expose internal room membership or private invitation
credentials in public room responses.

## Questions

  ----------------------------------------------------------------------------
  Method                  Endpoint                     Purpose
  ----------------------- ---------------------------- -----------------------
  GET                     `/questions`                 List questions

  GET                     `/questions/{question_id}`   Get candidate-visible
                                                       question details

  POST                    `/questions`                 Create a question
                                                       (authorized role only)

  PATCH                   `/questions/{question_id}`   Update a question
                                                       (authorized role only)
  ----------------------------------------------------------------------------

Candidate-visible question responses must exclude hidden test cases and
internal execution metadata.

## Code execution

  ------------------------------------------------------------------------------
  Method                  Endpoint                       Purpose
  ----------------------- ------------------------------ -----------------------
  POST                    `/executions`                  Submit code for
                                                         execution

  GET                     `/executions/{execution_id}`   Retrieve execution
                                                         status/result
  ------------------------------------------------------------------------------

The backend validates language, source size, room/session access, and
execution limits before calling Judge0. Do not trust client-supplied
Judge0 identifiers or hidden test case contents.

## Sessions and evaluation

  -------------------------------------------------------------------------------------
  Method                  Endpoint                              Purpose
  ----------------------- ------------------------------------- -----------------------
  GET                     `/sessions`                           List current user's
                                                                accessible sessions

  GET                     `/sessions/{session_id}`              Get session details

  POST                    `/sessions/{session_id}/evaluation`   Submit interviewer
                                                                evaluation

  GET                     `/sessions/{session_id}/evaluation`   Get permitted
                                                                evaluation

  POST                    `/sessions/{session_id}/report`       Request AI report
                                                                generation

  GET                     `/sessions/{session_id}/report`       Retrieve AI report
  -------------------------------------------------------------------------------------

All endpoints must enforce ownership, room membership, and role
permissions. Use pagination for list endpoints.

## WebSocket

Suggested connection path:

`/api/v1/ws/rooms/{room_id}`

Authenticate the connection using a secure mechanism appropriate to the
deployment. Avoid placing long-lived bearer tokens in query strings
because URLs may be logged. Validate room membership before accepting
the connection.

### Initial events

  -----------------------------------------------------------------------
  Event                   Direction               Purpose
  ----------------------- ----------------------- -----------------------
  `room:join`             Client → Server         Join room channel after
                                                  authorization

  `room:leave`            Client → Server         Leave room channel

  `editor:change`         Client → Server         Submit editor update

  `editor:update`         Server → Client         Broadcast accepted
                                                  update

  `editor:snapshot`       Server → Client         Send authoritative
                                                  editor state

  `cursor:update`         Client ↔ Server         Share cursor position

  `participant:joined`    Server → Client         Presence notification

  `participant:left`      Server → Client         Presence notification

  `timer:start`           Client → Server         Authorized timer start
                                                  request

  `timer:sync`            Server → Client         Authoritative timer
                                                  state

  `interview:end`         Client → Server         Authorized end request

  `error`                 Server → Client         Structured error
  -----------------------------------------------------------------------

### Event envelope

``` json
{
  "event": "editor:change",
  "client_message_id": "unique-client-message-id",
  "revision": 12,
  "payload": {
    "code": "print('hello')"
  }
}
```

The authenticated connection determines the sender. The server validates
the room, role, event, payload, message size, and revision before
accepting or broadcasting updates.

## API conventions

-   Use Pydantic request and response schemas.
-   Return consistent error structures.
-   Use appropriate HTTP status codes.
-   Validate all input on the backend.
-   Never return password hashes, invitation token hashes, secrets, or
    hidden test cases.
-   Document endpoints with FastAPI's OpenAPI support.
