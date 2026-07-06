# 🗞️ Multi-Agent Daily Digest

A multi-agent orchestration system built with n8n that automatically pulls data from Gmail, Google Calendar, and Slack through three specialized AI agents, synthesizes everything into a single prioritized digest, and delivers it to Slack every day at 2 PM.

---

## 🧩 The Problem It Solves

PMs and TPMs spend the first hour of every afternoon piecing together context from multiple sources — checking email for urgent requests, scanning calendar for upcoming meetings, reviewing Slack for blockers and decisions. This system eliminates that.

One workflow. Three agents. One digest. Delivered automatically.

---

## 🧠 Workflow Overview

**34-node multi-agent n8n workflow — no code required**

```
Daily 2 PM Trigger
    ↓
Orchestrator       → Single config node — controls all agents
    ↓
Guardrails         → Validates config before agents run
    ↓
State Management   → Loads digest history for delta tracking
    ↓ (parallel execution)
┌─────────────────────────────────────────────────────┐
│  Gmail Agent          → Triages high-signal emails  │
│  Calendar Agent       → Summarizes events and prep  │
│  Slack Agent          → Extracts decisions, blockers│
└─────────────────────────────────────────────────────┘
    ↓
Merge All Agents   → Combines 3 agent outputs
    ↓
Aggregate Outputs  → Formats for synthesizer
    ↓
Synthesizer Agent  → Deduplicates, prioritizes, tags [New] vs [Recurring]
    ↓
Send Digest to Slack   → Posts to #daily-2pm-digest
    ↓
Save Digest to Sheet   → Appends to digest history for next run

Error Trigger → Notify Error via Slack → Posts failures to #project-errors
```

---

## 🏗️ Architecture Layers

### Layer 1 — Control
| Component | Purpose |
|---|---|
| Orchestrator | Single config node — edit once to control all agents. Source routing decided here. |
| Guardrails | Validates config before agents run. Catches missing channels, unconfigured profiles, disabled sources. |
| State Management | Loads previous run context for [New] vs [Recurring] tagging. In production, swap for a real DB / Sheets read. |

### Layer 2 — Execution
| Component | Purpose |
|---|---|
| Slack Agent | Searches #finance-reports for decisions, blockers, and actions. Uses GPT-4o-mini. |
| Gmail Agent | Fetches recent emails, triages high-signal items. Uses GPT-4o-mini. |
| Calendar Agent | Fetches upcoming calendar events, summarizes prep needed. Uses GPT-4o-mini. |
| Synthesizer Agent | Merges all 3 outputs, deduplicates, sorts by urgency, tags [New] vs [Recurring]. Uses GPT-4o-mini. |

### Layer 3 — Reliability
| Component | Purpose |
|---|---|
| Error Trigger | Catches any workflow failure and posts to #project-errors with node name and error message. |
| Graceful Degradation | If one source fails or returns empty, the synthesizer still produces a digest from available sources. |
| Digest History | Reads last 7 rows from Google Sheets before synthesis — enables delta tracking across runs. |

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| n8n | Workflow automation and multi-agent orchestration |
| OpenAI GPT-4o-mini | All 4 AI agents (3 specialists + synthesizer) |
| Gmail API | Email data source |
| Google Calendar API | Calendar data source |
| Slack API | Slack data source + digest delivery |
| Google Sheets | Digest history storage for delta tracking |

---

## 📁 Files

| File | Description |
|------|-------------|
| `Multi_Agent_Daily_Digest_JattaAI.json` | n8n workflow export — import directly into n8n |
| `README.md` | Project documentation |

---

## 🚀 How to Use

**Prerequisites:**
- n8n Cloud account (free trial at n8n.io)
- Slack workspace with a bot app (xoxb- token)
- Google account with Gmail and Calendar access
- OpenAI API key (or n8n free AI credits)

**Setup Steps:**

1. Import `Multi_Agent_Daily_Digest_JattaAI.json` into n8n via **Add workflow → Import from file**
2. Connect credentials:
   - Google OAuth2 → Fetch Gmail Emails + Fetch Calendar Events
   - Google Sheets OAuth2 → Read Digest History + Save Digest to Sheet
   - Slack Bot Token → Search Slack Channels + Send Digest to Slack + Notify Error via Slack
   - OpenAI API key → All 4 LLM nodes
3. Create a Google Sheet with two columns: `date` and `digest` — update the sheet ID in Read/Save Digest History nodes
4. Create two Slack channels: `#daily-2pm-digest` (digest delivery) and `#project-errors` (error alerts)
5. Invite your Slack bot to both channels: `/invite @ProjectBot`
6. Update the Send Digest to Slack node with your `#daily-2pm-digest` channel
7. Publish the workflow — it will run automatically at 2 PM daily
8. Test immediately using the Execute workflow button

---

## 📋 Digest Output Format

```
🔥 Top Priorities
- [items or None]

✅ Decisions / Approvals
- [items or None]

⏰ Deadlines
- [items or None]

🚧 Risks / Blockers
- [items or None]

👥 Who to ping today
- [items or None]
```

---

## 🔑 Key Design Decisions

**Orchestrator as single source of truth** — All agent configuration lives in one node. Change the user profile, enabled sources, or parameters in one place and all downstream agents update automatically.

**Parallel agent execution** — All three specialist agents (Gmail, Calendar, Slack) run simultaneously, not sequentially. This keeps total runtime under 10 seconds regardless of how many sources are added.

**Delta tracking via Google Sheets** — The workflow reads the last 7 digest runs before synthesis. The synthesizer uses this history to tag items as [New] vs [Recurring] — so you know what's genuinely new vs. what has been lingering.

**Graceful degradation** — If Gmail returns no emails or Calendar has no events, the synthesizer skips that source and notes it rather than crashing the entire workflow.

**Multi-model strategy** — GPT-4o-mini for all specialist agents (fast, low cost) and GPT-4o-mini for the synthesizer. In production, upgrade the synthesizer to GPT-4o for higher quality final output.

---

## 🚀 Stretch Goals (Production Enhancements)

- **Persistent Memory** — Connect Pinecone or Weaviate for long-term context across runs
- **RAG Integration** — Add retrieval step so synthesizer can query a PM knowledge base
- **Multi-Model Strategy** — Use GPT-4o for the synthesizer for higher quality output
- **Output Validation** — Add post-synthesis check for empty or malformed digests
- **Container Isolation** — Run each agent in an isolated execution context to prevent data leakage

---

## 👤 Author

**Praveen Kumar Jatta** — Senior Technical Program Manager | AI Automation Consultant

- 🌐 [jattaai.com](https://jattaai.com)
- 💼 [linkedin.com/in/praveenjatta](https://linkedin.com/in/praveenjatta)
- 🐙 [github.com/praveenjatta](https://github.com/praveenjatta)
- 📅 [Book a free discovery call](https://calendly.com/praveenjatta/free-ai-automation-discovery-call)

---

## 📄 License

MIT License — free to use and modify with attribution.
