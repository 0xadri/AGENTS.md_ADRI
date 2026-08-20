# Author

I'm Adri, the creator of this web app.
You're my agent.
We work together a lot.

I love to build.
I focus on building complex things as simple as possible.
I love to find ways to reduce complexity when solving problems.

Below are my preferences so we can be more aligned.

---

# Default Response Style

IMPORTANT by default all your answers must:

- ALWAYS use skill `exe-caveman-style`.
- ALWAYS put each full sentence on its own line.

## Auto-Clarity Response Style

Halt `Default Response Style` and use normal clear prose for:

- security warnings
- irreversible or destructive actions
- multi-step sequences where ordering matters
- technically ambiguous compressed phrasing

Once such part is over, make sure to resume `Default Response Style` (c.f. section above).

---

# Questions Are Read-Only

- A question is a request for an answer, not for changes.
- If the answer is obvious and the change trivial, still answer first and offer the change. Ask before making it.
- A question is a prompt that either:
  - finishes by a question mark "?"
  - opens with "why does", "how can", "what is", "should we", "is it possible", "can X do Y", "how hard would it be", "what are your thoughts", or such asks

---

# Markdown Files: Editing & Writing

IMPORTANT if you edit or write markdown files:

- ALWAYS use `exe-caveman-style`
- ALWAYS put each full sentence on its own line.
  Preserve normal Markdown structure, but avoid wrapping multiple sentences onto one physical line.

---

# This Project

INSERT_ONE_LINE_DESCRIPTION

It's a Fullstack TypeScript monorepo with React frontend, Express API backend, and shared library.

- **Status:** Early development, Private Beta, solo developer (Adri)
- **Stack:** React 19, Vite 7, Tailwind v4, Express 4, Prisma 7, PostgreSQL
- **Package manager:** pnpm workspaces — ALWAYS use `pnpm`, never npm or yarn

---

# Hard stops — ALWAYS ask user before doing these

- Modifying types in `packages/shared/` — affects both frontend and backend simultaneously
- Changing `API_ROUTES` constants in `packages/shared/` — silently breaks all callers across the monorepo
- Modifying Zod validation schemas — tightening or loosening them changes the API contract for both sides
- Installing new pnpm dependencies — `pnpm add` in any package
- NEVER add, remove, or upgrade dependencies without explicit approval.
- Touching auth/security code — JWT logic, CORS config, helmet, rate limiting
- Changing Prisma schema — even with `db:push`, confirm first
- Adding global state management (Redux, Zustand, Jotai, etc.) — architectural decision, not an implementation detail; project uses React hooks/context
- NEVER commit changes without explicit user approval.
- NEVER edit or delete any existing files in `docs/plans_done/` directory. Creating or adding new files is allowed.

---

# Coding Preferences - General

- If writing commit messages, NEVER auto-add your agent name as co-author
- Keep things simple. Channel "yagni" energy unless told otherwise.
- If making technical decisions, do not give much weight to development cost.
  Instead, prefer quality, simplicity, robustness, scalability, and long term maintainability.
- Typesafety is useful, take advantage of it.
- Comments are a great way to clarify functionality and how code is used. Don't comment every line, but feel free to describe (concisely) how functions are used above function definitions, classes, etc.
- Keep comments up to date! When making changes, it's important to keep things in sync.

---

# Architecture

## Monorepo structure

```
packages/
  frontend/   React + Vite app
  backend/    Express API
  shared/     Types, constants, utils — imported by both
```

## Backend layer order

Routes → Controllers → Services → Repositories → Database

Never skip layers. Don't put business logic in controllers or routes.

## Frontend structure

Feature-based: co-locate components, hooks, services, types, and tests per feature under `packages/frontend/src/features/`.

---

# Engineering Rules

## Naming conventions

| Context                     | Convention                                    |
| --------------------------- | --------------------------------------------- |
| React components            | PascalCase (`TripCard.tsx`)                   |
| Files match component name  | `SearchBox.tsx` exports `SearchBox`           |
| Functions, variables, hooks | camelCase                                     |
| Classes, types, interfaces  | PascalCase                                    |
| Database columns (Prisma)   | snake_case                                    |
| Application code            | camelCase (Prisma maps to camelCase)          |
| Environment variables       | SCREAMING_SNAKE_CASE                          |
| API routes                  | Plural resources (`/api/trips`, `/api/users`) |

## TypeScript

- NEVER use `any` types — use proper types or `unknown`
- `any` is the enemy. Inferred types are our friend. Our systems should adapt to changes, instead of requiring changes everywhere.
- ALWAYS use shared types from `@MYPROJECT/shared` — never duplicate types that already exist there
- Zod validation for all backend API inputs

## Database

- **Schema changes:** use `prisma migrate dev --name <description>` for all schema changes — applies locally and generates the migration file needed for production. Never use `db:push` (generates no migration files, so `migrate deploy` in production would have nothing to apply).
- **Initial migration baseline:** already established at `packages/backend/prisma/migrations/20260119133435_init/`.
- **Reseeding:** seed data (reference users/trips/bookings) is imported from `@MYPROJECT/shared` dist at runtime.
  If anything in `packages/shared/` changed, run `pnpm build:shared` before reseeding — otherwise the seed uses stale data.
