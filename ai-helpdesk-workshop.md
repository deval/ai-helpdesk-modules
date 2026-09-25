# AI-Powered Helpdesk & Customer Service Workshop

**Format:** 3 days, ~6 hours/day
**Audience:** Technical / IT & ops staff
**Cost:** $0 — free tiers only
**Outcome:** Each participant leaves with a working end-to-end AI helpdesk demo (knowledge base + automation) and a one-page rollout plan for their own workplace.

---

## Tool stack (all free)

| Tool | Notes |
|---|---|
| **Google NotebookLM** | `notebooklm.google.com` — free Google account. Source-grounded Q&A, summaries, FAQ & audio generation from your own docs. |
| **n8n** | Self-hosted via a single `docker-compose.yml` (see below); free for this internal/self-hosted use. Runs entirely on `localhost:5678` — no tunnel needed for anything in this workshop except the optional Telegram stretch goal (see Module 6). |
| **Groq API / Console** | Free-tier key, no credit card. Used for Day 1's live prompt-tuning sprint — near-instant responses (LPU hardware) matter for a live, rapid iterate-and-break exercise. |
| **Gemini API** | Free-tier key, no credit card. Used for Day 2/3's n8n AI nodes — same model family as NotebookLM (reinforces the grounding discussion), and its 1M-token context comfortably holds a pasted knowledge-context file plus a ticket. |
| **Sample data** | Two official documents from the same Indonesian government body — **Direktorat Jenderal Pajak (DJP)** — covering its 2025 Coretax system: the *Panduan Ringkas Coretax* PDF guide and the official *FAQ Coretax* web page. Synthetic support tickets, authored for the course, reference both. |
| **Postgres + Adminer** *(optional)* | Local, credential-free alternative to Google Sheets for the review-queue/logging steps — opt in via `docker compose --profile local-db up`. See the storage note under Module 6. |

### ⚠️ Free-tier & licensing check (verified 2026-09-23 — re-verify close to your workshop date, these terms move fast)

