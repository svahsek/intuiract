# AI on community.mosip.io — Options and Honest Reality

> Context: community.mosip.io runs on Discourse. Thread summarization is already enabled for moderators. This note covers what we can do next — ingesting knowledge from official documentation and enabling auto-responses — based on what Discourse AI actually supports.

---

## What we already have

**Thread summarization** — built into Discourse AI. Moderators get a "Summarize" button on long threads to quickly get the gist. No additional setup needed.

---

## Option 1: AI Persona as a Knowledge Bot

Discourse AI lets you create a custom **AI Persona** — a named bot with a system prompt you write. You configure it to act as a MOSIP support assistant, attach a knowledge base via file uploads, and deploy it in the community.

### What it can do natively
- Respond in public topics, DMs, or chat channels
- Search the forum for prior answers before responding
- Browse live URLs on demand via a built-in Web Browser tool
- Read GitHub files, file content, and PR diffs — requires a GitHub API token
- Run Google Search — requires a Google API key

### Knowledge ingestion — source by source

| Source | Method | Reality |
|---|---|---|
| docs.mosip.io / docs.esignet.io / docs.inji.io | Upload as `.txt` or `.md` files | Manual export required — no live crawler |
| GitHub (repos, docs, markdown files) | Native GitHub tool (API token needed) OR export `.md` and upload | Works natively for GitHub content |
| Confluence | Export pages to `.md`, upload | Manual — no direct connector |
| YouTube | Download auto-captions as `.txt`, upload | Manual — no native integration |

### Key limitation
File upload currently accepts only `.txt` and `.md`. PDF and DOCX support is on Discourse's roadmap but not shipped. External documentation sites are not crawled automatically — you export once, upload, and must re-upload when docs are updated.

---

## Option 2: AI Auto Responder

Requires both the `discourse-ai` and `discourse-automation` plugins (both maintained by Discourse). You configure a trigger — such as a new post in a specific category — and the AI persona responds automatically.

### The responsible AI gate
Responses can be configured to post as a **whisper first** — visible only to moderators, who review and approve before it goes public. This is a human-in-the-loop mechanism and the right way to start.

### Configuration
- Trigger: new post created, or stalled topic with no activity for N days
- Scope: can target specific categories (e.g. only the Support category)
- Response mode: whisper (moderated) or direct reply (fully automated)

### Limitations
- Rate limited: 60 AI calls per minute globally, 2 per post
- LLM API costs increase at scale — monitor usage
- Accuracy is only as good as the uploaded knowledge base — gaps produce hallucinated answers

---

## Option 3: Pipeline we build alongside Discourse

This is the path to the full vision. Discourse provides the interface and the RAG engine; we build and maintain the data pipeline that keeps the knowledge base current.

### Steps

1. **Scrape documentation sites** — docs.mosip.io/1.2.0, docs.esignet.io, docs.inji.io — and convert to `.md` files using tools like `wget`, `scrapy`, or a Docusaurus export script
2. **Export Confluence** — pull pages via the Confluence REST API, convert to `.md`
3. **Pull YouTube transcripts** — use `yt-dlp --write-auto-subs` to extract captions as `.txt`
4. **Upload to the AI persona** — via Discourse API (`PUT /discourse-ai/ai-personas/:id` with `rag_uploads` parameter)
5. **Schedule re-sync** — weekly or monthly cron job to keep content fresh as docs update

This is buildable with moderate engineering effort and stays entirely within Discourse's native RAG system.

---

## What to be honest about

- Auto-responding accurately on DPI documentation requires a **curated, maintained knowledge base** — quality degrades if documentation drifts and files are not re-uploaded
- The bot will hallucinate on questions outside its knowledge base unless explicitly instructed to say "I don't know" and link to official docs
- **Start with whisper mode** — human review before any public reply — not full auto-response from day one
- Confluence and YouTube are **not plug-and-play** — they need a small data pipeline that someone owns and maintains
- GitHub works natively but only for reading file content and diffs — not for indexing an entire repo's documentation in bulk without a pipeline

---

## Recommended phasing

| Phase | What | Why |
|---|---|---|
| Now | Export MOSIP docs to `.md`, attach to a persona, deploy in whisper mode | Prove the knowledge base works before it speaks publicly |
| Next | Enable auto-responder in one category (e.g. Support) with whisper approval | Build moderator confidence, catch gaps |
| Then | Automate the doc sync pipeline (cron + Discourse API) | Keep the knowledge base current without manual effort |
| Later | Evaluate direct GitHub integration for code-level questions; explore Confluence API export | Expand coverage to developer queries |

---

## Sources

- [Discourse AI Plugin — Discourse Meta](https://meta.discourse.org/t/discourse-ai/259214)
- [Discourse AI Persona Upload Support](https://meta.discourse.org/t/discourse-ai-persona-upload-support/304049)
- [Discourse AI and RAG](https://meta.discourse.org/t/discourse-ai-and-retrieval-augmented-generation/286378)
- [AI Auto Responder — Discourse Meta](https://meta.discourse.org/t/discourse-ai-ai-auto-responder/356375)
- [AI Bot Personas — Discourse Meta](https://meta.discourse.org/t/ai-bot-personas/306099)
- [Discourse AI Features](https://www.discourse.org/ai)
