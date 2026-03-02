# ContinuLearn

ContinuLearn is an interactive learning platform for continuum robotics, combining a gamified roadmap, live simulation, and AI-assisted coaching.

## Inspiration

We built ContinuLearn because we love robotics and wanted to make continuum robotics accessible beyond a small niche. Most people learn rigid-link robots first, while continuum systems (catheter-like, trunk-like robots) stay hard to access. We wanted to turn that gap into an interactive learning experience.

## What It Does

ContinuLearn helps users explore continuum robotics through a gamified roadmap and live simulator. Users learn by adjusting key parameters like curvature, bend direction, and segment length, then validating their understanding through challenge checks.

For example, each segment follows constant-curvature intuition, where bend angle and curvature/radius relationships drive shape and tip behavior in 3D. This helps users connect intuition, math, and robot behavior in real time.

## How We Built It

We built a web app with:

- Next.js + React for the frontend experience and roadmap flow
- Unity + Blender for 3D continuum simulation and custom robot assets
- Unity WebGL to embed the simulator directly in the site
- AI coaching APIs for level hints, explanations, and voice-assisted support
- SQLite + auth scaffolding for persistence and user progress

We designed a level system where users unlock progress by solving simulator tasks, not just reading content.

## Challenges We Ran Into

One of the hardest parts was designing custom continuum robot parts and making them work together correctly in simulation.

We also faced challenges with:

- Unity rendering/setup issues (URP and WebGL pipeline details)
- Getting WebGL build integration, paths, and headers correct in the web app
- Bridging frontend controls to Unity runtime behavior reliably
- Keeping roadmap UI/UX smooth and modular as features expanded

## Accomplishments We're Proud Of

- Building a working pick-and-place robot demonstration
- Shipping a full web product around a niche robotics domain
- Creating an accessible learning experience that combines simulation, roadmap progression, and AI coaching
- Getting math/voice-aware feedback integrated into the learning flow

## What We Learned

We learned a lot about end-to-end robotics product development:

- Unity + Blender workflows for custom simulation assets
- Deploying Unity projects to the web via WebGL
- Integrating real-time simulation with a modern web stack
- Designing learning systems where validation is interactive and measurable

## What's Next For ContinuLearn

- Procedurally generated levels with obstacle-rich environments to highlight continuum robot advantages
- Dual-robot roadmap scenarios focused on coordinating two continuum robots
- More advanced singularity/redundancy lessons with adaptive coaching
- Deeper personalization based on user progress and error patterns

## Core Learning Experience

1. Users sign in and open the learning map at `/learn`.
2. They pick a track (`practical`, `theory`, `pickplace`) and unlock levels sequentially.
3. Practical levels launch the simulator (`/app?level=<id>`) where parameter checks validate completion.
4. Theory levels provide lessons/problems and optional AI review chat for student attempts.
5. Pick & place levels launch dedicated scenes (`/pick-place?level=<id>`) with manual completion marking.
6. Progress/settings are persisted per authenticated user via repository-backed API routes.

## How It Works (Architecture)

### Frontend

- Next.js App Router (`app/`) for pages and server routes
- React client components for simulator controls, learning map, auth state, and AI UX
- Tailwind CSS + Radix UI-based components in `components/ui/`
- Unity WebGL loaded dynamically through `hooks/use-unity-webgl.ts`

### Simulation Layer

- Practical simulator shell: `components/simulator/simulator-shell.tsx`
- Pick & place simulator shell: `app/pick-place/page.tsx`
- Unity builds served from `public/unity`, `public/unity2`, `public/unity3`
- Level definitions and evaluation logic in `lib/levels.ts`, `lib/theory-levels.ts`, `lib/pick-place-levels.ts`

### AI Layer

- Main usage gateway: `POST /api/ai/usage`
- Coach explanation: `POST /api/coach`
- Level hints: `POST /api/coach/level-hint`
- Theory review chat (supports files/images): `POST /api/coach/theory-chat`
- Theory helper mode endpoint: `POST /api/coach/theory-help`
- Voice narration: `POST /api/narrate`
- Voice query (speech-to-text + AI answer): `POST /api/ai/voice-query`

Gemini is used for text generation/reasoning; ElevenLabs is used for narration and transcription features when configured.

### Auth Layer