| Tool | Status | Notes |
|---|---|---|
| **No tunnel needed** (ngrok/Tailscale removed) | Simplified 2026-09-25 | Earlier versions of this doc used ngrok, then Tailscale Funnel, to expose n8n publicly for a live Telegram bot demo. Since only Telegram actually needs a public URL — everything else in the workshop (Chat Trigger, Form Trigger, all LLM calls) works entirely on `localhost` — the tunnel was removed from the default setup. Telegram is now an optional stretch goal (Module 6) where a participant who wants it brings their own tunnel (ngrok, Tailscale Funnel, Cloudflare Tunnel, etc.) — not required for the core workshop path. |
| **NotebookLM** | **Renamed, mostly fine** | Restructured into Google's AI subscription bundle in May 2026 and renamed **Gemini Notebook** on 2026-07-16 — the UI/branding participants see may say "Gemini Notebook," not "NotebookLM." Free tier itself is still usable: 50 sources/notebook, 100 notebooks, 50 chats/day. New as of 2026-09-02: a rolling compute quota that refreshes every 5 hours up to a weekly cap — worth knowing if a full room hammers it back-to-back during Lab 1.1/1.2, since that's a new failure mode not in the doc's existing Gemini-API caveat. |
| **Groq** | Free, as documented | Free tier confirmed still live with no credit card: ~30 RPM / 1K req/day / 8K TPM / 200K TPD on `openai/gpt-oss-120b` (updated model — see the model table below; the doc's original pick, `llama-3.3-70b-versatile`, is dead). |
| **Gemini API** | Free, with the caveat already in Module 11 | Confirmed: free-tier prompts/responses can be used to train Google's models; paid tier is excluded from training. The existing privacy checklist in Module 11 already tells participants to keep real customer data off the free tier — that's correct and sufficient, no change needed. |
| **Docker Desktop** | Free — but not for everyone | Free for individuals, students, and businesses under 250 employees **and** under $10M annual revenue. A company outside those thresholds needs a paid Docker subscription per seat. Not currently called out anywhere in this doc — added to Prerequisites below. |
| **n8n** | Free for this workshop's use case | Self-hosted n8n is under the **Sustainable Use License** (a "fair-code," not OSI-open-source, license): free to self-host and use internally, but you can't resell it as a multi-tenant SaaS. Since every lab here is internal/self-hosted automation, this workshop is squarely inside the free/permitted use — just don't describe n8n as "open source" in participant materials, since it technically isn't. |
| **Telegram Bot API / Google Sheets** | Free | No material licensing or quota concerns for workshop-scale usage. |

### Which model, which module

| Module | Model | Why |
|---|---|---|
| Day 1 · Lab 1.0 (prompt-tuning sprint) | **Groq** `openai/gpt-oss-120b` | Near-instant responses keep a live "write → run → break → rewrite" loop fast in front of a room. Free tier: 30 RPM, 1,000 req/day, 8K TPM, 200K TPD. (Replaces `llama-3.3-70b-versatile`, which Groq deprecated 2026-06-17 and fully decommissioned 2026-08-16 — the old model no longer responds at all.) |
| Day 2/3 (n8n classify + grounded draft + RAG) | **Gemini 2.5 Flash** | Same family as NotebookLM — ties the Day 3 grounding discussion back to Day 1. 1M-token context avoids trimming when a knowledge-context file is pasted into the prompt. Free tier: 10 RPM, ~250K TPM confirmed; **daily request cap is no longer a fixed published number** — Google now sets it per-project, reported anywhere from 500–1,500/day depending on source. Check the actual figure in your AI Studio console's quota page, don't plan around a specific number. `generateContent` (used throughout this doc) is confirmed still supported as of this check, but Google marked it "legacy" in June 2026 in favor of a newer Interactions API — fine to keep using for now, but worth a quick check that it's still live before running this again in a year. |

> **Caveat to flag in the setup email:** both providers here have a track record of yanking things with short notice — Google cut Gemini free-tier limits by 50–80% without notice in Dec 2025, and Groq fully killed `llama-3.3-70b-versatile` about two months after announcing its deprecation (this doc's original model pick). Re-check both the model ID and the rate limits in the Groq console / AI Studio the morning of the workshop, not just once during planning.

### What participants should have / do before Day 1

**Hardware & access:**
- A laptop with **admin/install rights** — Docker Desktop's installer needs them, and IT-locked corporate laptops are the single most common no-show cause for workshops like this. Confirm this explicitly in the setup email, don't assume.
- At least **5GB free disk space** (the n8n Docker image plus pulled layers) and a stable internet connection — the first `docker compose up` pulls several hundred MB.
- Check your organization's size against Docker Desktop's free-use threshold (**under 250 employees and under $10M revenue** — see the licensing table above). If your org doesn't qualify, either arrange paid Docker seats ahead of time or substitute Docker Engine/Colima/Podman, which aren't subject to the same license.

**Skills assumed:**
- Comfortable running commands in a terminal (copy/paste is enough — no programming required beyond editing values in a `.env` file and pasting provided JSON).
- No prior n8n, NotebookLM, or LLM API experience needed — that's what Day 1–2 teach from zero.

**Accounts & keys to set up:**

1. Install **Docker Desktop** (docker.com/products/docker-desktop) — see the licensing note above first.
2. Create/confirm a **Google account** with NotebookLM (now branded **Gemini Notebook**) access — sign in at notebooklm.google.com, confirm you can create a notebook.
3. Create a **Groq account** at console.groq.com → **API Keys** → **Create API Key** → copy and store it.
4. Create a **Google AI Studio** key at aistudio.google.com/apikey → **Create API key** → copy and store it. (This is the Gemini API key.)
5. A setup-check email goes out 3 days prior confirming all accounts/keys are working, **and explicitly asking participants to confirm they have laptop admin rights** — this is worth its own line item, not just a keys check.

*(No tunnel account needed — see Module 6 if you want to do the optional Telegram stretch goal, which requires bringing your own.)*

### `docker-compose.yml` — starts n8n, local only

See [`docker-compose.yml`](docker-compose.yml) in this directory. One service, no tunnel, no bootstrap:

```bash
docker compose up -d
```

Open `http://localhost:5678` to reach the n8n editor. On first load, n8n asks you to create an owner account (any local email/password — this stays on your machine). That's the entire setup for the core workshop path — Chat Trigger and Form Trigger (Lab 2.1, Parts C/D) both work over `localhost`, and every LLM call (Groq, Gemini) is outbound, so nothing here needs a public URL.

If you want the optional Telegram stretch goal (Module 6, Part B), you'll need to expose port 5678 publicly yourself — any tunnel tool works (ngrok, Tailscale Funnel, Cloudflare Tunnel). That's a bring-your-own-tool step now, not something this compose file sets up for you.

---

## Day 1 — Knowledge foundations with NotebookLM

**Goal:** Turn messy internal docs into a queryable, citation-backed knowledge base — the raw material every later automation will draw on.

### Module 1 · What is an LLM, and where does it fit in a helpdesk (09:00–10:00)

#### What is an LLM?

A large language model is not a database and it does not "look up" answers. It's a statistical next-token predictor: trained on huge amounts of text, it learns which token is most likely to come next given everything before it, and it generates a reply by repeating that prediction one token at a time. There is no internal fact-checker and no concept of "I don't know this" — an accurate answer and an invented one are produced by the exact same mechanism. This is the root cause of everything in Module 2's hallucination discussion, so it's worth internalizing before anything else: an LLM optimizes for *plausible*, not *true*.

#### How this helps a helpdesk

Because an LLM is fluent at pattern-completion over text, it's genuinely good at tasks that are fundamentally about *transforming* text it's already been given: drafting a reply in a consistent tone, summarizing a long thread, classifying a ticket into a category, or answering a question when the answer is fully contained in the text handed to it. None of that requires the model to "know" anything beyond its prompt — it's applying learned language patterns to your input. That's exactly why grounding (Day 1's NotebookLM, Day 3's context injection) works: you're not asking the model to recall a fact, you're asking it to transform a fact you already gave it.

#### The risk this creates

The same mechanism that makes an LLM useful — generating fluent, plausible text — is what makes it dangerous unsupervised. When it's asked something outside what it was given, it doesn't switch to "I don't have this information" by default; it keeps predicting plausible-sounding text, which produces a confident, fluent, wrong answer. For a helpdesk this is the worst-case failure mode, because a wrong answer delivered confidently is harder for a customer (or a reviewing agent) to catch than one that's obviously uncertain. Module 2 and Lab 1.0 make this concrete; grounding, guardrails, and human review — the rest of this workshop — exist specifically to contain this one risk.

#### The three levels of AI in support

There are three distinct levels of AI involvement in support work, and most of the confusion in "should we use AI here?" conversations comes from not naming which one you mean:

- **Ticket deflection** — the customer never reaches a human. A chatbot or self-serve search answers the question directly from documentation. Works well for high-volume, low-ambiguity questions ("how do I reset my password", "what's your return window"). Fails badly when the answer depends on account-specific state the bot can't see, or when the policy has exceptions.
- **Agent-assist** — a human agent stays in the loop, but AI drafts replies, summarizes long threads, suggests knowledge-base articles, or classifies/tags tickets. The human approves or edits before anything goes to the customer. This is the safest place to start, and where most of this workshop's pipeline lives (Day 2–3's "draft → hold for review" pattern).
- **Full automation** — AI takes an action with no human step: auto-closing a ticket, auto-issuing a refund, auto-escalating. Powerful, but any mistake reaches the customer or your systems directly with no checkpoint. Reserve this for narrow, low-risk, high-confidence cases only (Day 3's guardrail lab is designed to carve out exactly that narrow slice).

**Where LLMs genuinely help:** drafting text in a consistent tone, summarizing long ticket threads, classifying/tagging, answering questions that are fully contained in your documentation.

**Where they don't:** deciding on irreversible actions (refunds, account deletion, legal commitments), resolving ambiguous policy exceptions, anything requiring real-time account state the model wasn't given.

**Reference numbers to ground the discussion** (industry-reported ranges, cite as approximate — actual numbers vary widely by company and ticket mix): AI-assisted deflection commonly reported in the 20–40% range for well-scoped FAQ-style ticket categories; agent-assist drafting has been reported to cut average handle time by roughly 10–30% when agents edit rather than write from scratch. Treat these as *plausible ranges to sanity-check against*, not promises — one of the biggest risks in adopting AI for support is a vendor's inflated deflection number that doesn't hold once it hits a real, messy ticket queue.

**Outcome:** a shared vocabulary (deflection / agent-assist / full automation) and a working checklist for "should this specific ticket type be automated, and at which level?" — used again in Day 3's capstone.

### Module 2 · LLM & prompting basics for support content (10:15–11:30)

**Tokens and context window.** A token is roughly ¾ of an English word. Every model has a maximum context window — the total tokens it can consider at once, input plus output. Gemini 2.5 Flash's free tier gives you a 1-million-token window (roughly 750,000 words) — big enough to paste an entire knowledge-context file plus a ticket with room to spare. This matters concretely for Day 3: you will not need to worry about truncating your exported NotebookLM content to fit.

**Hallucination.** A model will produce fluent, confident, *wrong* text when it isn't given the actual facts and is asked a question that requires them — for example, being asked about a refund policy without being told what the policy actually is. It doesn't "know" it's guessing. This is the single biggest risk in a helpdesk context, because a confidently wrong answer about a policy is worse than no answer at all. The fix is **grounding**: giving the model the actual source text as part of the prompt, and instructing it to only use that text — not what it doesn't have.

**Anatomy of a support-reply prompt.** A reliable prompt for this use case has five parts:

1. **Role** — who is the model acting as (a support agent for a specific company).
2. **Rules/constraints** — hard boundaries: what it must never claim, what tone, what length.
3. **Reference facts** — the actual policy/documentation text (this is the grounding).
4. **The input** — the ticket itself.
5. **Output format** — plain reply text, or structured JSON if a downstream system (like n8n) needs to parse it.

Missing parts 2 and 3 is exactly what causes the failure in the worked example below.

#### Lab 1.0 — Prompt-tuning sprint (live, paired)

**Setup (5 min):**
1. Go to `console.groq.com`, sign in, click **Playground** in the left nav.
2. In the model dropdown at the top, select `openai/gpt-oss-120b`.
3. Confirm you can send a message and get a response before continuing.

**Round 1 — everyone writes the naive prompt (10 min):**
4. Paste the fixed ticket below into the Playground as a user message, preceded by the naive instruction.
5. Read the model's output out loud at your table. Note anything it invents that isn't in the ticket.

**Round 2 — pair up and break each other's prompts (15 min):**
6. Find a partner. Trade your improved prompt (write your own version of the "Improved prompt" pattern below, adapted to the ticket).
7. Run your partner's prompt against this second ticket: *"I was charged twice for order #55901, please fix this."*
8. Try to make it fail: does it invent a refund amount? Does it ignore the double-charge specifics? Note exactly what breaks.

**Round 3 — group diagnosis (15 min):**
9. Facilitator collects 3 of the worst outputs from the room and projects them.
10. As a group, identify which of the five prompt parts (role / rules / facts / input / format) was missing or weak, and rewrite the prompt together on screen.

##### Worked example (use this exact ticket + prompts on the slide)

**Fixed sample ticket:**
> Subject: Refund for late delivery
> "My order #48213 was supposed to arrive last Tuesday, it's now 6 days late and I still don't have it. I want a refund. Also your tracking page just shows 'in transit' with no updates."

**Naive prompt (what most people write first):**
```
Reply to this customer complaint politely.
```
**Typical output (fails):** "I'm so sorry for the delay! I've issued a full refund of $[amount] to your account, which should appear in 3–5 business days." — invents a refund amount and a policy decision no one made, ignores the tracking complaint. This is the hallucination moment the group diagnoses live.

**Improved prompt (built together after the diagnosis):**
```
You are a support agent for [Company]. Reply to the ticket below.

Rules:
- Only state facts present in the ticket or in the policy notes provided
- Do NOT promise a refund amount or approval — refunds over 5 days late are
  routed to a human for approval per policy
- Acknowledge the specific complaint (late delivery + broken tracking)
- Tone: warm, concise, max 120 words
- End with the concrete next step the customer should expect

Policy notes:
- Orders >5 days late qualify for refund review, not automatic refund
- Tracking issues get escalated to logistics within 1 business day

Ticket:
"My order #48213 was supposed to arrive last Tuesday..."
```
**Resulting output:** acknowledges both issues, states the order qualifies for refund *review* (not a promised amount), commits to a logistics escalation within 1 business day, stays under 120 words — no invented facts.

### Module 3 · NotebookLM deep dive (11:45–13:00)

NotebookLM's core mechanic: every source you upload is chunked and indexed, and every answer it gives is required to cite the specific passage(s) it drew from — click a citation and it highlights the exact sentence in the source viewer. This citation-first design is *why* it's a good teaching tool for grounding: participants can directly verify whether an answer is actually supported, rather than trusting it on faith.

#### Input vs. output

| Input — what you feed it | Output — what it produces |
|---|---|
| PDFs, Google Docs/Slides, plain text | Cited Q&A chat answers (source-linked) |
| Pasted website URLs, YouTube links | Notebook guide / summary of all sources |
| Audio files | Generated FAQ, study guide, timeline docs |
| Up to 50 sources/notebook (free tier), ~500k words each | Audio Overview (two-host podcast discussion) |

> **No API, no export button.** NotebookLM is a UI product — there is no official way for n8n (or anything else) to call it directly or pull structured data out automatically. Every output above is something a person reads or copies by hand. That's why Lab 1.2 below treats the knowledge base as a *manual, one-time export*: you copy NotebookLM's generated text into a plain file, and that static file is what Day 3's n8n workflow reads — not a live connection. If a notebook's sources change, someone has to re-export by hand.

#### Lab 1.1 — Build your first knowledge notebook

1. Go to `notebooklm.google.com` and sign in.
2. Click **+ New notebook** (or **Create new**) on the home screen.
3. Click **+ Add source**. Choose **PDF** and upload the sample DJP guide (`sample-data/panduan-ringkas-coretax.pdf` — see Sample Data), and choose **Website** and paste the sample government FAQ URL — do both, as two sources in the same notebook.
4. Wait for both sources to finish processing (a progress indicator appears next to each; usually under a minute per source).
5. In the chat panel on the right, ask: *"What are the main sections of this document?"* — confirm the answer includes numbered citation chips.
6. Click one of the citation chips — confirm it opens the source viewer and highlights the exact passage.
7. Ask 4 more real support-style questions, one at a time (e.g. *"What should I check first if my e-Faktur submission fails?"*). For each answer, click every citation and confirm the source actually supports the claim. Note any answer that cites weakly or not at all.
8. Open the **Studio** panel (right-hand side). Click **FAQ** — this generates a full FAQ document from your sources. Skim it for accuracy against what you already read.
9. In the same Studio panel, click **Study guide** — this generates a structured troubleshooting/reference guide. Keep this tab open; you'll use it in Lab 1.2.

### Module 4 · From notebook to reusable knowledge base (14:00–16:30)

Two practical retrieval-quality issues show up immediately once you're grounding real ticket replies, not just answering isolated questions:

- **Source granularity** — a single giant PDF with no headings gives NotebookLM (and any RAG system) worse retrieval than the same content split into clearly-titled sections. If your own team's docs are one giant wiki page, expect worse citation quality than the structured examples in this workshop.
- **Contradictions across sources** — when two uploaded documents disagree (an old policy PDF and a newer webpage, say), NotebookLM will cite *both*, and it's on you to notice the conflict and decide which is authoritative. This is a real failure mode in production knowledge bases, not just a lab exercise — worth calling out explicitly.

Because there's no automatic export (Module 3), turning NotebookLM's output into something n8n can use on Day 3 is a manual step: copying the generated text into a plain file. Keep the file's structure simple and predictable — a flat markdown file with clear headers — since on Day 3 you'll be pasting its entire contents into a prompt, not querying it with anything smarter.

#### Lab 1.2 — Draft real ticket replies, then export by hand

1. Open the sample ticket set (provided as `sample-tickets.csv` — see Sample Data). Pick 5 tickets.
2. For each ticket, paste its full text into the NotebookLM chat with this instruction prefix: *"Draft a reply to this support ticket using only the information in the sources. If the sources don't cover something the customer asked, say so explicitly instead of guessing."*
3. For each of the 5 drafts, check every claim against a citation. Mark any invented claim (a policy, a number, a promise) that has no citation — this is your **hallucination baseline** for the day; compare notes with your table.
4. Create a new plain-text or markdown file locally named `knowledge-context.md`.
5. Go back to the **Study guide** and **FAQ** outputs from Lab 1.1. Copy both in full into `knowledge-context.md`, under two headers: `## Study Guide` and `## FAQ`.
6. Skim once more for anything sensitive that shouldn't leave the notebook (internal names, unrelated info) and trim it — remember this file gets pasted into a third-party LLM API prompt on Day 3.
7. Save the file. This is your carried-forward deliverable.

**Deliverable:** one hand-exported `knowledge-context.md` file per participant, carried into Day 3.

---

## Day 2 — Automating tickets with n8n

**Goal:** Go from zero to a working, AI-augmented ticket pipeline: intake, classification, and drafted replies — all inside n8n.

### Module 5 · n8n fundamentals (09:00–10:15)

n8n workflows are directed graphs of **nodes**. A **trigger node** (diamond-shaped icon, always the first node) starts a workflow — on a webhook call, a schedule, or an incoming message. Every other node processes the data that flows through it. Data between nodes is always JSON; you reference a previous node's output using **expressions**, written as `{{ }}` — e.g. `{{ $json.subject }}` pulls the `subject` field from the current item.

**Credentials** are stored once (Settings → Credentials, or inline when configuring a node) and reused across workflows — you'll create one for Telegram and one for the Gemini API today, and never re-paste the key again.

**Manual vs. production execution:** while building, you run a node with **"Test step"** or the whole workflow with **"Test workflow"** — this uses real data you provide and shows you the exact JSON at every step, which is the fastest way to debug an expression. A workflow only listens for *real* incoming webhooks/messages once you flip the **Active** toggle (top right) to on.

#### Setup — bring up the stack

1. Run:
   ```bash
   docker compose up
   ```
2. Open `http://localhost:5678`. On first load, n8n asks you to create an owner account (any local email/password — this stays on your machine).

That's it — no tunnel, no bootstrap. (If you're doing the optional Telegram stretch goal in Module 6, set up your own tunnel tool before that point; not needed for anything else today.)

### Module 6 · Channels in & out (10:30–12:00)

Two default intake channels for the course, both served directly by n8n on `localhost:5678` — no tunnel, no external account: a generic **Webhook** trigger (any system that can POST JSON — this is the universal integration point for a real helpdesk platform like Zendesk or Freshdesk, demoed here with a local `curl`) and n8n's built-in **Chat Trigger** and **Form Trigger** (a real-time chat widget and a submission form, both good stand-ins for "a live support channel" without any networking setup).

> **Optional stretch goal — Telegram.** Lab 2.1's Part B wires up a real Telegram bot instead of the local Chat/Form Trigger. Telegram's servers need to reach n8n from the public internet, which means you'll need to expose port 5678 yourself first — any tunnel tool works (ngrok, Tailscale Funnel, Cloudflare Tunnel). This is intentionally left as a bring-your-own-tool step rather than baked into `docker-compose.yml`, since it's the only thing in the whole workshop that needs a public URL at all. Skip it if you'd rather not deal with tunnel setup — Chat/Form Trigger cover the same "live channel" teaching point for Module 7/8 onward.

> **Storage — default is Google Sheets, with two credential-free local alternatives.** Every "Add a Google Sheets node, Append Row" step in this workshop (Lab 2.1's four parts, Lab 2.3, Lab 3.1) can be swapped for one of these if a participant would rather not do the Google OAuth flow, or wants a more realistic backend:
> - **CSV file** — a **Read/Write File** node (operation: Write, with **Append** turned on) writing to a local path like `/home/node/.n8n-files/tickets.csv`. Simplest option, no extra container. One known rough edge: community reports of the Append toggle occasionally not flushing correctly on rapid back-to-back writes — fine for a workshop's pace of one ticket at a time, but don't rely on it for a real burst-traffic pipeline.
> - **Local Postgres** — a real database, run via `docker compose --profile local-db up` (adds `postgres` + `adminer` containers to the stack — see `docker-compose.yml`). n8n's native **Postgres** node, operation **Insert**, connects with host `postgres` (the container's service name — containers on the same compose network reach each other by name, not `localhost`), port `5432`, db `helpdesk`, user/password `n8n`/`n8n` (all pre-set in the compose file, nothing to sign up for). Browse the table during the workshop at `http://localhost:8080` (Adminer) instead of eyeballing a spreadsheet.
>
> Recommendation: keep Sheets as the taught default (it mirrors what most real support teams already use), but mention both alternatives up front so participants without a Google account, or who'd rather not do OAuth live, aren't blocked.

#### Lab 2.1 — Ticket intake workflow (Webhook, Chat, Form — plus optional Telegram)

**Part A — generic webhook:**
1. In n8n, click **+ Add workflow**.
2. Add a **Webhook** node (search "Webhook" in the node panel). Set **HTTP Method** to `POST`, and **Path** to `ticket-intake`.
3. Set **Respond** to `Immediately` for now (default response).
4. Click **Test step** on the Webhook node — n8n shows a **Test URL**. Copy it.
5. Open a terminal and send a test ticket:
   ```bash
   curl -X POST "<paste test URL here>" \
     -H "Content-Type: application/json" \
     -d '{"subject":"Refund for late delivery","body":"My order #48213...","customer_email":"test@example.com"}'
   ```
6. Confirm the JSON appears in the node's output panel in n8n.
7. Add a **Google Sheets** node (or **Airtable**) after the Webhook node. Connect a credential (OAuth for Sheets is fastest — n8n walks you through it). Set operation to **Append Row**, mapping `subject`, `body`, `customer_email` to columns.
8. Run the curl command again and confirm a new row appears in your sheet.

**Part B — Telegram (optional stretch goal, needs your own tunnel — see the note above Lab 2.1):**
9. In Telegram, message **@BotFather**. Send `/newbot`, follow the prompts (choose a display name, then a username ending in `bot`). Copy the API token it gives you.
10. Back in n8n, add a **Telegram Trigger** node to a *new* workflow. Click **Create new credential**, paste the bot token, save.
11. Set **Updates** to `message`. Save and **activate** the workflow (toggle top-right). If your tunnel tool auto-populates `WEBHOOK_URL` for n8n (as Tailscale Funnel or ngrok can), n8n registers the public webhook with Telegram automatically; otherwise register it by hand via Telegram's `setWebhook` API pointed at your tunnel's URL.
12. Open Telegram, find your bot by its username, and send it a message: *"My order #48213 hasn't arrived."*
13. Back in n8n, click into the workflow's **Executions** list (left sidebar) and confirm your message arrived as a new execution, with the message text under `$json.message.text`.
14. Add the same Google Sheets **Append Row** node as Part A, mapping `message.text` and `message.from.username` to your columns — now both channels log to the same place.

**Part C — Chat Trigger (no tunnel needed):**
15. Add a **Chat Trigger** node (search "Chat Trigger" in the node panel — it appears on the canvas as **"When chat message received"**) to a new workflow. This is n8n's built-in conversational trigger, served locally.
16. Leave the default options (Public Chat off is fine — this is for local demo use, not a real customer-facing widget). Save and activate the workflow.
17. Click **Open Chat** at the top of the Chat Trigger node — a chat window opens inside n8n. Type: *"My order #48213 hasn't arrived."*
18. Confirm the message arrived as a new execution, with the text under `$json.chatInput`.
19. Add the same Google Sheets **Append Row** node as Parts A/B, mapping `chatInput` to your ticket-body column — now three channels can log to the same place.

**Part D — Form Trigger (no tunnel needed, async instead of live chat):**
20. Add a **Form Trigger** node to a new workflow. Set **Form Title** to `Submit a support ticket`, and add three fields: `Subject` (element type **Text**), `Body` (element type **Textarea**), `Customer Email` (element type **Email**).
21. Save and activate. Click **Test step** — n8n shows a local form URL. Open it in a browser tab.
22. Fill in the form using the same fixed sample ticket from Module 2 (subject: "Refund for late delivery") and submit.
23. Back in n8n, confirm the submission arrived as a new execution with `Subject`, `Body`, and `Customer Email` as top-level fields — no parsing needed, unlike Telegram's nested `message.text`.
24. Add the same Google Sheets **Append Row** node, mapping the three form fields directly.

Whichever of Parts B–D a participant used, the output feeding into Module 7 is the same shape: a `subject`/`body`-style ticket record ready to classify. The rest of Day 2 and Day 3 don't care which intake channel produced it.

### Module 7 · Calling an LLM from n8n (13:00–14:30)

Two ways to call an LLM from n8n: a purpose-built node (fast to configure, less visible about what's actually happening) or a raw **HTTP Request** node hitting the provider's REST API directly (a few more fields to fill in, but it teaches the actual request/response contract — and the same pattern transfers to any other API you'll integrate later). This workshop uses **HTTP Request** for exactly that reason.

Gemini's REST contract, for reference:
```bash
curl "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"contents":[{"parts":[{"text":"Say hello in one sentence."}]}]}'
```
The response text is nested at `candidates[0].content.parts[0].text`.

#### Lab 2.2 — Classify & score every ticket

1. Build a small workflow: a **Manual Trigger** → a **Set** node with one sample ticket (`subject`, `body` fields, typed in directly) → an **HTTP Request** node.
2. Configure the HTTP Request node: Method `POST`, URL `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent`, and under **Query Parameters** add `key` = your Gemini API key (or use a **Header Auth** credential instead, for a cleaner separation — either works).
3. Set **Body Content Type** to JSON, and the body to:
   ```json
   {
     "contents": [{
       "parts": [{
         "text": "Classify this support ticket. Respond with ONLY valid JSON, no other text: {\"category\": \"billing|technical|general\", \"sentiment\": \"positive|neutral|negative\", \"urgency\": \"low|medium|high\"}\n\nTicket subject: {{ $json.subject }}\nTicket body: {{ $json.body }}"
       }]
     }]
   }
   ```
4. Test step the node. In the output, find `candidates[0].content.parts[0].text` — this is a JSON *string*, not yet parsed.
5. Add a **Code** node after it (JavaScript). Parse it into real fields:
   ```javascript
   const raw = $input.first().json.candidates[0].content.parts[0].text;
   const parsed = JSON.parse(raw.replace(/```json|```/g, '').trim());
   return [{ json: { ...$('Set').first().json, ...parsed } }];
   ```
6. Test step — confirm you now have `category`, `sentiment`, and `urgency` as real top-level fields.
7. Add a **Switch** node keyed on `{{ $json.category }}` with three outputs (billing / technical / general).
8. Add an **IF** node right after the Switch's outputs, checking `{{ $json.urgency }} == "high" AND {{ $json.sentiment }} == "negative"` — route true to a separate **"needs review"** branch (leave it as a dead-end node for now, e.g. a **No Operation** node, as a placeholder — you'll wire it up in Lab 2.3).

### Module 8 · Drafting replies automatically (14:45–16:30)

Same HTTP Request pattern as Lab 2.2, but now the prompt asks for a *drafted reply* instead of a classification — and instead of sending it, you write it to a place a human checks first. This is the "draft → hold for review" pattern that keeps Day 2 safely in agent-assist territory (Module 1's middle tier) before Day 3 adds any auto-send path.

#### Lab 2.3 — Draft-and-hold workflow

1. After the classification Code node from Lab 2.2, add another **HTTP Request** node calling Gemini again, this time with a drafting prompt:
   ```json
   {
     "contents": [{
       "parts": [{
         "text": "You are a support agent. Draft a reply to this ticket. Do not invent policy details you weren't given. End with a confidence note: state HIGH, MEDIUM, or LOW confidence based on whether you had enough information to answer fully.\n\nCategory: {{ $json.category }}\nTicket: {{ $json.body }}"
       }]
     }]
   }
   ```
2. Add a **Set** node to build the review-queue record: `ticket_subject`, `category`, `sentiment`, `urgency`, `draft_reply` (from the response text), `status` = `"pending review"`.
3. Add a **Google Sheets** (or Telegram, CSV, or local Postgres — see the storage note under Module 6) node to write that record out. This is your review queue.
4. **Manual approval step:** for this lab, approval is intentionally manual and outside the workflow — a reviewer reads the sheet/chat, edits the `status` column to `"approved"` or copies the approved text and sends it themselves. Building an automated approve-and-send trigger (e.g. a second workflow watching for `status == approved`) is a good stretch goal if time allows, but isn't required for the deliverable.
5. Run the full pipeline end to end with 3 different sample tickets (one clearly billing, one clearly technical, one ambiguous/negative-sentiment) and confirm all 3 land correctly in the review queue with sensible category/sentiment/urgency/draft values.

**Deliverable:** working classify → draft → hold-for-review n8n workflow.

---

## Day 3 — Connecting the two, and the capstone

**Goal:** Feed Day 1's grounded knowledge into Day 2's automation, add real guardrails, and ship a working end-to-end helpdesk agent.

### Module 9 · Grounding n8n replies in your knowledge base (09:00–10:30)

Right now, Day 2's drafting prompt has no access to real policy facts — it's exactly as prone to hallucination as Module 2's naive prompt was. Two ways to fix that at free-tier:

- **(a) Direct injection** — paste the entire `knowledge-context.md` file from Day 1 into the prompt, ahead of the ticket. Simple, no new infrastructure, and Gemini's 1M-token window means a knowledge file of a few thousand words costs you nothing in context budget. This is what the lab below does.
- **(b) Vector retrieval** — for a knowledge base too large to paste in full every time, a vector store node (n8n's built-in **Simple Vector Store** node, or an external free-tier Postgres/pgvector instance via Supabase) retrieves only the most relevant chunks per ticket. Same underlying principle NotebookLM uses internally. Worth introducing conceptually here even though the lab uses approach (a) — it's the natural next step once a real knowledge base outgrows "one file, pasted whole."

**Sizing check — if a participant's own knowledge base is a CSV/Sheet (a FAQ table, a ticket-history export) instead of prose,** here's how to estimate whether it still fits under approach (a) before deciding they need (b). Using ~¾ word per token (Module 2) and rounding row length to "words per row including all columns":

`max rows ≈ word budget ÷ words per row`

| Row size (example) | ~Words/row | Rows @ reliable-recall budget (~15K words) | Rows @ free-tier per-minute budget (~187.5K words) | Rows @ technical max (~750K words) |
|---|---|---|---|---|
| Short (FAQ: one question + one-line answer) | ~15 | ~1,000 rows | ~12,500 rows | ~50,000 rows |
| Medium (ticket: subject + short body + email) | ~40 | ~375 rows | ~4,700 rows | ~18,750 rows |
| Long (ticket with a full paragraph body) | ~100 | ~150 rows | ~1,875 rows | ~7,500 rows |

The middle column is the one to actually plan around — it's where recall stays reliable, not just where it technically fits. A participant can sanity-check their own data by opening it, eyeballing 5 rows, averaging their word count, and plugging that into the formula. If their real row count clears the middle column, approach (b) — vector retrieval — is worth the extra setup; if not, direct injection is simpler and just as accurate.

**Pipeline you're building today:**

```
Ticket in → Classify → Retrieve context → Draft reply → Confidence check → Send / Escalate
```

#### Lab 3.0 — Inject the knowledge-context file

1. Open your Day 2 draft-workflow. Add a **Read/Write File** node (or a **Set** node with the file's contents pasted in directly, if that's simpler) that loads your `knowledge-context.md` from Day 1.
2. Update the drafting HTTP Request node's prompt to include it:
   ```json
   {
     "contents": [{
       "parts": [{
         "text": "You are a support agent. Using ONLY the reference material below, draft a reply to the ticket. If the reference material doesn't cover the customer's question, say so explicitly rather than guessing.\n\nReference material:\n{{ $json.knowledge_context }}\n\nTicket: {{ $json.body }}"
       }]
     }]
   }
   ```
3. Re-run the same 3 test tickets from Lab 2.3. Compare the new drafts against the old ones — check specifically whether claims now trace back to the reference material.

### Module 10 · Guardrails & escalation logic (10:45–12:00)

A confidence label from the model itself (Lab 2.3's `HIGH/MEDIUM/LOW`) is a weak signal on its own — models are often confidently wrong. Pair it with **deterministic checks** that don't rely on the model judging itself: keyword tripwires for categories that should never auto-send (refunds, legal, safety, account cancellation), regardless of what the model's confidence says.

#### Lab 3.1 — Add the guardrail branch

1. After the drafting node, add an **IF** node with an OR condition:
   - `{{ $json.confidence }}` does NOT equal `"HIGH"`, **OR**
   - `{{ $json.body.toLowerCase() }}` contains any of: `refund`, `cancel`, `lawsuit`, `legal`, `chargeback`
2. **True branch (escalate):** route to your review queue from Lab 2.3 (whichever storage you used — Sheets, Telegram, CSV, or Postgres), tagged `"needs human review"`.
3. **False branch (auto-send eligible):** route to a **Set** node that marks the record `"auto-approved"` — for the lab, stop here rather than actually sending to a real customer (log it as "would have sent" instead). In production this branch would call your actual send channel (email/Telegram reply).
4. Test with 4 tickets designed to hit each path: one high-confidence/no-keyword (should auto-approve), one high-confidence-but-flagged-keyword (should escalate despite high confidence — this is the point of the OR), one low-confidence/no-keyword (should escalate), one low-confidence-and-flagged (should escalate).
5. Confirm all 4 land in the branch you expected. If the keyword-flagged-but-high-confidence ticket auto-approved, your OR condition is wrong — fix it before moving on.

### Module 11 · Metrics, privacy & rollout (13:00–14:00)

**Metrics that matter, and how to compute them from what you're already logging:**
- **Deflection rate** = tickets resolved without human involvement ÷ total tickets. Read directly off your review-queue log: `count(status == auto-approved) / count(all)`.
- **Escalation rate** = the inverse view: `count(status == needs human review) / count(all)` — track this by category; a category escalating >80% of the time is a sign your guardrail keywords (or your knowledge base coverage) need work, not that the category is unautomatable.
- **Time-to-first-response** — for agent-assist, measure time from ticket-in to draft-ready, not to customer-sent (the human approval step is outside your control).
- **CSAT delta** — compare satisfaction scores on AI-assisted vs. fully-human tickets over the same period; needs a few weeks of real volume to be meaningful, not something you'll see in a 3-day workshop.

**Privacy checklist before any real rollout:** strip or mask PII (full names, account numbers, payment details) before it goes into any third-party API call — Gemini's free tier explicitly states prompts/responses may be used to improve Google's products, which is a real reason to treat a company's actual customer data as out of scope for that tier. A production rollout uses a paid tier with a no-training data agreement, not the free tier this workshop runs on.

**Rollout plan, phased:**
1. **Shadow mode** — the AI drafts a reply, a human writes their own reply independently, and you compare the two without the AI output ever being sent. Measures quality without any customer-facing risk.
2. **Co-pilot mode** — human reviews and edits the AI draft before sending (Day 2's pattern). Runs until deflection/escalation numbers are trusted.
3. **Narrow auto-send** — Day 3's guardrail pattern, but scoped to one low-risk ticket category only (e.g. "where is my order" status lookups), with the keyword tripwire list reviewed by whoever owns policy risk before it goes live.

### Capstone — Build your team's helpdesk agent

Working solo or in pairs, assemble the full pipeline end to end using your own (or the provided) support docs and ticket samples, then demo it live to the group.

- NotebookLM knowledge base for at least one real product/policy area
- n8n workflow: intake → classify → grounded draft → guardrail → send/escalate
- One escalation rule specific to your team's actual risk area (not just the workshop's generic refund/legal keywords)
- 5-minute live demo + a one-page rollout plan for your workplace, using the phased structure from Module 11

---

## Take-home resources

- Sample dataset (DJP Coretax PDF guide + DJP FAQ page + synthetic ticket set, see Sample Data section)
- n8n workflow templates — exported JSON for each day's lab, ready to re-import
- `docker-compose.yml` — starts n8n on `localhost:5678`, plus optional local Postgres/Adminer
- Guardrail checklist — one-page confidence & escalation design worksheet

---

## Sample data — sources and tickets

**PDF source:** DJP *Panduan Ringkas Coretax* — https://www.pajak.go.id/en/node/113531 (local backup: `sample-data/panduan-ringkas-coretax.pdf`)
**Web source:** DJP *FAQ Coretax* — https://www.pajak.go.id/index.php/en/node/107900 (local backup: `sample-data/faq-coretax.html`)

> If either live link is down on workshop day, use the local backup in `sample-data/` instead — same content, saved 2026-09-23. Re-check both live URLs a few days before running this again, since government sites restructure without redirects sometimes.

Both are official Direktorat Jenderal Pajak (DJP) publications about the same system — Coretax, Indonesia's 2025 tax administration platform — so a participant can hold the full content of both in their head and check any NotebookLM answer against it directly, and the two sources genuinely overlap enough to make cross-referencing (Module 4) meaningful.

**Synthetic support tickets** (author these directly — no scraping needed, keeps licensing entirely clean):

1. *"I can't log into Coretax, it says my NPWP isn't recognized even though I registered last month. What do I do?"*
2. *"Do I need to re-register as a taxpayer now that Coretax has replaced the old system, or does my existing data carry over?"*
3. *"My e-Faktur won't generate in Coretax — I get an error at the final submit step. Is this a known issue?"*
4. *"I'm a PJAP (tax application provider) — does my integration still work with the new system, or do I need to change anything?"*
5. *"I made a payment through Coretax but it's not showing as received three days later. Should I pay again or wait?"*

Ticket 5 mirrors the Day 1 worked-example structure deliberately (a status/delay complaint with an implied request for reassurance or action) — useful for participants to compare against the refund-ticket example once they reach Lab 1.2.
