# KrishiSahay

**A voice-first farmer scheme guide.** Ask about agriculture benefits, eligibility, deadlines, and application steps in Hindi, Punjabi, or English. Type or use browser speech recognition, review the transcript, search official documents locally, inspect citations, and approve only selected excerpts before an AI answer is sent through Prune.

**Quick start:** Run the local demo, open **Scheme guide**, choose **Load scheme examples**, select an answer language, then ask an eligibility, benefit, application, or notice question. Sample excerpts cover PM-KISAN and PMFBY from their official portals. They are snapshots, not live updates; confirm current terms and individual eligibility with the linked government sources.

**GhostLoop spend guard:** **Compute & receipts** shows unresolved request reservations and recent AI calls. Billable requests require approval, use durable idempotency admission, count against the daily attempt ceiling, and are never automatically retried. For an unknown outcome, check the exact request in Prune first; after one minute, a signed-in user can acknowledge that review in the UI. Acknowledging records the decision and does not retry or refund the call.

Uploaded PDF, DOCX, and text files are parsed in the browser. Retrieval uses lexical BM25 with a small Hindi/Punjabi agriculture glossary and works best when the question uses terms from the source's language; semantic translation is not implemented. Browser speech recognition can send audio to the browser vendor's service; the transcript is shown for review. The full document is never sent to the model, but selected excerpts may contain private facts and require explicit approval.

Every remote generative request uses the configured Prune gateway. Local demo mode disables remote AI, so it demonstrates document loading, local search, citations, cost guardrails, and no-retry behavior; a signed-in cloud environment is required for Prune-generated answers.

## Run the complete local demo

Requires Node.js 22+ (24 recommended):

```powershell
npm ci
npm run demo
```

Open **http://127.0.0.1:8787**. Port 8787 avoids the unrelated service occupying 8000 on the development machine. Local mode is explicit, binds to loopback, stores aggregate history in `.data/`, and **disables all external AI calls**.

1. Click **Load scheme examples** to add curated official PM-KISAN and PMFBY excerpts to browser memory.
2. Choose Hindi, Punjabi, or English. Ask by voice or type a question. Speech processing may occur through your browser's provider.
3. Search in the source language, inspect the supporting passages and citations, and select only the excerpts needed.
4. In a configured cloud workspace, review and approve the exact question, language, and selected excerpts before the Prune call. Local mode never simulates an AI answer.
5. Open **Compute & receipts** to inspect the no-auto-retry policy and unresolved call ledger.

## Connect the existing Supabase + Prune setup

Copy `.env.example` to `.env` and configure it locally. Never commit `.env`, API keys or database passwords. Set `LOCAL_MODE=false`, your Supabase URL/public anon key, the database connection string, `PRUNE_API_KEY`, and a currently supported `PRUNE_MODEL`. Store the provider key in Prune's vault, not this project. Start with `npm start`.

The production mode uses Supabase Auth and owner-scoped PostgreSQL state through Drizzle. The startup database role needs table creation/alter/revoke permission; this prototype creates `public.indicedge_state`, enables RLS and revokes direct table access from Supabase's `anon`/`authenticated` roles. A separate migration role and tested least-privilege runtime role are a later production milestone. Existing `indicedge_runs` data is not deleted or migrated; the new history UI reads the new aggregate ledger.

Keep frontend and API on the **same origin**. For split hosting use an HTTPS reverse proxy, not browser-selected API hosts. Configure `FRONTEND_ORIGINS` to match the actual origin. The CSP allows the pinned Supabase browser SDK from esm.sh and HTTPS Supabase project hosts. Custom auth domains need an explicit reviewed CSP change.

Optional cloud features:

