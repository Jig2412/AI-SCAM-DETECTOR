# AI Scam Caller Detector

## Overview

This is a full-stack web application called "AI Scam Caller Detector" that analyzes suspicious phone call transcripts or scam messages using Google Gemini AI. Users paste a transcript or message, and the app returns a detailed scam analysis including risk score (0–100), scam category, red flags, psychological manipulation tactics, recommended actions, and confidence level. Results are displayed with animated cards and a visual risk gauge. Recent analyses are kept in memory for quick review.

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend
- **Framework**: React 18 with TypeScript, bundled by Vite
- **Routing**: wouter (lightweight client-side router)
- **State/Data fetching**: TanStack React Query for server state management, with custom hooks (`useAnalyze`, `useHistory`) wrapping mutations and queries
- **UI Components**: shadcn/ui (new-york style) built on Radix UI primitives, styled with Tailwind CSS and CSS variables for theming
- **Animations**: framer-motion for staggered result card animations and transitions
- **Icons**: lucide-react
- **Theme**: Dark, security-focused design with CSS custom properties for colors. The theme uses HSL color variables defined in `client/src/index.css`
- **Path aliases**: `@/` maps to `client/src/`, `@shared/` maps to `shared/`, `@assets/` maps to `attached_assets/`

### Backend
- **Runtime**: Node.js with Express.js
- **Language**: TypeScript, executed via tsx
- **API Design**: RESTful with two endpoints defined in `shared/routes.ts`:
  - `POST /api/analyze` — accepts a transcript, sends it to Gemini AI, returns structured scam analysis JSON
  - `GET /api/history` — returns recent analysis history
- **Input Validation**: Zod schemas shared between client and server (`shared/schema.ts`). The server validates request bodies; the client pre-validates before sending and parses responses.
- **AI Integration**: Google Gemini API (`@google/genai` package) with structured JSON output via `responseSchema`. Uses the `gemini-2.5-flash` model. The API key is stored in the `GEMINI_API_KEY` environment variable and never exposed to the client.
- **Development**: Vite dev server is used as middleware in development mode (HMR via `server/vite.ts`). In production, the built static files are served from `dist/public`.

### Data Storage
- **Current**: In-memory storage (`MemStorage` class in `server/storage.ts`). Stores the last 5 analyses in an array. No database is currently used for application data.
- **Database Configuration**: Drizzle ORM with PostgreSQL is configured (`drizzle.config.ts`) and the schema file is at `shared/schema.ts`, but the schema currently only defines Zod types for the API — no Drizzle table definitions exist yet. The `DATABASE_URL` environment variable is expected for Drizzle but is not required for the app to run since storage is in-memory.
- **Session Store**: `connect-pg-simple` is listed as a dependency but not actively used.

### Shared Code (`shared/`)
- `schema.ts` — Zod schemas for request/response types (`AnalyzeRequest`, `AnalyzeResponse`, `HistoryItem`)
- `routes.ts` — API route definitions with paths, methods, input schemas, and response schemas. This serves as a contract between frontend and backend.

### Build System
- **Client build**: Vite outputs to `dist/public`
- **Server build**: esbuild bundles `server/index.ts` to `dist/index.cjs` with select dependencies bundled (allowlisted) to reduce cold start times
- **Commands**:
  - `npm run dev` — development mode with HMR
  - `npm run build` — production build (client + server)
  - `npm start` — run production build
  - `npm run db:push` — push Drizzle schema to database

## External Dependencies

### APIs & Services
- **Google Gemini AI**: Used for scam transcript analysis. Requires `GEMINI_API_KEY` environment variable. Uses the `@google/genai` SDK with structured JSON output mode.

### Database
- **PostgreSQL**: Configured via Drizzle ORM but not actively used for app data. Requires `DATABASE_URL` environment variable when using Drizzle commands (`db:push`). The app currently functions without a database using in-memory storage.

### Key NPM Packages
- `express` — HTTP server
- `drizzle-orm` + `drizzle-kit` — ORM (configured but tables not yet defined)
- `@google/genai` — Google Gemini AI client
- `zod` + `drizzle-zod` — Schema validation
- `@tanstack/react-query` — Client-side data fetching
- `framer-motion` — Animations
- `wouter` — Client routing
- `shadcn/ui` ecosystem (Radix UI, class-variance-authority, tailwind-merge, clsx)
- `nanoid` — ID generation
- `connect-pg-simple` — PostgreSQL session store (available but not actively wired)

### Environment Variables
- `GEMINI_API_KEY` — Required for AI analysis functionality
- `DATABASE_URL` — Required only for Drizzle database operations
- `NODE_ENV` — Controls development vs production behavior