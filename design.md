# TypingAI Design Document

## 1. Overview
TypingAI is a full-stack typing platform with three primary experiences:
- Single-player typing test
- Time-boxed practice mode
- Real-time multiplayer battleground

The system uses a React + Redux frontend with an Express + Socket.IO + MongoDB backend.

## 2. High-Level Architecture

### 2.1 Frontend
- Framework: React 18 + TypeScript + Vite
- State: Redux Toolkit (`auth`, `profile`, `typing`, `practice`, `room`, `ai*` slices)
- Routing: React Router
- Styling: Tailwind CSS and custom utility classes
- Networking: Axios instance with JWT interceptor
- Realtime: `socket.io-client` shared socket

### 2.2 Backend
- Framework: Express + TypeScript
- Database: MongoDB via Mongoose
- Realtime: Socket.IO room handlers in-memory
- Auth: JWT middleware + Google token verification
- AI service: Groq Chat Completions API
- Email: Nodemailer for activation/reset flows

## 3. Component Design

### 3.1 App Shell and Navigation
- `App.tsx` defines global navigation and route map.
- Protected pages are wrapped by `ProtectedRoute`.
- Theme state handled by `ThemeProvider`.
- Startup hydration loads `auth` user and profile when token exists.

### 3.2 Feature Modules
- Auth: login/register/google/reset/activation flows.
- Typing Test: AI generation, countdown lock, live metrics, results view.
- Practice: timer-first setup, script selection, progress and result summary.
- Battleground: room workflow, host privileges, race countdown, ranking.
- Learn: structured lessons, keyboard placement drills, completion certificate.
- Profile: stats dashboard, streak card, recent sessions, avatar management.

### 3.3 Shared Utilities
- Metrics calculations (`calcWPM`, `calcCPM`, `calcAccuracy`).
- Certificate generation/share/download.
- Avatar metadata and theme helpers.

## 4. Data Model

### 4.1 `User`
Core fields:
- Identity: `email`, `passwordHash`, `displayName`, `googleId`, `photoUrl`
- Profile: `avatarId`
- Aggregates: `bestWPM`, `averageAccuracy`, `totalTestsTaken`, `totalPracticeSessions`, `totalBattles`
- References: `history[]` (`TestResult`), `sessions[]` (`Session`)
- Recovery: `resetCode`, `resetCodeExpiry`
- Activation: `isActivated`

### 4.2 `TestResult`
Used for classic typing result records:
- `user`, `wpm`, `cpm`, `accuracy`, `errors`, `duration`, `text`, `room`

### 4.3 `Session`
Unified session model for test/practice/battle:
- `user`, `type`, `wpm`, `cpm`, `accuracy`, `errors`, `duration`, `text`
- Optional: `difficulty`, `mode`, `battleResult`, `opponent`

## 5. Request/Response Design

### 5.1 Axios Client
- Base URL: `VITE_API_URL + /api` or `/api` fallback.
- Request interceptor injects bearer token from `localStorage`.
- Response interceptor clears token on `401`.

### 5.2 Key HTTP Flows
- Auth flow: register -> activate -> login -> load `/users/me`.
- AI flow: post `/ai/generate` with `{topic,length,difficulty}`.
- Session flow: post `/sessions/create`; fetch `/sessions/user/:id` and `/sessions/stats/:id`.
- Result flow: optional post `/results` and get `/results/me`.

## 6. Real-Time Design (Battleground)

### 6.1 Room State
In-memory object keyed by room code:
- `text`
- `players` map keyed by socket ID (`name`, `progress`, `ready`, `wpm`, `accuracy`, `finished`)
- `host`
- `raceStart`
- `finishedPlayers[]`

### 6.2 Lifecycle
1. Host creates room (`room:create`).
2. Players join (`room:join`).
3. Host sets text (`room:setText`).
4. Players ready (`player:ready`).
5. Host starts race (`race:start`) with server timestamp + 5s delay.
6. Players send progress (`room:progress`) until finish.
7. Results computed client-side from finish order and saved as battle session.

### 6.3 Time Sync
- `SocketProvider` performs sample-based offset calculation using `time:request` / `time:response`.
- Clients use computed offset for synchronized countdown behavior.

## 7. State Design

### 7.1 Redux Slices
- `auth`: token/user/auth-check lifecycle.
- `profile`: user profile, avatar, history snapshot.
- `typing`: core typing text/typed/errors/timing.
- `practice`: timer, script mode, workflow state.
- `room`: multiplayer identity/workflow/players/race flags.
- `aiTest`, `aiPractice`, `aiMultiplayer`: generated text per context.

### 7.2 Derived/UI State
- Page-level UI state in components for dialogs, view stages, and temporary selections.
- Persistence decisions:
  - JWT in `localStorage`.
  - Most runtime session state in Redux.

## 8. Security and Reliability Design
- JWT auth middleware guards protected APIs.
- Passwords hashed with bcrypt.
- Activation + reset codes validated server-side.
- Google auth verified through `google-auth-library` or Google userinfo endpoint.
- Error handling returns clear JSON errors and logs server exceptions.

## 9. Deployment and Environment

### 9.1 Backend Environment
- `MONGO_URI`
- `PORT`
- `JWT_SECRET`
- `GROQ_API_KEY`
- `GOOGLE_CLIENT_ID`
- Email transport settings (`EMAIL_*`)
- `FRONTEND_URL`

### 9.2 Frontend Environment
- `VITE_API_URL`
- `VITE_GOOGLE_CLIENT_ID`

## 10. Current Design Risks and Gaps
- DG-1: Progress/streak endpoints used by frontend are missing in backend (`/users/progress`, `/users/learning`, `/users/streak/record`).
- DG-2: Room storage is in-memory only; horizontal scaling is not supported.
- DG-3: Typing test currently writes both session and result models (possible duplication).
- DG-4: CORS config is currently permissive (`origin: '*'`) in backend and Socket.IO.
- DG-5: Secrets are present in `.env`; operational security needs hardening.

## 11. Recommended Next Design Iterations
- Add backend progress/streak module with persistent schema and routes.
- Unify result persistence strategy (choose `Session` only or formal dual-write policy).
- Introduce Redis-backed Socket.IO adapter for scalable rooms.
- Add request validation layer (zod/joi) and rate limiting on auth endpoints.
- Tighten production CORS and secret management.
- Add integration tests for auth, session stats, and socket race lifecycle.