- When local intent matching cannot understand a request, **Preview AI interpretation** offers a sanitized instruction and fixed JSON schema. No source rows accompany it. Review the whole request: pattern filtering cannot identify every confidential fact.
- After a run, **Explain with AI** offers an aggregate-only request: match count, outstanding paise, threshold and language. These aggregates can still be sensitive.
- Both require exact-payload approval, authentication and Prune configuration. Every call uses `https://api.withprune.com/v1`, bounded output/time, disabled SDK retries, a request ID and a Prune run-budget header. Configure and verify account-level spend caps in Prune separately.
- `MAX_DAILY_CALLS` caps admitted attempts per account per UTC day, not local electricity/hosting costs. Failed attempts count. The application does not claim that an estimated model price is an actual charge.
- Actual returned `prune_metadata` and usage appear in **Run history**. Signature presence is not signature verification. Use [Prune's verifier](https://www.withprune.com/verify) for a real receipt.

[Official Prune setup](https://www.withprune.com/docs/hackathon) · [Gateway metadata and estimates](https://www.withprune.com/docs/how-it-works)

## Source format and privacy boundary

CSV headers or JSON object keys:

```csv
order_id,customer_name,contact,due_date,amount,status
ORD-2001,Aarav Mehta,aarav@example.test,2026-10-20,2450.50,pending
```

For order reminders, required status values are `pending`, `paid`, `cancelled`. Dates are strict `YYYY-MM-DD`; amounts are non-negative INR decimals with at most two decimal places. Maximum 2,000 orders. Duplicate IDs, unknown statuses and malformed order values block reminder execution, while the dataset stays loaded for inspection and other operations. Generic import allows 5 MB, 5,000 records and 200 columns. Overdue comparisons use calendar days relative to the editable IST evaluation date; the threshold is inclusive.

Source rows and drafts remain in browser memory. They are cleared by reload, clear-source or sign-out. Extra source fields such as bank details and unrelated notes are excluded from the execution representation. Audit requests send aggregate totals, validated rules, date and a source hash. This hash is not encryption or independent proof. In hosted mode these audit records reach your backend; that is not the same as the whole system being local.

Saved workflow names and plans are retained (up to 50). History retains up to 100 aggregate events. Finished AI replay records, including model output and metadata, are retained for up to seven days and pruned on subsequent AI admission; pending unknown outcomes remain until reconciled. No raw instruction is intentionally stored by the new workflow endpoints. The legacy `/api/ask` route remains for compatibility and **does not provide task-aware minimization**; do not submit private documents there. Its model output is also retained for replay.

## Idempotency and unknown outcomes

Billable endpoints require a UUID `Idempotency-Key`. The backend reserves it durably **before** calling Prune. A repeated completed request returns the recorded result; mismatched payloads or failed/pending keys are rejected. If the provider succeeds and saving fails, the response includes the actual result and a persistence warning. Never automatically retry an unknown outcome.

Inspect `GET /api/requests` with your authenticated session to locate pending requests. After checking the corresponding Prune request/receipt and waiting at least one minute, explicitly acknowledge it:

```text
POST /api/requests/<request-id>/reconcile
{"receiptReviewed":true}
```

This releases the pending admission lock, records the acknowledgement, and does not refund, retry, or fabricate a receipt. Ledger limits and daily attempt counters remain in force.

## Docker

```powershell
docker compose up -d --build
```

The standard Compose setup requires the real cloud environment. `HOST=0.0.0.0` inside the container is required; the published host port remains loopback-only. The Dockerfile uses a lockfile, `npm ci`, and a non-root user. Frontend assets and locally licensed fonts/icons are included in the image.

## Verify

```powershell
npm test
npx playwright install chromium
npm run test:browser
# Optional: TEST_DATABASE_URL must point at a disposable PostgreSQL database
npm run test:postgres
npm audit --omit=dev
docker build -t indicedge:local .
```

Tests cover intent rejection, dates, integer money, confidential marker exclusion, fresh-data reuse, source parsing/export safety, owner isolation, request replay/quota and API errors. Browser tests exercise desktop and narrow screens, local-only source processing, keyboard focus and axe accessibility checks. Browser tests use a separate local store. Set `CAPTURE_UI=true` to capture screenshots under the gitignored `.impeccable/review/` directory. The checked-in CI runs unit/API/browser tests, the dependency audit and Docker build.

## Current scope

Implemented: bounded intent, editable rules, field minimization, local order execution, bilingual editable/exportable drafts, saved workflows, aggregate run evidence, approved Prune integration, a source-cited Notice Desk, GhostLoop request-ledger visibility and reconciliation, black responsive UI and reduced-motion support.

**Not implemented:** GPU execution, OCR, Sarvam speech, external message sending or a measured full-context AI baseline. GPU availability is real telemetry only. The user confirmed existing cloud credentials; live Supabase/Prune integration is not claimed as verified by this checkout's local tests.

See [REVIEW.md](REVIEW.md) for concrete draft findings, [PLAN.md](PLAN.md) for milestone status, and [DESIGN.md](DESIGN.md) for the UI/motion system. Geist and Phosphor assets are self-hosted with license files in `assets/`.