- Provider switch: `NEXT_PUBLIC_AUTH_PROVIDER` (`local` or `auth0`)
- Local mode uses app auth routes with session cookie handling
- Auth0 mode uses redirect OAuth flow (`/api/auth/login`, `/api/auth/signup`, `/api/auth/callback`, `/api/auth/logout`)
- Session endpoint: `GET /api/auth/session`

### Data Layer

Repository abstraction in `lib/db`:

- `SqliteRepository` (local development fallback)
- `TursoRepository` (production/serverless persistence)
- Adapter-based backend selection from environment configuration

Primary persisted entities (`db/schema.sql`):

- `users`
- `user_settings`
- `learning_progress`
- `simulator_snapshots`
- `theory_chat_threads`
- `theory_chat_messages`
- `auth_sessions`

## Project Layout

```text
app/
  api/                  # App Router API endpoints (auth, coach, AI, user data, db health)
  app/                  # Main practical simulator route
  learn/                # Learning map and level dialogs
  pick-place/           # Pick & place simulation route
  simulator/            # Alias route to simulator shell
  login/, signup/, intro/

components/
  auth/                 # Auth context and auth buttons
  landing/              # Marketing/landing sections
  learn/                # Learning map board UI
  simulator/            # Control panel, coach panel, Unity placeholders
  ui/                   # Shared UI primitives

lib/
  ai/                   # Gemini and ElevenLabs integrations
  auth/                 # Auth config, Auth0 helpers, session helpers
  db/                   # Repository interfaces + SQLite/Turso implementations
  levels.ts             # Practical levels and check/evaluation logic
  theory-levels.ts      # Theory track content
  pick-place-levels.ts  # Pick & place track content
  kinematics.ts         # Core kinematics helpers

hooks/
  use-unity-webgl.ts    # Unity loader/instance lifecycle

db/
  schema.sql            # Canonical SQL schema

docs/
  auth0-setup.md
  persistence-notes.md
  unity-webgl-integration.md
```

## Technology Stack

- Framework: Next.js 16, React 19, TypeScript
- Styling/UI: Tailwind CSS v4, Radix UI primitives, shadcn-style components
- Simulation: Unity WebGL (built assets in `public/unity*`) + Blender-authored robot assets
- Math rendering: KaTeX + `remark-math`/`rehype-katex`
- Persistence:
  - Local: `better-sqlite3`
  - Cloud: Turso (`@libsql/client`)
- Authentication: local session mode or Auth0
- AI: Gemini (coaching/reasoning), ElevenLabs (voice)

## Local Development

### Prerequisites

- Node.js 20+
- npm

### Install

```bash
npm install
```

### Configure Environment

Copy `.env.example` to `.env.local` and fill values.

```env
NEXT_PUBLIC_AUTH_PROVIDER=auth0

AUTH0_DOMAIN=...
AUTH0_CLIENT_ID=...
AUTH0_CLIENT_SECRET=...
AUTH0_BASE_URL=http://localhost:3000

TURSO_DATABASE_URL=libsql://...
TURSO_AUTH_TOKEN=...
# SQLITE_DB_PATH=./db/app.sqlite

GEMINI_API_KEY=...
OPENAI_API_KEY=...
ELEVENLABS_API_KEY=...
ELEVENLABS_VOICE_ID=21m00Tcm4TlvDq8ikWAM
ELEVENLABS_MODEL=eleven_turbo_v2_5

NEXT_PUBLIC_APP_URL=http://localhost:3000
```

Notes:

- If Turso variables are set, Turso is used.
- If Turso variables are missing (non-Vercel), local SQLite is used.
- On Vercel without Turso config, initialization fails intentionally.

### Run

```bash
npm run dev
```

Open `http://localhost:3000`.

## Scripts

- `npm run dev` - Start development server
- `npm run build` - Build production app
- `npm run start` - Start production server
- `npm run lint` - Run ESLint

## API Surface (High-Level)

- Auth: `/api/auth/login`, `/api/auth/signup`, `/api/auth/callback`, `/api/auth/logout`, `/api/auth/session`
- User data: `/api/user/progress`, `/api/user/settings`
- Coaching/AI: `/api/coach`, `/api/coach/level-hint`, `/api/coach/theory-help`, `/api/coach/theory-chat`, `/api/ai/usage`, `/api/ai/voice-query`, `/api/narrate`
- Infra: `/api/db/health`

## Deployment

This project is designed for deployment on Vercel with Turso-backed persistence.

Recommended docs before deployment:

- `docs/auth0-setup.md`
- `docs/persistence-notes.md`
- `docs/unity-webgl-integration.md`
