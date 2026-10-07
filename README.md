# AI Data Analyst Assistant (Olist E-Commerce)

> Dashboards answer the questions someone predicted. This assistant answers the ones nobody predicted.

## The problem

Olist is a Brazilian marketplace connecting thousands of small sellers to online shoppers.
Its analytics team maintains solid dashboards, but everything outside them lands as ad-hoc
requests from operations leads, category managers, and the seller-success team. Some are
quantitative ("what's our on-time delivery rate by state this year?"). Others are contextual
("what's the difference between `customer_id` and `customer_unique_id`?", "do canceled
orders count toward GMV?").

This assistant answers both kinds. A LangGraph agent reads each question and routes it:

- a **text-to-SQL path** queries the order database directly;
- a **RAG path** retrieves from the team's data dictionary, metric definitions, and analyst notes;
- **both** handle questions that need numbers and context together;
- a **clarifying question** comes back when the request is ambiguous.

It's Copilot-style conversational BI, built from the ground up so every layer (retrieval,
query generation, routing, evaluation, guardrails) is visible and explainable.

## Architecture (planned; diagram generated in Stage 5)

```
User question
   │
   ▼
LangGraph router ──► SQL path  (schema-aware text-to-SQL → validated → read-only SQLite)
   │             ──► RAG path  (docs/corpus → embeddings → Chroma → retrieval)
   │             ──► hybrid    (both, merged)
   │             ──► clarify   (ask a follow-up)
   ▼
Synthesizer → grounded answer (numbers + definitions + sources)
```

Model layer: Claude via the Anthropic API locally → Claude on Microsoft Foundry in the
cloud stage, plus an AWS Bedrock design comparison.

<!-- STATUS:START -->
## Status

| Stage | Topic | Status |
|---|---|---|
| 0 | Prerequisites + setup | 🔄 |
| 1 | GenAI foundations | ⬜ |
| 2 | RAG core | ⬜ |
| 3 | Text-to-SQL + tool calling | ⬜ |
| 4 | LangChain v1 refactor + memory | ⬜ |
| 5 | LangGraph router agent | ⬜ |
| 6 | Evaluation, guardrails, tracing | ⬜ |
| 7 | MCP server | ⬜ |
| 8 | Microsoft Foundry + ship | ⬜ |
<!-- STATUS:END -->

## Setup

Requires [uv](https://docs.astral.sh/uv/) (it installs the right Python for you).

```bash
uv sync                      # creates .venv from uv.lock
cp .env.example .env         # Windows: copy .env.example .env, then add your key
uv run python scripts/stage0_check.py
```

Data: download the "Brazilian E-Commerce Public Dataset by Olist" from Kaggle and extract
all 9 CSVs into `data/raw/`. The data is not included in this repo.

## Repo layout

```
src/assistant/   reusable package (rag, sql, graph, guardrails)
scripts/         one runnable demo per stage
tests/           pytest (data/LLM-dependent tests are skipped in CI)
eval/            test set + eval reports
docs/corpus/     RAG corpus: data dictionary, business context, analyst notes
docs/            DECISIONS.md (architecture decisions), STORY.md
data/            local only (gitignored)
```
