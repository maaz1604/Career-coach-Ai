
# AI Career Coach — README

A full-stack Next.js application that helps professionals accelerate their careers with AI-powered tools: industry insights, resume building, cover letter generation, and interview preparation. It uses Clerk for auth, Prisma + PostgreSQL for data, Google Generative AI (Gemini) for content generation, and Inngest for scheduled background updates.

## Features

- Authentication and user management with Clerk
- Onboarding with industry and skills to personalize insights
- AI Industry Insights dashboard with:
    - Growth rate, demand level, market outlook
    - Salary ranges by role (chart)
    - Top skills, key trends, and recommended skills
- AI Resume Builder
    - Structured form → Markdown resume
    - “Improve with AI” for bullet points
    - Live Markdown preview and PDF export
    - Save resume to DB
- AI Cover Letter Generator
    - Tailored to job title, company, user profile, and JD
    - Markdown output and persistent storage
- AI Interview Preparation
    - 10-question MCQ quizzes based on industry/skills
    - Explanations, scoring, analytics, improvement tip
    - History, stats cards, performance trend
- Weekly auto-refresh of industry insights via Inngest cron


## Tech Stack

- Framework: Next.js (App Router), React, Server Actions
- UI: Tailwind CSS, shadcn/radix primitives, Recharts, @uiw/react-md-editor, Sonner
- Auth: Clerk
- AI: Google Generative AI (Gemini 1.5 Flash)
- DB/ORM: PostgreSQL + Prisma
- Scheduler: Inngest
- Misc: date-fns, html2pdf.js


## Project Structure

- actions/ — Server actions for cover letters, dashboard insights, interview, resume, and user profile
- app/ — App Router pages and routes (auth, main sections, API, lib)
    - (auth)/ — Sign-in/up (Clerk)
    - (main)/ — Dashboard, Resume, Cover Letters, Interview, Onboarding
    - api/inngest/route.js — Inngest handler
    - lib/ — app-level helpers and schemas for forms
- components/ — UI primitives and shared components (Header, ThemeProvider)
- data/ — Static site content for the landing page
- hooks/ — Custom hooks (useFetch)
- lib/ — Prisma client, utils, Clerk user bootstrap, Inngest client/functions
- prisma/ — Prisma schema and SQL migrations
- public/ — Static assets (banner images)
- Configs — Next, Tailwind, ESLint, PostCSS


## Prerequisites

- Node.js 18+
- PostgreSQL database
- API keys and secrets:
    - CLERK_SECRET_KEY, NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY
    - DATABASE_URL (PostgreSQL)
    - GEMINI_API_KEY (Google Generative AI)
    - INNGEST signing (optional for local; needed for production)


## Environment Variables

Create a .env.local file:

- DATABASE_URL=postgres://USER:PASSWORD@HOST:PORT/DB
- NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxx
- CLERK_SECRET_KEY=sk_test_xxx
- GEMINI_API_KEY=your_gemini_api_key
- (Optional) Inngest related variables depending on deployment


## Installation

1) Install dependencies

- npm install

2) Generate Prisma client

- npx prisma generate

3) Run migrations

- npx prisma migrate deploy
- Or for development:
    - npx prisma migrate dev

4) Start the dev server

- npm run dev

The app will be available at http://localhost:3000

## Database

- Prisma models: User, Assessment, Resume, CoverLetter, IndustryInsight
- Users are auto-created from Clerk on first request (lib/checkUser.js)
- One Resume per user (unique userId)
- IndustryInsight is referenced by User.industry (optional relationship)

Run Prisma Studio for quick inspection:

- npx prisma studio


## Authentication

- Clerk is initialized in app/layout.js via ClerkProvider
- Sign-in/up routes under app/(auth)
- SignedIn/SignedOut components used in the Header


## AI Integrations

- Google Generative AI (Gemini 1.5 Flash)
    - actions/cover-letter.js: generateCoverLetter
    - actions/interview.js: generateQuiz + improvement tips
    - actions/resume.js: improveWithAI for descriptions
    - actions/dashboard.js + lib/inngest/function.js: industry insights JSON

Note: Ensure GEMINI_API_KEY is set and billing/quota allows usage.

## Background Jobs (Inngest)

- Cron “Generate Industry Insights” runs weekly (Sunday midnight) to refresh insights
- Handler: app/api/inngest/route.js
- Function: lib/inngest/function.js

To run Inngest locally, follow Inngest’s Next.js setup docs. In production, ensure the route is reachable and cron is configured.

## Frontend Highlights

- Charts: Recharts (BarChart for salaries, LineChart for performance)
- Editor/Preview: @uiw/react-md-editor for cover letters and resume
- PDF Export: html2pdf.js
- UX: Sonner toasts, Radix primitives, Tailwind theming (light/dark)


## Key Flows

- Onboarding
    - app/(main)/onboarding: select industry and specialization; profile saved via actions/user.updateUser
    - Generates IndustryInsight if not present using AI
- Dashboard
    - app/(main)/dashboard: fetches IndustryInsight and renders cards, chart, trends, skills
- Resume
    - Build from structured form; live Markdown; AI “Improve” for descriptions; persist via actions/resume.saveResume; export PDF
- Cover Letters
    - Create from company, role, JD; AI generates markdown; list/manage letters
- Interview
    - Generate industry/skill-based quiz; answer flow with explanations; save assessment; show results, stats, performance chart


## Commands

- Development: npm run dev
- Build: npm run build
- Start: npm start
- Lint: npm run lint
- Prisma Studio: npx prisma studio
- Migrate dev: npx prisma migrate dev
- Migrate deploy: npx prisma migrate deploy


## Tailwind \& Styling

- Tailwind CSS via @import in app/globals.css
- Theming variables and dark mode supported
- shadcn-style UI primitives in components/ui


## Important Notes and Gotchas

- Ensure Clerk domain and publishable key match the deployment URL
- actions/user.updateUser currently returns result.user but the transaction returns { updatedUser, industryInsight } — consider returning the correct object shape for client checks
- Industry linkage: User.industry references IndustryInsight.industry (string key); onboarding composes industry as `${industry}-${subIndustry.toLowerCase().replace(/ /g, "-")}` — ensure consistency with insight keys
- AI outputs are parsed from model responses; code strips code fences before JSON.parse — handle errors gracefully
- PDF generation renders a hidden Markdown block for clean export styling


## Deployment

- Recommended platforms: Vercel (Next.js), Neon/Railway/Supabase/Postgres for DB
- Set environment variables on the platform
- Run migrations on deploy (or CI/CD)
- Ensure Clerk OAuth redirect URLs and Inngest routes are configured


## Security

- Server Actions guard with Clerk auth; throws Unauthorized if no session
- Prisma client reused across dev hot reloads
- Do not commit .env files or secrets


## Acknowledgements

- Clerk for auth
- Prisma for ORM
- Google Generative AI for content generation
- Inngest for background jobs
- shadcn/radix for UI building blocks
- Recharts and @uiw/react-md-editor for visualization and editing
