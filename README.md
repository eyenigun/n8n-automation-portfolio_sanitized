# n8n Automation Portfolio — Erhan Yenigun

A curated set of production n8n workflows I designed, built and maintained at **SolveStack.ai** (2025–2026). They cover social content automation, platform reporting, an n8n-hosted REST API, lifecycle email, AI-assisted vetting and an AI learning coach.

> **Sanitized export.** Credentials, tokens, API hosts, Slack/Favro/GA4 IDs, bucket names and email addresses have been removed or replaced with placeholders (`<REDACTED>`, `<your …>`, `user@example.com`). To run a workflow, import the JSON into n8n and attach your own credentials. Internal identifiers have also been replaced: n8n workflow, Data Table and project IDs (`<WORKFLOW_ID>`, `<DATA_TABLE_ID>`, `<N8N_ID>`), webhook IDs, the platform API base URL (`{{PLATFORM_API_BASE_URL}}`), the Slack app ID and internal channel names (`alerts-channel`, `testing-channel`). After import, re-point each Data Table, sub-workflow and Slack channel reference to your own. No clients, partner companies, startups or private individuals are referenced; examples use placeholder names.

## Stack

**Platform:** n8n Cloud (Webhooks, Schedule triggers, Data Tables, Code nodes in JavaScript, sub-workflows via Execute Workflow)
**Integrations:** Slack API (slash commands, interactive buttons, canvases, file uploads) · OpenAI (Responses API, LangChain Agent + Structured Output Parser) · Google Analytics 4 · AWS (API Gateway, Lambda, DynamoDB, S3) · Favro · Firecrawl · QuickChart · SMTP email · RSS
**Patterns:** event-driven webhooks · scheduled reporting · human-in-the-loop approval · sub-workflow orchestration · anomaly detection · REST endpoints served from n8n

## Workflows

### 01 — SoMe Robot: social content pipeline (LinkedIn / X)
A multi-workflow pipeline that turns AI news and team ideas into approved social posts. Slack is the UI and n8n Data Tables are the only storage.
| Workflow | What it does |
|---|---|
| `01a Slack Idea Capture` | `/input_idea` slash command → Webhook → parse payload → insert into ideas table → instant Slack reply |
| `01b Weekly Ideation Shortlist` | Weekly schedule → 5 RSS feeds (TechCrunch AI, MarkTechPost, Hugging Face, AI Weekly, BBC AI) merged with Slack ideas → dedupe against a topic-memory table → OpenAI scores and shortlists (capped at 7) → route by source/decision → update tables → trigger status board and canvas sub-workflows |
| `01d Slack Interaction Handler` | Receives Slack button clicks (approve / regenerate / mark published) → updates state in Data Tables → routes to draft generation, platform adaptation or publish-to-X sub-workflows |
| `01e Draft Generation` | Daily schedule → approved topics → OpenAI drafts the post and generates an image → uploads to Slack for review → cleans up temporary upload messages |

### 02 — Reporting & statistics
| Workflow | What it does |
|---|---|
| `1 Platform Statistics` | Daily → pulls startups, customers and use-case views from the platform API (10 HTTP calls) → removes test records → computes growth/country/usage stats in JS → renders charts with QuickChart → uploads to Slack → rebuilds a Slack canvas dashboard |
| `3 Google Analytics Statistics` | Daily → 7 GA4 report queries (monthly/weekly trends, country, source/medium, new users, campaigns) over rolling date ranges → computed KPIs → chart → Slack canvas |

### 03 — AWS daily report + n8n-hosted REST API
| Workflow | What it does |
|---|---|
| `4 AWS Daily Report Stats — Production` | Reads the latest AWS daily report from Slack → parses it → compares with recent history for **anomaly detection** → stores a row → posts an alert if anomalous → renders a chart → replaces the canvas |
| `5 AWS Daily Report Stats — API` | A small REST API served from n8n webhooks: `GET /reports/aws-daily/runs`, `GET …/runs/latest`, `GET …/runs/:reportDate` (with 404 handling), and `POST …/runs` to trigger a new run asynchronously (202 Accepted). Documented with an OpenAPI spec and tested with Postman. |

### 04 — Lifecycle email & campaign tooling
| Workflow | What it does |
|---|---|
| `Customers Notify` | Calculates days since sign-up and sends the right lifecycle email (welcome, 1 week, 2 weeks, 1 month) over SMTP. Also posts a daily summary to Slack, with test and production paths kept separate |
| `UTM Link Builder` | Reusable sub-workflow: looks up a program in a `utm_taxonomy` Data Table and returns a consistently tagged URL. Unknown programs return an explicit error instead of an untagged link |

### 05 — AI startup vetting
`1 Startup Vetting`: a Slack `/startup-vet` command checks the channel, extracts the URL, and checks Favro for duplicates. A LangChain **AI Agent** (OpenAI + Structured Output Parser) then validates the startup. If it's accepted, a sub-workflow scrapes the site with **Firecrawl** and creates a Favro card. Error outputs post clear Slack messages instead of failing silently.

### 06 — AI learning coach (CoDe)
`1 Interview Engine`: the webhook entry point of a Slack-based competency interview. It creates and tracks interview state in Data Tables, then branches to "ask next question" or "finish interview" sub-workflows. Later stages (not included) build an AI competency model and a learning roadmap on a Slack canvas.

## Design principles I follow
- **Native nodes first**, with Code nodes only for parsing, stats and payload building
- **Small, single-purpose workflows** chained through sub-workflows and named in run order (`01a`, `01b`, …)
- **Human approval before anything publishes**
- **Separate test and production paths**, with test records filtered out of statistics
- **Sticky notes in most workflows** documenting triggers, inputs, outputs and caveats