- Dev data is seed-generated and disposable; databases can be cleared/reseeded freely

## Docs first

Before working on a feature area, read the relevant doc:

| Area                        | Doc                                |
| --------------------------- | ---------------------------------- |
| API endpoints               | `docs/api-contracts-backend.md`    |
| Data models / Prisma schema | `docs/data-models-backend.md`      |
| Architecture & integration  | `docs/integration-architecture.md` |
| Source structure            | `docs/source-tree-analysis.md`     |
| Dev workflow & commands     | `docs/development-guide.md`        |

## SVG icons

- NEVER inline SVG markup in JSX — ALWAYS extract to a component under `packages/frontend/src/components/icons/`
- Before creating a new icon component, search for existing ones first (`Glob` on `packages/frontend/src/components/icons/**` or `Grep` for the icon name/shape)
- Reuse or extend an existing icon if similar; only create a new file if truly unique
- Icon components: PascalCase name, accept a `className` prop, export as named export

## Logging

ALWAYS use Winston — never `console.log` in backend code. The logger is set up in `error.middleware.ts` and should be imported from there or instantiated via `winston.createLogger`. Use appropriate levels: `logger.error`, `logger.warn`, `logger.info`, `logger.debug`.

## ID generation

ALWAYS use `@paralleldrive/cuid2` for generating IDs — never `uuid` or `crypto.randomUUID`. Import `createId` from `@paralleldrive/cuid2`.

## Auth flow

Magic link, passwordless. Never change this pattern without explicit instruction.

1. `POST /api/auth/magic-link` — user submits email, receives a verification code via AWS SES
2. `POST /api/auth/verify` — user submits email + code → returns JWT access token (in response body) + refresh token (HttpOnly cookie)
3. `GET /api/auth/me` — returns current user from access token

Access token: short-lived JWT in `Authorization: Bearer` header. Refresh token: HttpOnly cookie, used to silently re-issue access tokens. Auth middleware: `authenticateToken`.

## Backend error handling

ALWAYS use the existing error classes — never throw generic `Error` or invent new patterns.

**Error classes** (`packages/backend/src/utils/errors/`):

| Class               | Status | Code                    |
| ------------------- | ------ | ----------------------- |
| `NotFoundError`     | 404    | `NOT_FOUND`             |
| `ValidationError`   | 400    | `VALIDATION_ERROR`      |
| `UnauthorizedError` | 401    | `UNAUTHORIZED`          |
| `ForbiddenError`    | 403    | `FORBIDDEN`             |
| `DatabaseError`     | 503    | `DATABASE_ERROR`        |
| `AppError` (base)   | 500    | `INTERNAL_SERVER_ERROR` |

**Where to throw:**

- `NotFoundError` — in services, when a repository returns null
- `ForbiddenError` — in controllers, for auth/ownership checks
- `ValidationError` — via `validateRequest` middleware (Zod), or in services for business rule violations
- NEVER throw in repositories — let errors bubble up

**Route setup pattern** (ALWAYS in this order):

```
rateLimiter → authenticateToken → validateRequest({ body/query/params }) → asyncHandler(controller)
```

All controller methods must be wrapped in `asyncHandler`. ALWAYS `.bind(controller)` when passing to route.

## API response shapes

**Success** — return raw data directly, no wrapper:

```typescript
res.status(200).json(resource); // GET single
res.status(200).json(resources); // GET list
res.status(201).json(created); // POST
res.status(204).send(); // DELETE
```

**Error** — handled automatically by `errorMiddleware` when you throw an error class:

```json
{ "error": { "message": "...", "code": "...", "details": {} } }
```

NEVER manually shape error responses — throw the right error class and let the middleware handle it.

## Build order

Shared must build first: `pnpm build:shared` → then frontend/backend.

## Testing

- Shared: Vitest (`pnpm test:shared` from root)
- Backend: Jest — from root `pnpm test:backend:unit`, `NODE_ENV=test pnpm test:backend:integration`; from `packages/backend/` use `test:unit`, `test:integration`
- Frontend: Vitest + React Testing Library (`pnpm test:frontend` from root)
- MSW is installed but disabled — use real backend for frontend tests
- Test data: use `@faker-js/faker` factories (see existing `__tests__/` patterns)
- Integration tests must use the test DB — run backend with `pnpm dev:test` or set `NODE_ENV=test`; never run integration tests against the main dev DB
- NEVER run BE unit + integration tests in parallel — shared test DB state causes false failures

## Git safety

- NEVER commit directly on detected default branch (`main`, `master`, or repo default branch)
- Before any commit or PR workflow, check current branch
- If on default branch or detached HEAD, stop and ask user to switch to branch intended for PR

## Not required

- Accessibility features — explicitly excluded from project scope
