# PROJECT CONTEXT: MCP Finance Analyzer

## What This Project Is
A personal finance analysis tool built as an MCP (Model Context Protocol) server,
allowing AI assistants (Claude Desktop, and later a LangGraph agent) to query,
categorize, and analyze bank transaction data. This is a portfolio/learning
project demonstrating MCP server development with realistic architecture.

## Core Purpose
- Import bank transaction CSVs
- Auto-categorize transactions using rule-based keyword matching
- Detect spending anomalies statistically
- Generate reports on demand via natural language (through Claude) or
  automated schedules (through LangGraph)

## Tech Stack
- **Language:** Python 3.11+
- **MCP SDK:** `mcp` (official Python SDK)
- **Database:** SQLite (via SQLAlchemy ORM) — local file at `data/finance.db`
- **CSV Parsing:** pandas
- **Synthetic Data:** Faker (for demo/testing, no real financial data used)
- **Anomaly Detection:** Statistical (Z-score) initially; may upgrade to
  scikit-learn IsolationForest later
- **Testing:** pytest
- **Phase 2 (remote access):** HTTP/SSE transport, deployed to [Railway/Fly.io/TBD]
- **Phase 3 (automation):** LangGraph for scheduled/conditional workflows

## Architecture Decisions (Important Context!)

1. **No Plaid or bank APIs** — deliberately using CSV import to avoid
   external API dependencies and mirror how many real fintech MVPs start.

2. **stdio transport in Phase 1** — MCP server runs as a local subprocess,
   spawned by Claude Desktop. No network involved. Database is local-only
   at this stage — this is intentional, not a limitation to "fix" yet.

3. **SQLite, not PostgreSQL (for now)** — appropriate for single-user,
   local-first Phase 1. Will reconsider if/when Phase 2 (remote, multi-device)
   is implemented, since SQLite has concurrency limitations for
   simultaneous remote users.

4. **MCP server and LangGraph agent are SEPARATE clients hitting the
   same MCP server** — LangGraph is not "inside" the MCP server. It's an
   alternative orchestration layer for automated workflows (see `agent/`
   folder), while Claude Desktop remains the ad-hoc conversational client.

5. **Categorization is rule-based first** — simple keyword matching against
   a `categories` table. Explicitly NOT using ML/embeddings yet — that's a
   documented stretch goal, not a current requirement.

## Current Phase Status
> **UPDATE THIS SECTION AS YOU PROGRESS**

- [ ] Phase 1: Local MCP server (stdio) — IN PROGRESS
  - [x] Database models defined
  - [ ] CSV parser (Chase format only so far)
  - [ ] Core tools: get_spending_by_category, import_bank_csv
  - [ ] Anomaly detection tool
  - [ ] Tested end-to-end with Claude Desktop
- [ ] Phase 2: Remote access (HTTP/SSE + auth) — NOT STARTED
- [ ] Phase 3: LangGraph automation agent — NOT STARTED

## File/Folder Conventions
- All MCP tool definitions live in `src/tools/`, grouped by domain
  (spending, import, budget) — NOT all in one giant file
- Business logic (categorization, anomaly detection, forecasting) lives in
  `src/services/` and is UNIT TESTABLE independent of MCP — tools in
  `src/tools/` should be thin wrappers calling into `services/`
- Bank-specific CSV parsing logic goes in `src/importers/bank_formats/`,
  each inheriting from `base.py` — new bank support = new file, no
  modification of existing parsers

## What NOT To Do (Constraints)
- Do NOT integrate Plaid or any live bank API — CSV import is a
  deliberate, permanent design choice for this project
- Do NOT add authentication complexity until Phase 2 begins
- Do NOT use real personal financial data in commits, tests, or examples —
  use Faker-generated synthetic data or `.example` sample files only
- Do NOT put business logic directly inside `@server.tool()` functions —
  keep tools thin, logic in `services/`

## Key Terminology (for consistency)
- "Transaction" = a single bank line item (not "expense" or "entry")
- "Category" = spending category (food, transport, etc.) — NOT "tag"
- "Account" = a bank account/card (checking, credit) — one user may have many

## Next Immediate Task
> UPDATE THIS EVERY SESSION so the LLM knows exactly where to pick up

Currently working on: [e.g., "Building the Chase CSV parser in
src/importers/bank_formats/chase.py — need to handle their specific date
format (MM/DD/YYYY) and negative amounts for debits"]