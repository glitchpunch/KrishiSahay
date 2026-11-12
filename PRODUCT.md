# KrishiSahay
<!-- impeccable:product-schema 1 -->

## Platform
web

## Users
Farmers who need to understand government agriculture schemes, eligibility conditions, benefits, application steps, and official notices in Hindi, Punjabi, or English. Hackathon judges need a complete, inspectable demonstration.

## Product Purpose
Help a farmer ask a practical question about a government scheme and receive a plain-language answer from official documents, with source citations and clear uncertainty. Process source text locally, send only approved excerpts through Prune, and bound/reconcile billable AI attempts.

## Capabilities and Constraints
The Scheme Guide combines browser-local BM25 retrieval across up to eight sources, field selection, bounded passages, and page/paragraph citations with farmer-focused questions about benefits, eligibility, dates, and required actions. Optional generated answers send only an explicitly approved question, answer language, and selected redacted excerpts through Prune; source-ID validity is checked independently. Retrieval copies remain in browser memory. Import supports PDF text layers, Word DOCX text/tables, UTF-8/UTF-16 text, CSV/TSV, JSON/JSONL, and XLSX; scanned PDFs need OCR, which is not implemented. Generic dataset analysis and an older order workflow remain in the codebase but are not the main farmer-facing route. Import limits are 5 MB / 5,000 records / 200 columns.
KrishiSahay supports typed questions and optional browser speech recognition, with Hindi/Punjabi/English answer selection. The browser's speech service may process microphone audio externally; the transcript is shown for review. Search uses local BM25 lexical matching and a small Hindi/Punjabi agriculture glossary; semantic translation is not implemented, and same-language documents give the best results. Selected passages, not complete files, may be shared through Prune after explicit approval. The answer and citation IDs are checked, but neither factual grounding nor current official policy is guaranteed. Demo snippets are dated reference snapshots; farmers should check the linked official portal before acting.
GhostLoop makes the existing spend-safety controls visible: approval, durable idempotency admission, daily attempt limits, no automatic retries, and an account-scoped unresolved-request ledger with manual reconciliation after receipt review.
Existing vanilla HTML/CSS/JavaScript, Express, Supabase Auth, Drizzle/PostgreSQL and Prune are retained. Every remote generative call goes through Prune. Customer messaging remains draft/export only. Local means the backend runs on the user's machine; remote hosting does not make uploaded data local. GPU availability must not be described as acceleration.

## Brand Commitments
Use the KrishiSahay name, a calm dark interface, readable multilingual text, and honest product claims. Explain scheme details plainly without implying official government affiliation or guaranteeing eligibility.

## Product Principles
- A proposed plan never grants its own data access.
- Approve exact minimized data before remote processing.
- Reuse instructions with current records and current validation.
- Prefer deterministic code for dates, money, filtering and templates.
- Show actual outcomes, provider metadata and limitations.

## Evidence on Hand
User confirmed working Prune/Supabase setup and NVIDIA GPU availability. This session has not yet independently verified private cloud configuration. Source draft is baseline commit 2bd6779. No measured savings have been supplied.
