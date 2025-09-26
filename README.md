## SpeakEasy

Live App: https://speak-easy-frontend-eta.vercel.app/

A full‑stack language exchange app where learners can discover partners, send/accept friend requests, chat in real time, and share instant video call links. Built with a Node/Express + MongoDB backend and a React + Vite frontend, integrating Stream Chat for messaging and optional video call link sharing.

### Key Features
- Discover recommended partners after onboarding (language, bio, location)
- Send/accept friend requests; manage incoming/outgoing requests
- Auth with HTTP‑only cookies (JWT)
- Real‑time 1:1 chat powered by Stream Chat
- One‑click “Start Call” button sends a video call link in chat
- Theming (DaisyUI/Tailwind) and responsive UI

## Tech Stack

- Backend: Express, Mongoose (MongoDB), JWT, CORS, Cookie‑Parser, Morgan, Dotenv
- Frontend: React 19, Vite, React Router, TanStack Query, Stream Chat React, Zustand, Axios, Tailwind CSS + DaisyUI, React Hot Toast
- Realtime/Chat: Stream Chat

## Monorepo Structure

```
.
├─ backend/
│  ├─ src/
│  │  ├─ controllers/ (auth, user, chat)
│  │  ├─ lib/ (db, stream)
│  │  ├─ middleware/ (auth)
│  │  ├─ models/ (User, FriendRequest)
│  │  ├─ routes/ (auth, user, chat)
│  │  └─ server.js
└─ frontend/
   └─ src/ (pages, components, hooks, lib, store)
```

## Environment Variables

Create a `.env` file in `backend/` with:

```env
PORT=5001
MONGO_URI=mongodb+srv://<user>:<password>@<cluster>/<db>?retryWrites=true&w=majority
JWT_SECRET_KEY=replace-with-strong-secret
NODE_ENV=development

# Stream Chat (server)
# Note: backend expects STEAM_API_* (spelled as in code). If you control the code, consider renaming to STREAM_API_*.
STEAM_API_KEY=your_stream_api_key
STEAM_API_SECRET=your_stream_api_secret
```

Create a `.env` file in `frontend/` with:

```env
VITE_STREAM_API_KEY=your_stream_api_key
```

Backend CORS is configured for `http://localhost:5173` in development and serves the built frontend in production.

## Install & Run

### Prerequisites
- Node.js 18+ and npm
- MongoDB database (Atlas or local)
- Stream Chat account (API key/secret)

### Backend

```bash
cd backend
npm install
npm run dev
```

or PowerShell:

```powershell
cd backend
npm install
npm run dev
```

Server starts on `http://localhost:5001`.

### Frontend

```bash
cd frontend
npm install
npm run dev
```

or PowerShell:

```powershell
cd frontend
npm install
npm run dev
```

App runs on `http://localhost:5173`.

### Production Build

```bash
cd frontend && npm run build
```

Then ensure backend `NODE_ENV=production`; it will serve `frontend/dist`.

## Application Flow

1) Authentication
- Signup `POST /api/auth/signup` creates a user, sets `jwt` httpOnly cookie.
- Login `POST /api/auth/login` sets `jwt` cookie; Logout clears it `POST /api/auth/logout`.
- Session check `GET /api/auth/me` returns `{ success, user }` when authenticated.

2) Onboarding
- `POST /api/auth/onboarding` (protected): submit `fullName, bio, nativeLanguage, learningLanguage, location` to mark `isOnboarded=true`.
- On success, user is upserted to Stream Chat (server‑side).

3) Discover & Friends
- Recommended users: `GET /api/users` (protected) excludes self, existing friends, and non‑onboarded users.
- My friends: `GET /api/users/friends` returns populated list.
- Send request: `POST /api/users/friend-request/:id`
- Accept request: `PUT /api/users/friend-request/:id/accept`
- Incoming requests: `GET /api/users/friend-requests`
- Outgoing requests: `GET /api/users/outgoing-friend-requests`

4) Chat & Calls
- Chat token: `GET /api/chat/token` (protected) returns a server‑generated Stream token for the logged‑in user.
- Frontend connects to Stream Chat using `VITE_STREAM_API_KEY` and the token, then opens a 1:1 channel whose id is a sorted combination of the two user ids.
- The “Start Call” button composes a call URL like `/call/<channelId>` and sends it as a message in the chat. The call page can use your preferred video SDK; the included UI surfaces the link.

## Frontend Routing (guards)

- `/` and `/friends`: require authenticated + onboarded; otherwise redirect to `/login` or `/onboarding`.
- `/signup`, `/login`: redirect to `/` or `/onboarding` if already authenticated.
- `/notifications`: requires authenticated + onboarded.
- `/chat/:id`: requires authenticated + onboarded; renders Stream Chat UI.
- `/onboarding`: accessible only when authenticated and not yet onboarded.

## Notable Implementation Details

- Cookies: JWT is stored in an httpOnly cookie `jwt` with `sameSite=strict` and `secure` in production.
- Axios: `withCredentials: true` is enabled; dev baseURL points to `http://localhost:5001/api`.
- Stream Chat server helper uses env vars `STEAM_API_KEY` and `STEAM_API_SECRET` (typo in variable prefix is intentional to match code). Ensure they’re set.

## Scripts

- Backend
  - `npm run dev` — nodemon on `src/server.js`
  - `npm start` — node `src/server.js`
- Frontend
  - `npm run dev` — Vite dev server
  - `npm run build` — production build
  - `npm run preview` — preview built app

## Troubleshooting

- 401 or redirects to login: ensure cookies are enabled and backend CORS `credentials` is allowed, and requests are sent with `withCredentials`.
- Stream token errors: verify `STEAM_API_KEY/STEAM_API_SECRET` (backend) and `VITE_STREAM_API_KEY` (frontend) are correct and the user id passed to token generation matches the connected user id.
- CORS errors: confirm frontend dev URL matches the backend CORS `origin` (`http://localhost:5173`).

## License

MIT


