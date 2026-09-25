# AI-Powered Helpdesk & Customer Service Workshop

A free, 3-day, ~6-hour/day workshop teaching AI-powered helpdesk automation with NotebookLM, n8n, Groq, and Gemini. Participants leave with a working end-to-end AI helpdesk demo and a one-page rollout plan for their own workplace.

**[▶ View the slide deck](https://deval.github.io/ai-helpdesk-modules/deck/workshop-deck.html)**

## Contents

- [`ai-helpdesk-workshop.md`](ai-helpdesk-workshop.md) — the full workshop doc: all 3 days, 11 modules, labs, and the capstone
- [`docker-compose.yml`](docker-compose.yml) — starts n8n locally (`localhost:5678`), plus an optional local Postgres/Adminer profile
- [`deck/`](deck) — the reveal.js slide deck built from the workshop doc
- [`sample-data/`](sample-data) — sample DJP Coretax documents and synthetic support tickets used throughout the labs

## Quick start

```bash
docker compose up -d
```

Then open `http://localhost:5678`. See the workshop doc's Prerequisites section for the full account/setup checklist (Groq, Gemini, NotebookLM).
