# JobPilot AI — Cloud Web MVP

Next.js 15 + React 19 frontend, FastAPI backend, PostgreSQL/SQLite database, Greenhouse/Lever public job feeds, keyword matching, optional OpenAI drafting, and application tracking.

## Important scope

This release **does not auto-submit job applications**. It saves applications, generates drafts, opens the official application URL and allows manual status updates. Settings can mark a host *auto-eligible* but do not submit. Real submission needs a site-specific tested Playwright worker, verified screening responses, consent handling and explicit authorization. No CAPTCHA bypass or automated assessment completion.

This is a **single-user admin MVP**, not a multi-tenant SaaS. Do not deploy publicly with default passwords. Uploaded CV files, background job scheduling, and fully automated browser workers are not implemented.

## Local development (Windows PowerShell)

Requirements: Python 3.11+, Node 20+, npm. In two terminals:

```powershell
cd backend
py -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt
$env:JWT_SECRET='local-development-secret-change-later'
$env:ADMIN_PASSWORD='local-dev-password'
.venv\Scripts\python.exe -m uvicorn app.main:app --reload
```

```powershell
cd frontend
npm install
npm run dev
```

Open http://localhost:3000 and log in with the `ADMIN_PASSWORD`. The backend uses SQLite locally if `DATABASE_URL` is unset. API documentation: http://localhost:8000/docs.

## Docker Compose

Copy `.env.example` to `.env`, replace passwords and secret with strong random values, then:

```bash
docker compose up --build -d
```

Web: http://localhost:3000; API: http://localhost:8000. PostgreSQL data is stored in a named Docker volume. Use a reverse proxy with HTTPS for an internet-facing deployment.

## Deploy to Vercel + Railway / Render + managed PostgreSQL

1. Push this folder to your own private GitHub repository (do not commit `.env`).
2. Provision a managed PostgreSQL database and a persistent FastAPI service (Railway or Render). Use `backend` as the service root; Dockerfile is provided.
3. Set `DATABASE_URL`, `JWT_SECRET` (32+ random characters), `ADMIN_PASSWORD` (12+ characters), `ENVIRONMENT=production`, `FRONTEND_ORIGIN=https://your-frontend-domain`, and optionally `OPENAI_API_KEY` on the backend. Use your database provider's required SSL settings.
4. Deploy `frontend` as a Next.js project on Vercel, set **build-time** `NEXT_PUBLIC_API_URL=https://your-api-domain`, then redeploy. This value is bundled into browser JS at build time; changing it requires rebuilding.
5. Configure HTTPS for both sites, verify `/health`, then sign in and import jobs.

For Docker-based cloud deployments, build the frontend with `NEXT_PUBLIC_API_URL` set during image build (not just runtime). The included compose file defaults to localhost for local Docker usage.

## Data/security notes

- One shared admin account; JWT held in browser sessionStorage. For a multi-user production SaaS, replace this with proper user accounts, per-user data isolation, refresh token handling, rate limits and audit logs.
- Only the admin password is used to log in; profiles are private to the admin. Do not enter secrets in candidate profile fields.
- Company tokens are restricted to alphanumeric, `_`, `-` and public Greenhouse/Lever endpoints to avoid arbitrary URL fetching.
- Job descriptions are untrusted; the AI prompt explicitly forbids following embedded instructions and fabricating credentials. Review all drafts.
- The job match percentage is a simple keyword heuristic, not a calibrated likelihood of receiving an offer.
- Application statuses are user-maintained, not verified against ATS systems.
- Database tables are created automatically on startup. For production schema changes, add Alembic migrations.
