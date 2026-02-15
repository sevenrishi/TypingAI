# TypingAI Requirements

## 1. Purpose
TypingAI is a full-stack web application for typing improvement with AI-generated scripts, timed practice, multiplayer races, and learning progress tracking.

## 2. Scope
This document describes requirements based on the current codebase in:
- `TypingAI-frontend`
- `TypingAI-backend`

It includes implemented requirements and implementation gaps observed in the current project.

## 3. Product Goals
- Improve typing speed and accuracy through guided and free-form training.
- Provide engaging modes: test, practice, and battleground (multiplayer).
- Persist user history and surface progress over time.
- Support account-based experience with JWT auth and Google sign-in.

## 4. User Roles
- Guest user: can view landing content and open auth flows.
- Authenticated user: can access all protected training, profile, and account pages.

## 5. Functional Requirements

### 5.1 Authentication and Account
- FR-A1: User can register with `email`, `password`, and `displayName`.
- FR-A2: System sends account activation email after registration.
- FR-A3: User can activate account using `/auth/activate?code=<userId>`.
- FR-A4: User can log in with email/password after activation.
- FR-A5: User can log in via Google OAuth token/credential.
- FR-A6: JWT token is stored client-side and sent in `Authorization` header.
- FR-A7: Protected routes must require authenticated user state.
- FR-A8: User can request password reset code, verify code, and set new password.

### 5.2 Typing Test Mode
- FR-T1: User can generate AI script from topic and selected length (`short|medium|long`).
- FR-T2: User can start test only after script generation.
- FR-T3: App calculates and displays WPM, CPM, accuracy, errors, elapsed time.
- FR-T4: Completed test result is persisted to backend.
- FR-T5: Session summary is saved with type `test`.

### 5.3 Practice Mode
- FR-P1: User selects timer (1m, 2m, 5m, 10m).
- FR-P2: User chooses script source: curated script or AI-generated script.
- FR-P3: Timer-based typing session runs with live metrics.
- FR-P4: Session ends on timer completion or early text completion.
- FR-P5: Session summary is saved with type `practice` and mode metadata.

### 5.4 Battleground (Multiplayer)
- FR-B1: User enters player name and can create or join room.
- FR-B2: Host can generate/review/select race script before start.
- FR-B3: Room state sync includes players, host, text, ready status, race start timestamp.
- FR-B4: Players can toggle ready and host starts race.
- FR-B5: Real-time progress updates broadcast progress, WPM, and accuracy.
- FR-B6: Finish order is tracked and race results are displayed.
- FR-B7: Battle session is persisted with `battleResult` and optional `opponent` list.

### 5.5 Learn Mode
- FR-L1: User can navigate lesson phases and run guided typing drills.
- FR-L2: App tracks lesson completion state and progress percentage.
- FR-L3: App supports finger-placement guided drills in phase-1 lessons.
- FR-L4: Completing all tracked lessons issues certificate data.
- FR-L5: User can share/download certificate.

### 5.6 Profile and Account
- FR-U1: User can view profile details and historical stats.
- FR-U2: User can change avatar from predefined avatar set.
- FR-U3: User can view recent sessions and aggregated test/practice/battle stats.
- FR-U4: User can view and share streak/certificate info from profile UI.

### 5.7 Results and Session History
- FR-R1: System stores individual test results (`TestResult` collection).
- FR-R2: System stores normalized session records (`Session` collection).
- FR-R3: User can fetch paginated sessions and stats summary by user ID.

## 6. Backend API Requirements

### 6.1 Implemented REST Endpoints
- `POST /api/ai/generate`
- `POST /api/auth/register`
- `POST /api/auth/login`
- `POST /api/auth/google`
- `POST /api/auth/forgot-password`
- `POST /api/auth/verify-reset-code`
- `POST /api/auth/reset-password`
- `GET /api/auth/activate`
- `GET /api/users/me`
- `PUT /api/users/avatar`
- `POST /api/results`
- `GET /api/results/me`
- `POST /api/sessions/create`
- `GET /api/sessions/user/:userId`
- `GET /api/sessions/stats/:userId`
- `GET /api/sessions/:sessionId`

### 6.2 Implemented Socket Events
Client to server:
- `room:create`
- `room:join`
- `room:leave`
- `room:progress`
- `player:ready`
- `race:start`
- `race:reset`
- `room:setText`
- `time:request`

Server to client:
- `room:state`
- `room:error`
- `race:start`
- `room:host`
- `time:response`

## 7. Data Requirements
- DR-1: MongoDB stores `User`, `TestResult`, and `Session` documents.
- DR-2: User must maintain references to `history` and `sessions`.
- DR-3: Session record must include type, metrics, duration, and source text.
- DR-4: Battle sessions should include `battleResult` and optional opponent metadata.

## 8. Non-Functional Requirements
- NFR-1: Frontend is responsive for desktop and mobile layouts.
- NFR-2: Token-protected APIs must reject unauthorized requests with 401.
- NFR-3: Socket room updates should be near real-time.
- NFR-4: App should handle AI/API failures with user-visible fallback messages.
- NFR-5: Build stack: React + Vite + TypeScript (frontend), Express + TypeScript (backend).

## 9. Security Requirements
- SR-1: Do not commit real secrets in `.env` to version control.
- SR-2: Rotate leaked credentials immediately if exposed.
- SR-3: JWT secret must be set in environment for production.
- SR-4: CORS origin should be restricted in production.
- SR-5: Password reset and activation flows should include expiry/validation checks.

## 10. Known Gaps (Current Implementation)
- GAP-1: Frontend calls `/users/progress`, `/users/learning`, and `/users/streak/record`, but backend does not currently implement these routes.
- GAP-2: `TypingTest` currently saves both `/sessions/create` and `/results` for one completion; this duplicates persistence paths.
- GAP-3: Some docs and `.env.example` keys are inconsistent (`MONGO_URI` vs `MONGODB_URI`, OpenAI wording vs Groq usage).
- GAP-4: Registration response does not return JWT token, while some old docs imply token return.

## 11. Acceptance Criteria (Current Baseline)
- AC-1: Authenticated user can complete test, practice, and battleground sessions.
- AC-2: Completed sessions are retrievable in profile/session history.
- AC-3: Multiplayer room state and race start synchronize across connected clients.
- AC-4: Password reset and account activation flows are functional end-to-end.
- AC-5: AI generation supports topic and length controls and returns single paragraph text.
