---
title: Projects
icon: fas fa-cubes
order: 2
---

A selection of things I've built — mostly in the intersection of **AI engineering, security and backend infrastructure**. Almost all of them started as a problem I had personally and turned into something useful enough to keep around.

---

## DemoFlow — AI-powered interactive product demos
**Live at [www.demoflow.dev](https://www.demoflow.dev)**

Upload screenshots, Claude Vision writes the captions and hotspot positions, share a link — or export a self-contained HTML file that works offline forever, no DemoFlow dependency. Branching flows, not linear slideshows; password-protected demos; per-screen analytics.

> **Stack:** Python · FastAPI · React · TypeScript · Tailwind · Supabase (Postgres + Auth + Storage + RLS) · Stripe (subscriptions, webhooks, customer portal) · Resend · Railway · Vercel · Docker · Claude Haiku 4.5 Vision
{: .prompt-info }

**What's interesting about it:**
- Self-contained HTML export with inlined CSS + vanilla JS — XSS-safe, no CDN, works in ten years with no DemoFlow servers running.
- Full REST API + zero-dependency Node.js CLI, so you can drive it from a CI pipeline.
- Row-level security in Postgres, API-key auth (SHA-256 hashed), bcrypt passwords, rate limiting per IP and per user.
- Parallel uploads with a two-pass Pillow image compression pipeline.

---

## EvalPriv — Self-hosted AI gateway with PII interception

A proxy you drop in front of your LLM traffic. One-line SDK change and every prompt leaving your network gets scanned for PII — emails, phone numbers, SSNs, Luhn-validated credit cards, ISO-13616-validated IBANs — with configurable **redact / block / log** modes per type.

> **Stack:** Ruby on Rails 8.1 (API-only) · React/Vite (zero UI deps, custom dark theme) · SQLite · Active Record Encryption (AES-256-GCM) · Solid Queue · Docker
{: .prompt-info }

**Other things it does:**
- Live feed of every request with model, latency, tokens, cost, PII alerts.
- Quick Eval: send a prompt to N models simultaneously, compare responses side by side.
- Quality judge: appoint any model as judge, auto-score responses 1–10 with reasoning.
- Supports OpenAI, Anthropic, Gemini, any OpenAI-compatible endpoint, and Ollama locally.

**Why I built it:** AI governance is the concern security teams in regulated industries actually raise. I wanted a concrete artefact showing what the controls could look like in practice.

---

## ModelArena — Head-to-head AI model benchmarking

Pairwise benchmarking for LLMs. Every model pair competes on identical prompts; A/B labels are randomly swapped per comparison to kill position bias in the judge. The output is a colour-coded **win-rate matrix** — who beats who on each task category.

> **Stack:** Ruby on Rails 8.1 · React/Vite · SQLite · Active Record Encryption · Solid Queue · Docker
{: .prompt-info }

**Design choices worth noting:**
- A and B are called in parallel per comparison (~2× faster than sequential).
- Error disqualification: a model that errors can never "win" — the working response wins automatically.
- Five built-in task suites (Code Gen, Code Review, Debugging, System Design, Technical Explanation) plus custom prompts added from the UI.
- **Switch prediction:** before swapping models in production, see the expected quality impact from historical data.

---

## AI-Augmented Zettelkasten — PKM with a Claude co-pilot

My personal knowledge base, Obsidian-backed, with Claude wired in as an integrated thinking partner through a custom **MCP server**. Deliberate separation of concerns: the AI handles retrieval and consistency at scale; the human handles emergent discovery through the graph view.

> **Stack:** Obsidian · Claude API · MCP server integration · structured markdown schema · custom prompt engineering
{: .prompt-info }

**What the MCP layer actually does:**
- Gives Claude live vault access for consistency checking across atomic notes.
- Suggests bidirectional link candidates on new notes.
- Surfaces cross-domain idea collisions — the moment Zettelkasten was invented for.
- Encodes a maturity progression model (seedling → evergreen) so notes have a lifecycle.

---

## Resume Tailor — AI pipeline for targeted résumés

A local CLI that reads a master profile, fetches a job description from any URL (including JS-rendered ATS platforms), runs a keyword gap analysis, and generates a tailored DOCX + PDF with an ATS keyword coverage report.

> **Stack:** Python · Anthropic Claude API · python-docx · Playwright · httpx
{: .prompt-info }

This one writes the résumés I actually send out. The Claude call is the interesting part — it's a careful prompt that preserves factual accuracy while reshaping emphasis toward the role.

---

## What's next

I have half-finished projects around agent observability, autonomous security-research pipelines, and a small language for describing AI evaluation rubrics. If any of them mature into something useful, they'll show up here — and get a write-up on the [blog](/).

Want to talk about any of this? I'm on [GitHub](https://github.com/cotneok) and [LinkedIn](https://www.linkedin.com/in/tsotne-okrostsvaridze-ab1b5958/), or just [email me](mailto:cotne.ok@gmail.com).
