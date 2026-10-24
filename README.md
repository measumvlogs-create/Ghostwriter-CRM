#Ghostwriter CRM
# GhostWriter CRM

**Talk to your CRM. Let AI handle the rest.**

An AI-powered, multi-tenant CRM built for the Fastn hackathon. Record or paste meeting notes, and AI extracts contacts, summaries, deals and follow-up tasks for review before anything is saved.

> **Status:** hackathon prototype. The demo UI uses simulated data. The Supabase schema and Edge Functions are included but must be configured and deployed, and Fastn endpoint URLs must be filled in from your own Fastn workspace.

## Features

- AI Meeting Capture: paste notes, review extracted fields, then save (voice input is planned)
- Contacts, deals pipeline (7 stages), tasks and follow-ups, AI email drafts
- Five connector cards: Google Calendar, Gmail, Slack, Google Drive, HubSpot
- Fastn embed widget, one token per tenant
- Multi-tenant isolation using Postgres row-level security
- Two-tenant test panel in the demo UI

## Project structure

```
frontend/public/index.html          Demo UI
supabase/migrations/001_schema.sql  Schema + RLS (tenant_id on every table)
supabase/functions/extract-meeting  LLM extraction + duplicate check
supabase/functions/fastn-embed-token  Mints per-tenant Fastn embed token
supabase/functions/run-workflow     Runs a Fastn workflow for the caller's tenant
supabase/seed_two_tenants.sql       Creates two test tenants
.env.example                        Required secrets
```

## Setup

### Prerequisites
- Node.js and npm
- A Supabase project
- A Fastn workspace with the connector configured
- An Anthropic API key (or another LLM key, edit `extract-meeting`)

### 1. Install and link the Supabase CLI

```bash
npm i -g supabase
supabase login
supabase link --project-ref YOUR_PROJECT_REF
```

### 2. Create the database schema

```bash
supabase db push
```

Or paste `supabase/migrations/001_schema.sql` into the Supabase SQL Editor and run it.

### 3. Configure secrets

```bash
cp .env.example .env
# fill in the values, then:
supabase secrets set --env-file .env
```

| Variable | Purpose |
|---|---|
| `ANTHROPIC_API_KEY` | LLM extraction |
| `LLM_MODEL` | Model name (default `claude-sonnet-4-6`) |
| `FASTN_API_KEY` | Server-side Fastn auth |
| `FASTN_EMBED_TOKEN_URL` | Fastn endpoint that issues embed tokens (from your Fastn workspace) |
| `FASTN_WORKFLOW_URL` | Fastn workflow execute endpoint (from your Fastn workspace) |

`SUPABASE_URL`, `SUPABASE_ANON_KEY` and `SUPABASE_SERVICE_ROLE_KEY` are provided to Edge Functions automatically. If `secrets set` rejects them, remove them from `.env`.

Never commit `.env` or expose these keys in frontend code.

### 4. Deploy the Edge Functions

```bash
supabase functions deploy extract-meeting
supabase functions deploy fastn-embed-token
supabase functions deploy run-workflow
```

### 5. Create two test tenants

1. In Supabase Auth, create two users and copy their UUIDs.
2. Put the UUIDs into `supabase/seed_two_tenants.sql`.
3. Run the file in the SQL Editor.

### 6. Run the demo UI

```bash
cd frontend
npm run dev
```

## Two-tenant isolation test

1. Sign in as user A and run the `run-workflow` function with the same input.
2. Sign in as user B and do the same.
3. Query `contacts`, `deals` and `workflow_runs` as each user. Each user should see only their own tenant's rows.

Isolation is enforced by RLS policies using `current_tenant()` and by giving each tenant its own Fastn tenant and embed token.

## Security notes

- API keys and OAuth secrets stay on the server.
- AI output is shown for review before saving.
- Emails and external actions require user confirmation and a genuinely connected provider.
- Simulated demo actions are labeled as such in the UI.

## Not yet implemented

- Frontend wired to Supabase and the Edge Functions (the demo page uses in-memory data)
- React/TypeScript/Tailwind/shadcn rewrite
- Real OAuth flows for the five connectors
- Voice recording and speech-to-text
- Analytics, Settings, CSV import/export

