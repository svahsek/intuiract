# MOSIP Nexus AI — Talking Points

> Synthesized from the internal product brief. Use as speaker notes — glance and explain in your own words.

---

## What It Is

- An **in-house AI knowledge assistant** built by our internal QA and engineering team
- Answers questions about the MOSIP platform in plain English — or any language
- Eliminates manual searching across docs, community threads, GitHub, Confluence, and source code
- Single conversational interface across all MOSIP knowledge

---

## The Problem It Solves

- Engineers waste time hunting across 5+ disconnected sources for a single answer
- Documentation, forum threads, GitHub issues, Confluence, and source code are all siloed
- Nexus AI searches all of them simultaneously and returns one grounded, cited answer

---

## What It Knows — 5 Sources, Searched Together

| # | Source | Scale |
|---|---|---|
| 1 | MOSIP Docs (docs.mosip.io/1.2.0) | 6,094 chunks across 449 pages |
| 2 | Community Forum (community.mosip.io) | 11,236 chunks |
| 3 | GitHub Issues — 86 MOSIP repositories | 2,513 chunks |
| 4 | Confluence — QT, ENGG, PMS spaces | 12,264 chunks |
| 5 | Source Code — Java, YAML, properties | ~38,000 chunks |

**Total: ~70,000 chunks of MOSIP knowledge, searched on every query**

---

## Key Features

1. **Multilingual** — Ask in Tamil, Hindi, French, Arabic, or any language; responds in the same language
2. **Source attribution** — Every answer cites the exact page, thread, issue, or Confluence page it came from
3. **Confidence scoring** — Green / Yellow / Red badge based on retrieval quality — not just a guess
4. **No hallucination guard** — Returns "not available in MOSIP sources" instead of making something up
5. **Chat memory** — Follow-up questions retain full conversation context
6. **Duplicate detection** — Surfaces similar community threads before generating a new answer
7. **REST API** — Full API so it can integrate with portals, bots, and other tooling

---

## How It Works — Simply

```
Question → check similar threads → search all 5 sources simultaneously
→ score confidence → generate grounded answer via LLM → return answer + sources + confidence badge
```

- LLM used: **Groq Llama 3.3-70B** (fast, capable)
- Embeddings: **multilingual-e5-base** — covers 100+ languages
- Vector DB: **ChromaDB**
- Interface: **Streamlit** (chat UI) + **FastAPI** (REST API)

---

## 18-Week Roadmap

| Phase | Timeline | What's Coming |
|---|---|---|
| Phase 1 — Quick Wins | Weeks 1–4 | Support Desk indexing, eSignet docs, GitHub design docs, Prompt Library |
| Phase 2 — Core & Diagnostic | Weeks 5–11 | Cross-module GraphRAG, Environment Diagnostic System, Video KT indexing |
| Phase 3 — Advanced | Weeks 12–17 | Architecture diagram generation, Confluence data governance, Confidence scoring improvements |

---

## Connection to Community (The Bigger Picture)

- Right now Nexus AI is an **internal engineering tool**
- When mature, the same knowledge base and retrieval engine can power the **community.mosip.io AI bot**
- Instead of building two separate systems, the community bot becomes a public-facing interface on top of the same curated, cited, confidence-scored knowledge
- One source of truth — for engineers internally, and for the community externally

---

## One Line to Leave With

> Our team didn't wait for a vendor to solve this. They built it themselves — grounded in citations, honest about what it doesn't know, and designed to grow into the community layer we are already building.
