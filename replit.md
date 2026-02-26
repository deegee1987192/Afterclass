# AfterClass - Professional Network for Sports & Marketing Students

## Overview

AfterClass is a community-driven professional networking web application designed for sports and marketing students, alumni, and industry professionals. The platform features job opportunity listings, alumni/industry spotlight stories, newsletter signup, and a protected admin dashboard for content management. The app includes an AI job parsing endpoint (currently returning placeholder data, designed for future OpenAI integration).

**Key pages:** Homepage, Opportunities, Spotlights, Give Back, and Admin Dashboard.

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend
- **Framework:** React 18 with TypeScript
- **Routing:** Wouter (lightweight client-side routing)
- **State Management:** TanStack React Query for server state (caching, mutations, invalidation)
- **UI Components:** shadcn/ui (new-york style) built on Radix UI primitives
- **Styling:** Tailwind CSS with CSS variables for theming. Warm, approachable design with teal primary color, coral secondary, and soft cream background. Uses DM Sans for body text and Outfit for display/headings.
- **Forms:** react-hook-form with Zod validation via @hookform/resolvers
- **Animations:** Framer Motion for page transitions and hover effects
- **Build Tool:** Vite with React plugin

### Backend
- **Runtime:** Node.js with Express
- **Language:** TypeScript (compiled with tsx for dev, esbuild for production)
- **API Pattern:** RESTful JSON API under `/api/*` prefix. Routes are defined in `shared/routes.ts` with Zod schemas for input validation and response types.
- **Authentication:** Replit Auth (OpenID Connect). Sessions stored in PostgreSQL via `connect-pg-simple`. Protected routes use `isAuthenticated` middleware.
- **Key API endpoints:**
  - `GET/POST /api/jobs` — List and create jobs
  - `GET/PUT/DELETE /api/jobs/:id` — Individual job operations
  - `GET/POST /api/spotlights` — List and create spotlights
  - `GET/PUT/DELETE /api/spotlights/:id` — Individual spotlight operations
  - `POST /api/subscribers` — Newsletter signup
  - `POST /api/parse-job` — AI job description parser (placeholder, ready for OpenAI integration)
  - `GET /api/auth/user` — Current authenticated user

### Shared Code (`shared/`)
- **Schema:** `shared/schema.ts` defines all database tables and Zod validation schemas using Drizzle ORM's `pgTable` and `drizzle-zod`
- **Routes:** `shared/routes.ts` defines a typed API contract with paths, methods, input schemas, and response schemas — shared between client and server
- **Auth Models:** `shared/models/auth.ts` defines the users and sessions tables required by Replit Auth

### Database
- **Database:** PostgreSQL (required, provisioned via Replit)
- **ORM:** Drizzle ORM with `drizzle-zod` for schema-to-validation bridging
- **Schema push:** `npm run db:push` uses `drizzle-kit push` to sync schema to database
- **Migrations directory:** `./migrations`
- **Tables:**
  - `jobs` — id, title, company, location, salary, job_type, description, link, featured, created_at
  - `spotlights` — id, name, title, bio, photo_url, content, featured, archived, created_at
  - `subscribers` — id, email (unique), role, created_at
  - `sessions` — sid, sess (jsonb), expire (required for Replit Auth)
  - `users` — id, email, first_name, last_name, profile_image_url, created_at, updated_at (required for Replit Auth)

### Storage Layer
- `server/storage.ts` implements `IStorage` interface with `DatabaseStorage` class
- All database operations go through this storage layer (not direct DB calls in routes)

### Build Process
- **Dev:** `tsx server/index.ts` with Vite dev server middleware for HMR
- **Production:** `script/build.ts` runs Vite build for client, then esbuild for server bundling. Output goes to `dist/` with client assets in `dist/public/`
- **Server serves SPA:** In production, Express serves static files from `dist/public` with fallback to `index.html` for client-side routing

### Path Aliases
- `@/*` → `./client/src/*`
- `@shared/*` → `./shared/*`
- `@assets` → `./attached_assets/`

## External Dependencies

- **PostgreSQL** — Primary database, connection via `DATABASE_URL` environment variable
- **Replit Auth (OpenID Connect)** — Authentication provider using `ISSUER_URL` and `REPL_ID` environment variables
- **Express Session** — Requires `SESSION_SECRET` environment variable
- **OpenAI API** — Listed in build allowlist but not yet actively integrated. The `/api/parse-job` endpoint is a placeholder ready for future OpenAI connection
- **Unsplash** — Used for placeholder profile images in spotlight cards
- **Google Fonts** — DM Sans, Outfit, and other fonts loaded via CDN