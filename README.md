# AI Interviewer

This repository contains a monorepo for an interview automation project. It includes a backend service, a Bun-powered frontend, and shared UI components. The project is implemented in TypeScript and uses Prisma for data modeling and migrations.

**Repository layout**
- **apps/frontend**: Bun-based React frontend and bundler. See [apps/frontend/package.json](apps/frontend/package.json#L1-L35) and [apps/frontend/build.ts](apps/frontend/build.ts#L1-L150).
- **apps/backend**: TypeScript backend with Prisma and API routes. See [apps/backend/package.json](apps/backend/package.json#L1-L35) and [apps/backend/db.ts](apps/backend/db.ts#L1-L10).
- **apps/backend/prisma**: Prisma schema and migrations. See [apps/backend/prisma/schema.prisma](apps/backend/prisma/schema.prisma#L1-L42).
- **packages/ui**: Shared React UI components used by the frontend.

**High-level purpose**
This project runs automated interviews: it collects metadata (e.g., GitHub data), drives a conversation flow, stores the interaction messages and feedback, and computes a score. The backend persists interview state; the frontend provides an interview UI and audio integrations.

**Tech stack**
- **Language**: TypeScript (workspace TypeScript v5+).
- **Monorepo tooling**: Turbo (turbo) for running builds and scripts across packages. See [package.json](package.json#L1-L25).
- **Frontend**: Bun runtime, React 19, Bun bundler configuration and Tailwind integration. Frontend build and dev are driven by Bun (`bun --hot` and `bun run build.ts`). See [apps/frontend/package.json](apps/frontend/package.json#L1-L35).
- **Backend**: TypeScript server code, Express-compatible libraries available, WebSocket support, Playwright for scraping/testing, HTTP clients via Axios.
- **Database**: Prisma ORM with PostgreSQL as the datasource (see Prisma datasource provider). The Prisma schema and generated client live under `apps/backend/prisma` and `apps/backend/generated/prisma` respectively. See [apps/backend/prisma/schema.prisma](apps/backend/prisma/schema.prisma#L1-L42).
- **Other**: Zod for validation, Deepgram SDK and other third-party SDKs for audio/transcription, Radix UI primitives in the frontend.

**Architecture and approach**
- Monorepo structure: frontend and backend are separate workspaces under `apps/`, with shared UI components in `packages/ui`.
- Frontend is served and built with Bun. Development runs with Bun's hot-reload; build uses `apps/frontend/build.ts` which invokes Bun's bundler APIs.
- Backend uses Prisma as the single source of truth for data models. The main domain models include `Interview` and `Message` (see schema). Interview state transitions (Pre → InProgress → Done) and messages (User/Assistant) are persisted for replay and scoring.
- The backend exports a Prisma client wrapper in `apps/backend/db.ts` which configures the PostgreSQL adapter.
- Data flow: frontend drives interview interactions (messages, audio capture) → backend APIs accept messages and persist them → scoring and feedback stored on the Interview model → optional integrations (GitHub scraper, external SDKs) enrich metadata.

**Important files**
- Project manifest: [package.json](package.json#L1-L25)
- Frontend manifest: [apps/frontend/package.json](apps/frontend/package.json#L1-L35)
- Frontend build: [apps/frontend/build.ts](apps/frontend/build.ts#L1-L150)
- Backend manifest: [apps/backend/package.json](apps/backend/package.json#L1-L35)
- Prisma schema: [apps/backend/prisma/schema.prisma](apps/backend/prisma/schema.prisma#L1-L42)
- DB client: [apps/backend/db.ts](apps/backend/db.ts#L1-L10)

**Development**
Prerequisites:
- Bun (workspace `package.json` sets `packageManager` to `bun@1.3.11`).
- Node 18+ for some tooling that expects Node (Turbo). 
- A PostgreSQL instance and a `DATABASE_URL` environment variable for Prisma migrations and runtime.

Typical local setup:

```bash
# Install dependencies at the repo root
bun install

# Generate Prisma client (from apps/backend)
cd apps/backend
npx prisma generate

# Apply migrations (ensure DATABASE_URL is set)
npx prisma migrate dev

# Start development for all workspaces (turbo orchestrates workspace scripts)
cd ../.. # back to repo root
bun run dev
```

If you prefer to run services individually:

```bash
# Frontend (hot reload)
cd apps/frontend
bun run dev

# Backend (start according to available index files)
cd ../backend
# if there's an entry point like index.ts, run it with Bun or Node
bun index.ts
```

**Database & Migrations**
- The project uses Prisma with PostgreSQL. Schema is in [apps/backend/prisma/schema.prisma](apps/backend/prisma/schema.prisma#L1-L42).
- Keep `DATABASE_URL` in your environment (or a .env in the backend folder) and run `npx prisma generate` and `npx prisma migrate dev` when changing models.

**Testing**
- Frontend: Bun's test runner can be used (`bun test`) if tests are added.
- Backend: Playwright is included as a dependency and may be used for end-to-end or scraping tasks. Add unit and integration tests as needed.

**Deployment notes**
- Frontend: build with Bun (`bun run build.ts`) and serve static `dist` artifacts with a static host or Bun. The build is configured to output optimized assets.
- Backend: deploy with your preferred Node/TS-capable host or containerize with an environment that provides `DATABASE_URL`. Ensure the Prisma client is generated during build.

**Contributing / Next steps**
- Add explicit start scripts for the backend (root turbo scripts expect workspace scripts to exist).
- Add README sections for API endpoints and frontend UI flows when the routes are stabilized.
- Add CI steps for Prisma migrations, type checking, and frontend build validation.

