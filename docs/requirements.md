# CodePair --- Product Requirements

## 1. Product vision

CodePair provides a dedicated online environment for structured
technical mock interviews. An interviewer creates a room and invites a
candidate. Both participants see the same coding problem and
collaboratively work in a shared editor. The candidate's solution can be
executed against test cases, and the interviewer records structured
feedback.

## 2. Target users

### Interviewer

Creates and manages interview rooms, chooses questions, observes the
candidate's work, controls the interview timer, and submits an
evaluation.

### Candidate

Joins an invited room, reads the problem, writes and runs code, and
reviews feedback after the interview.

## 3. Goals

-   Enable two users to participate in the same interview room.
-   Synchronize editor changes in real time.
-   Provide a shared interview timer and participant presence.
-   Execute submitted code using an isolated external execution service.
-   Preserve completed session details and final code.
-   Collect structured interviewer feedback.
-   Provide AI-assisted summaries as a later enhancement.

## 4. Functional requirements

  ID      Requirement                                   Priority
  ------- --------------------------------------------- ----------
  FR-01   Register and authenticate users               Must
  FR-02   View and update user profile                  Must
  FR-03   Create an interview room                      Must
  FR-04   Join a room using an invitation link/token    Must
  FR-05   Limit a room to two active participants       Must
  FR-06   Display the selected coding question          Must
  FR-07   Provide a Monaco-based code editor            Must
  FR-08   Synchronize editor changes in real time       Must
  FR-09   Display participant presence                  Must
  FR-10   Synchronize the interview timer               Must
  FR-11   Execute code through Judge0                   Must
  FR-12   Persist interview sessions                    Must
  FR-13   Allow the interviewer to submit a scorecard   Must
  FR-14   Show session history and feedback             Must
  FR-15   Generate an AI performance summary            Should
  FR-16   Share live cursor positions                   Should
  FR-17   Replay a previous session                     Later
  FR-18   Provide a collaborative whiteboard            Later

## 5. Roles and permissions

  Capability                 Interviewer   Candidate
  -------------------------- ------------- ----------------------
  Create room                Yes           Yes
  Join invited room          Yes           Yes
  Select/change question     Yes           No
  Edit shared code           Yes           Yes
  Execute code               Yes           Yes
  Control timer              Yes           No
  End interview              Yes           No
  Submit evaluation          Yes           No
  View own session history   Yes           Yes
  View candidate feedback    Yes           Yes, for own session

The room creator is the interviewer by default. The invited participant
is the candidate. Role switching is outside the initial release.

## 6. Non-functional requirements

-   Performance: low-latency editor synchronization under normal network
    conditions.
-   Reliability: handle disconnects and reconnects without silently
    corrupting room state.
-   Security: authenticate REST and WebSocket connections; enforce room
    membership and role permissions.
-   Consistency: maintain an authoritative server-side room state and
    detect stale editor revisions.
-   Maintainability: use modular backend domains and feature-oriented
    frontend organization.
-   Data integrity: preserve completed session records, final code, and
    evaluations.
-   Privacy: avoid sending unnecessary personal data to external AI
    services and define retention behavior.
-   Safety: never execute untrusted code inside the FastAPI process.

## 7. MVP acceptance criteria

-   Two authenticated users can join the same room.
-   Both users see the same selected question.
-   Both users can edit the shared code.
-   Changes are synchronized and stale updates are handled predictably.
-   Participant presence is visible.
-   Both participants see a synchronized timer.
-   Code executes through Judge0 with configured limits.
-   The interviewer can submit an evaluation.
-   The candidate can view their feedback.
-   Completed interviews appear in session history.

## 8. Out of scope for the first release

-   More than two active participants in a room.
-   Built-in video/audio conferencing.
-   Production-grade distributed WebSocket scaling.
-   Yjs/CRDT collaboration in the first iteration.
-   Session replay and collaborative whiteboard.
-   Custom container-based execution infrastructure.
