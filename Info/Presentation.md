# Agents, Copilot Studio & Cowork

## 1. What is an "Agent"?
An agent is an AI assistant built for a **specific job** with its own knowledge, instructions, and (optionally) actions it can take on your behalf.

**Analogy:** If **M365 Copilot Chat** is a *general office temp* who can help with anything but knows nothing about your team, an **agent** is a *specialist colleague* you've trained for one role — e.g. "the Comms handbook expert" or "the Media & PR inbox triager." It only knows what you've given it, and only does what you've told it it can do.

---

## 2. M365 Copilot – "New Agent" (Agent Builder)
The lightweight, declarative agent created from inside M365 Copilot Chat ("+ Create agent").

### What it can do
- Ground answers in **SharePoint sites, files, or public web** you point it to.
- Custom instructions / persona / starter prompts.
- Lives inside M365 Copilot Chat and Teams.

### Limitations
- **No topics, tools, Power Automate flows, or AI prompts** (that's Copilot Studio territory). [1](https://microsoft.github.io/FastTrack/copilot-agent-strategy/copilot-agents-cost-tool/index.html)
- No autonomous triggers — it only responds when a user asks.
- No "deep thinking"/reasoning model toggle.
- Limited governance/analytics vs Copilot Studio.

### Where admins manage it
- **Microsoft 365 Admin Center** → Copilot → Agents (inventory, block, allow).
- **Power Platform Admin Center** for environments/credits when PAYG is enabled. [1](https://microsoft.github.io/FastTrack/copilot-agent-strategy/copilot-agents-cost-tool/index.html)

### Who can use it & sharing
- Any user with a **M365 Copilot license** can build one.
- Can be shared to: **specific people, a Team, or the whole org** (subject to admin policy).
- Other colleagues can use it **only if shared**, and ideally only if they also have a M365 Copilot license (otherwise PAYG credits get consumed). [2](https://www.linkedin.com/pulse/beginners-guide-copilot-credits-formerly-known-tiffany-songvilay-rgtjc)

---

## 3. Sadie's Two Requests – Context
1. **Comms Ways of Working handbook chat agent** – currently erratic; wants **deep thinking / reasoning** enabled.
2. **Media & PR email triage agent** – sift large volumes of inbound emails.

### Are these overkill for Copilot Studio?

- **Request 1 (handbook Q&A): No — but it does *not* need deep reasoning.** This is a classic **RAG (retrieval) problem**, not a reasoning problem. A standard Copilot Studio agent with a **generative answer (2 credits)** grounded on the handbook is the right tool. Sadie's "erratic results" are far more likely caused by **knowledge source scoping, weak system prompt, or missing fallback topic** than by the model not "thinking hard enough." Fix grounding first; only consider the reasoning toggle if accuracy is still poor on genuinely ambiguous questions.  [1](https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-messages-management)
- **Request 2 (email triage): No, but it's the *expensive* end of Copilot Studio.** It requires an **autonomous agent** with triggers, actions, and likely Power Automate flows — fine technically, but every autonomous trigger is **25 credits** regardless of M365 Copilot licensing. Needs a proper cost model and DPIA before build. [2](https://samexpert.com/copilot-studio-licensing-guide/)

> **Verdict:** Neither is overkill — but Request 1 needs **better configuration, not a bigger model**, and Request 2 needs **governance, scoping, and AI Engineer review** before rollout.
> **Verdict:** Not overkill — but Request 2 needs governance, scoping, and an AI engineer review before rollout.

---

## 4. Copilot Studio – Capability by Role

| Role | What they can do |
|---|---|
| **End users (with access)** | Chat with published agents in Teams / SharePoint / web / M365 Chat. Provide feedback. |
| **Makers (Copilot Studio licensed)** | Build agents with topics, knowledge sources, generative answers, actions, flows, AI prompts, reasoning models. [5](https://www.microsoft.com/en-us/microsoft-365-copilot/pricing/copilot-studio) |
| **Admins (Power Platform / M365)** | Manage environments, security groups, DLP, credit allocation, publishing approval, analytics, audit, block/allow models (e.g. Anthropic).  [3](https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-messages-management) |
| **Software developers / AI engineers** | Build custom connectors, Dataverse schemas, complex flows, prompt engineering, C# / Pro-code extensions, model selection, evaluation & guardrails, monitoring of credit burn. |

---

## 5. IT Team – Managing Agents Built in Copilot Studio
- **IT owns the platform**: environments, DLP, security groups, credit pools, publishing pipeline.
- **Makers build in dev/test environments only** — no direct publish to production.
- **AI Engineer reviews** every agent before org-wide release (prompt quality, data exposure, reasoning model use, cost forecast).
- **IT publishes** agents org-wide; **makers can publish team-wide** within their own environment.
- **Team-wide (≤1 directorate):** Maker request → IT/AI Engineer review → publish to that Team/site.
- **Org-wide:** Mandatory AI Engineer + IT review → security & cost sign-off → publish via central environment.
- **Quarterly review** of all live agents (usage, credit burn, accuracy, ownership still valid).

---

## 6. Costs – Copilot Studio
Source: [Microsoft Learn – Billing rates](https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-messages-management#copilot-credits-billing-rates)  [3](https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-messages-management)

### Key rates (per event)
| Feature | Credits |
|---|---|
| Classic answer | 1 |
| Generative answer | 2 |
| Agent action | 5 |
| Tenant Graph grounding | 10 |
| Premium AI tools (reasoning / "deep thinking") | 10 per response (100 / 10) |
| Content processing | 8 per page |
| Autonomous agent trigger | 25 (always charged, even for M365 Copilot users) |

**PAYG: $0.01 per Copilot Credit.** B2E usage is *free* for users with a M365 Copilot licence — **except** autonomous triggers. [6](https://www.linkedin.com/pulse/demystifying-microsoft-copilot-studio-credits-how-costs-kim-brian-cv5vc)[4](https://samexpert.com/copilot-studio-licensing-guide/)

---

### 💡 Example 1 – SharePoint agent on **SPS-Devops** site
**Scenario:** Agent grounded on the SPS-Devops SharePoint page. Users ask questions about MsgBroker, Toca automation, stored procedure logic, and how they link together.

**Assumptions:** 10 users × 5 questions/day × 22 working days. Each Q = generative answer (2) + tenant graph grounding (10) = **12 credits**.

| Variant | Credits / Q | Monthly credits | PAYG cost (no M365 Copilot license) | Cost if all 10 are M365 Copilot licensed |
|---|---|---|---|---|
| Standard | 12 | 13,200 | **~$132** | **$0** (B2E free) |
| With deep reasoning (+10) | 22 | 24,200 | **~$242** | **$0** (B2E free) |

> Even with deep thinking on, this is a low-cost agent. The cost only really lands if users are **unlicensed**.

---

### 💡 Example 2 – Sadie's Media & PR email triage agent (autonomous)
**Scenario:** Agent monitors a shared mailbox, classifies each email, routes/labels it, drafts a reply summary.

**Assumptions:** 200 emails/day × 22 working days. Each email = autonomous trigger (25) + generative answer (2) + graph grounding (10) + 1 action (5) = **42 credits** per email.

| Variant | Credits / email | Monthly credits | PAYG cost |
|---|---|---|---|
| Standard | 42 | 184,800 | **~$1,848 / month** |
| With deep reasoning (+10) | 52 | 228,800 | **~$2,288 / month** |

> **Important:** the 25-credit autonomous trigger is **always charged**, M365 Copilot licence or not. [4](https://samexpert.com/copilot-studio-licensing-guide/)
> This agent alone could run **~£18k–£22k/year** — needs business case + cost cap.

---

## 7. Copilot Credit Pricing Models (Reminder)
- **PAYG:** $0.01 / credit, billed in arrears. No upfront commit. Best for pilots.
- **Pre-Purchase Plan (P3):** 1-year upfront pool. Discount **5% → 20%** depending on tier (300k → 300M credits). Unused credits expire at year-end. [3](https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-messages-management)
- Prepaid pack: 25,000 credits / **$200/month** per tenant. [7](https://msadvance.com/en/copilot-studio-pricing-licensing-plans/)

---

## 8. Copilot Cowork – Pricing & How It Differs from Studio

### Pricing (same $0.01/credit currency)
| Scenario | Example | Illustrative credits/task | PAYG cost / task |
|---|---|---|---|
| **Light** | Weekly status update from calendar + priorities | 70–200 | ~$0.70 – $2.00 |
| **Medium** | Customer meeting briefing from email/CRM/files | 400–600 | ~$4 – $6 |
| **Heavy** | 6-month usage data analysis with leadership report | >1,500 | >$15 |

**Monthly forecast formula:** *Users × prompts/month × avg credits per prompt.*

### How Cowork differs from Copilot Studio

| | **Copilot Studio** | **Copilot Cowork** |
|---|---|---|
| Audience | Makers / developers building **bespoke agents** | End users running **agentic multi-step tasks** on their own work |
| UX | Low-code builder, topics, flows, connectors | Natural language; no build phase — just a prompt |
| Use case | Reusable agents shared org-wide (HR bot, IT helpdesk, handbook Q&A) | Personal productivity agentic workflows (briefings, analysis, drafts) |
| Governance | Environments, DLP, publish approval | M365 Admin Center cost dashboard, per-user caps |
| Billing | Per credit, per feature event | Per credit, per task (Light/Medium/Heavy) |
| Best for | Sadie's **handbook chat agent** | Sadie's **Media & PR triage** could also be done as a Cowork workflow per analyst |

### Practical considerations to flag to leadership
- **Cowork is more user-friendly and less dev-heavy** — faster to roll out, no maker training needed.
- **But cost per task can be high**, especially Heavy scenarios using reasoning models (Anthropic Opus, GPT-5.x). A single Heavy task >$15 stacks fast across a team.
- **Anthropic (Opus) models tend to consume more credits than GPT** for the same task — Opus is optimised for complex coding/reasoning, so it's deeper but more expensive. Standard tasks should stay on GPT. [8](https://community.powerplatform.com/forums/thread/details/?threadid=bf20bd8c-4745-f111-88b5-7ced8dcd2480)
- **AI Engineering team must approve Anthropic model enablement** (EUDB residency, data handling, model evaluation) before turning it on tenant-wide for Cowork or Studio. [6](https://www.linkedin.com/pulse/demystifying-microsoft-copilot-studio-credits-how-costs-kim-brian-cv5vc)

---

## 9. Recommendation Summary
- ✅ Build **Comms handbook agent** in **Copilot Studio** with reasoning enabled — low cost, B2E free for licensed users.
- ⚠️ **Email triage agent**: pilot only, autonomous + cost-capped, AI Engineer must review. Consider Cowork as an alternative for individual analysts before committing to an autonomous Studio build.
- 🛡️ IT to own platform & publishing; AI Engineer gates org-wide release; quarterly review of all agents.
- 💰 Stay on **PAYG** until 60-day usage baseline exists, then evaluate P3 tier for discount. [2](https://www.linkedin.com/pulse/beginners-guide-copilot-credits-formerly-known-tiffany-songvilay-rgtjc)

---

### Request 1 — Comms handbook bot
- Comms staff = M365 Copilot licensed ✅
- Published to Teams / M365 Copilot Chat ✅
- Interactive (users ask questions) ✅
- Standard generative answer + SharePoint grounding ✅
- → **$0 per interaction**

### Request 2 — Media & PR email triage bot
- Even if Media & PR staff are licensed, the bot is **autonomous** — it triggers itself when emails arrive
- → **25 credits per email, always charged** (~$1,800+/month at 200 emails/day)
- The B2E waiver does **not** apply

## Who can access Copilot Studio?

| Capability | M365 Copilot user (no extra Studio licence) | Copilot Studio licensed user |
|---|---|---|
| **Use** agents published by makers | ✅ Yes (free, B2E) | ✅ Yes |
| **Build agents in Agent Builder** (lightweight, inside M365 Copilot Chat) | ✅ Yes — included | ✅ Yes |
| **Build agents in full Copilot Studio** (topics, flows, actions, knowledge sources) | ✅ Yes — *access is included*, but… | ✅ Yes |
| **Publish agents that consume credits beyond the M365 Copilot included usage** | ⚠️ Requires tenant to have a credit source (PAYG / pack / P3) | ✅ Yes |
| **Use premium connectors, custom connectors, Dataverse storage** | ⚠️ Needs Power Platform premium / Dataverse capacity | ✅ Yes (depending on licence tier) |
| **Publish to external channels** (web, WhatsApp, etc.) | ❌ Not included | ✅ Yes |

## Copilot Studio vs Copilot Cowork — Side-by-Side

| Dimension | **Copilot Studio** | **Copilot Cowork** |
|---|---|---|
| **What it is** | Low-code platform to **build** custom agents | Agentic AI **inside M365 Copilot** that completes multi-step work for you |
| **Who uses it** | Makers / developers build it; end users consume it | Individual end users (information workers) |
| **Skill needed** | Low-code, but still technical (topics, knowledge sources, flows) | Natural language only — no build phase |
| **Output** | A **reusable agent** shared across a team or org | A **one-off task result** for the person who prompted it |
| **Best for** | "We need a *thing* that everyone in Comms can ask the handbook" | "I need to prep for my customer meeting / draft my weekly update" |
| **Persistence** | Agent lives on, gets reused, governed centrally | Task runs, returns result, done |
| **Knowledge** | Curated sources you point it at (SharePoint, Dataverse, APIs) | Whatever the user has access to across M365 (mail, files, calendar, Teams) |
| **Actions** | Can call APIs, run flows, update systems autonomously | Mostly read/synthesise/draft within M365 |
| **Governance** | Heavy — environments, DLP, publishing approvals, AI Eng review | Lighter — per-user caps, M365 admin policies |
| **Pricing model** | **Per credit, per feature event** (generative answer = 2, action = 5, etc.) | **Per credit, per task** (Light 70–200, Medium 400–600, Heavy >1,500) |
| **B2E (free for M365 Copilot users)?** | ✅ Yes for interactive use | ✅ Included for M365 Copilot licensed users (with per-user task limits) |
| **Examples in your org** | Comms handbook bot, IT helpdesk bot, Finance policy bot | "Prep me for the Toca review meeting", "Summarise this week's MsgBroker tickets" |
