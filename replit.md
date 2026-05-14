# Resume AI

A SaaS Resume AI system that helps users upload their resume, compare it against a job description, get AI-powered skill gap analysis, and download a polished resume in multiple professional templates.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/resume-ai run dev` — run the frontend (port 18731)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string
- Required env: `AI_INTEGRATIONS_OPENAI_BASE_URL`, `AI_INTEGRATIONS_OPENAI_API_KEY` — Replit AI Integrations for OpenAI

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite, TailwindCSS, Wouter, TanStack Query
- API: Express 5
- DB: PostgreSQL + Drizzle ORM
- AI: OpenAI via Replit AI Integrations (gpt-5.1)
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `lib/api-spec/openapi.yaml` — OpenAPI 3.1 spec (source of truth)
- `lib/db/src/schema/resumes.ts` — Resume table schema
- `lib/db/src/schema/resume_versions.ts` — Resume version history schema
- `artifacts/api-server/src/routes/resumes.ts` — Resume CRUD + upload + download routes
- `artifacts/api-server/src/routes/analysis.ts` — AI analysis routes (compare, enhance, ATS)
- `artifacts/api-server/src/routes/templates.ts` — Resume template list
- `artifacts/resume-ai/src/pages/` — Frontend pages (home, analyze, enhance, templates, resumes)

## Architecture decisions

- Resume file extraction uses mammoth for DOCX and GPT-4o vision for PDF — no native PDF parsing dependency needed
- All AI calls go through Replit AI Integrations (no user API key required)
- Resume download generates HTML server-side which the browser can print to PDF — avoids complex PDF generation libraries
- Analysis flow uses React context (AnalysisFlowContext) to pass state between the multi-step wizard pages
- File upload bypasses the generated hooks (multer multipart) — uses raw fetch with FormData

## Product

- Upload resume (PDF/DOCX) with drag-and-drop
- Paste job description and get AI-powered match percentage + missing skills list
- Select individual skills or add all missing skills at once
- AI generates domain-relevant context for each skill
- Preview enhanced resume section by section with inline editing
- Choose from 3 templates (Modern, Classic, ATS-Friendly) and download as PDF
- Dashboard with resume history, versions, and match stats
- ATS optimization suggestions with priority levels

## Gotchas

- Always run codegen after changing openapi.yaml: `pnpm --filter @workspace/api-spec run codegen`
- The `/resumes/stats` route must be registered before `/resumes/:id` in Express
- PDF download returns HTML — browser window.print() triggers the PDF save dialog
- DB push is safe to run repeatedly: `pnpm --filter @workspace/db run push`
