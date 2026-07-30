# DevOps Triage — Conversational Intake (Chatbot Front Door)

> A conversational redesign of the DevOps incident-triage automation. Instead of a Microsoft
> Form, staff talk to the **DevOps Assistant** (Copilot Studio agent) in Teams. The bot gathers
> the issue details, attempts a live self-service fix, and only escalates to the DevOps team
> after an explicit **escalation gate**. Once escalated, it reuses the existing triage flow
> (routing → Word report → upload → emails).

**Owner:** Tommy Bui — Junior Developer / AI Technical Lead, Prostate Cancer UK
**Builds on:** the form-based `DevOps Triage` flow (see `DevOps-Triage-Flow.md`)
**Platform:** Copilot Studio (agent + topic) + Power Automate (handoff engine) + SharePoint + Teams

---

## Table of Contents

1. [Why conversational](#1-why-conversational)
2. [Target experience](#2-target-experience)
3. [The four-stage lifecycle](#3-the-four-stage-lifecycle)
4. [Stage detail](#4-stage-detail)
5. [The one change to the existing flow](#5-the-one-change-to-the-existing-flow)
6. [Reusing existing assets](#6-reusing-existing-assets)
7. [Upgrades the conversation unlocks](#7-upgrades-the-conversation-unlocks)
8. [Design guardrails](#8-design-guardrails)
9. [Cost & deployment notes](#9-cost--deployment-notes)
10. [Comparison: form vs chatbot](#10-comparison-form-vs-chatbot)

---

## 1. Why conversational

The form version is one-shot: whatever the user types is what gets triaged, even if it's
half-empty. A conversation can:

- **Ask follow-ups** until it has enough context (no more blank-diagnosis tickets).
- **Attempt a self-service fix live** — many "incidents" are actually questions the bot can
  answer, deflecting them before DevOps ever sees them.
- **Escalate deliberately**, only when a fix failed or isn't applicable, with explicit user
  consent — the conversational equivalent of a "Submit" button, but earned.

The key new concept is the **escalation gate**: a deliberate decision point where the bot stops
helping and hands off to humans.

---

## 2. Target experience

```
User opens the DevOps Assistant in Teams
  → "Hi! Tell me what's going wrong."
  → conversation gathers the details naturally
  → bot tries to fix it live from the knowledge base
  → "Did that sort it?"  →  Yes: done ✅   No: ↓
  → "I'll forward this to the team — okay?"  →  handoff 🚀
  → existing flow logic runs (classify → route → report → email)
```

---

## 3. The four-stage lifecycle

```
1. GATHER
   Bot asks what's wrong → asks clarifying follow-ups
   until it has: Summary, Description, Capability, Urgency

2. ATTEMPT SELF-SERVICE  (the interaction stage)
   Bot searches KB → offers a fix → "Did that resolve it?"
        ├─ YES → close politely, log "Resolved-in-chat" ✅
        └─ NO  → continue

3. ESCALATION GATE  (the decision to forward)
   Bot has enough info AND self-service failed
        → "Shall I forward this to DevOps?" → user confirms

4. HANDOFF
   Call existing flow → classify → two-key gate → route to owner
   → generate report → upload → email owner + user
```

---

## 4. Stage detail

### Stage 1 — Gather (multi-turn, agent-driven)

Don't script rigid questions — set agent instructions + required variables and let generative
orchestration ask naturally.

**Agent instructions:**
```
You are the DevOps intake assistant. Hold a short, friendly conversation to
understand the user's issue before doing anything else. You MUST capture:
 • Summary (one line)
 • Description (what's happening, any error messages)
 • Capability/team/process affected
 • Urgency (Low / Medium / High / Critical)

Ask ONE follow-up at a time. Don't overwhelm. If the user is vague after
2–3 questions, proceed anyway with what you have.
```

**Topic variables captured:** `Summary`, `Description`, `Capability`, `Urgency`
(these map 1:1 to the form fields being replaced, so downstream needs no change).

### Stage 2 — Attempt self-service (the interaction)

1. **Search knowledge / generative answer node** grounded on the KB (SQL objects + past
   incident reports).
2. Present the suggested fix conversationally.
3. **Question node:** "Did that resolve your issue?" → **Yes / No**
   - **Yes** → set `resolved_in_chat = true` → thank + close → optionally log a lightweight
     "Resolved-in-chat" report (great deflection metric).
   - **No** → fall through to the gate.

### Stage 3 — Escalation gate (the "time to forward" moment)

A **Question node** for explicit consent:

> "I wasn't able to fix that myself. Shall I forward this to the DevOps team so someone can
> pick it up?" → **Yes / No**

- **Yes** → proceed to handoff.
- **No** → "No problem — anything else I can help with?" → loop or end.

This confirmation makes the forward **deliberate** and stops the bot escalating trivial
questions the user was just browsing.

### Stage 4 — Handoff (reuse the existing flow)

Add a **Call an action / flow node** that triggers the existing DevOps Triage flow, passing the
captured variables. The flow then runs unchanged:
classify → two-key gate (`resolved` / `confidence`) → route → generate report → email owner + user.

---

## 5. The one change to the existing flow

The current flow triggers on **Forms → new response**. To call it from the bot, swap that for an
**input trigger**:

- Change the trigger to **"When an agent/flow calls this flow"** (Run a flow from Copilot / Power
  Automate input trigger).
- Define **input parameters:**
  `Summary`, `Description`, `Capability`, `Urgency`, `ReporterName`, `ReporterEmail`,
  `ConversationTranscript`
- Everything **after** "Get response details" stays identical — feed these inputs instead of
  reading them from the form.

> The form is not lost. Keep the old Forms-triggered flow as a fallback channel, or retire it.
> The core logic (routing, report, emails) is shared either way.

---

## 6. Reusing existing assets

| Existing asset | Role in the chatbot version |
|---|---|
| MsgBroker Assistant + KB | Powers gathering + self-service |
| Routing SharePoint list | Unchanged — used at handoff |
| Word template + upload | Unchanged — used at handoff |
| Email branches | Unchanged — used at handoff |
| `resolved` / `confidence` two-key gate | Unchanged — runs in the flow |
| **Forms trigger** | **Replaced** by conversation start + flow input trigger |

---

## 7. Upgrades the conversation unlocks

- **Transcript into the report.** Capture the whole conversation (append each turn to a variable,
  or use the built-in summary) and pass it as `ConversationTranscript` → drop it into the report's
  Description. Owners get far richer context than a form field.
- **Better data quality.** The bot won't reach handoff until all four variables are filled — no
  more half-empty tickets.
- **Higher deflection.** Questions get answered in-chat and never become tickets.

---

## 8. Design guardrails

- **Cap the follow-ups** — "if still unclear after 2–3 questions, escalate anyway." Prevents
  frustrating loops.
- **Escape hatch** — "type 'forward' anytime to skip straight to DevOps." Some users just want a
  human.
- **Two-key gate still applies downstream** — "resolved in chat" (Stage 2) is separate from the
  agent's auto-fix `resolved` / `confidence` decision in the flow. Both layers add safety.
- **Never auto-instruct destructive actions** — the original guardrail stays: risky fixes
  (DB writes, `sp_rename`, rerunning production jobs) are escalated, never handed to the user.

---

## 9. Cost & deployment notes

- **Deploy to Teams** for internal staff — authenticated users, and this surface **may qualify for
  the B2E Copilot Credit waiver** (agent credits = 0 for M365 Copilot–licensed users in an
  authenticated M365 context), unlike the Form→Power Automate trigger which is billable.
- **Credit model:** generative answer ≈ 2 credits, KB grounding ≈ 10, agent tool ≈ 5, agent flow
  actions ≈ 0.13 each (1 credit = $0.01). A multi-turn conversation chains several of these, so a
  self-service session may burn ~20–50 credits if billable — verify the B2E waiver applies.
- **Watch the model** — a premium/reasoning model (e.g. Opus) adds the premium AI-tools meter
  (~10 credits per 1K tokens). Test whether it materially improves triage vs a standard model.
- **Start on pay-as-you-go**, gather 60–90 days of telemetry, then convert steady-state to a pack.

---

## 10. Comparison: form vs chatbot

| | Form version | Chatbot version |
|---|---|---|
| Intake | One-shot, fixed fields | Multi-turn conversation |
| Missing info | Whatever the user typed | Bot asks follow-ups |
| Self-service | None | Bot resolves live before escalating |
| Escalation | Automatic on submit | Gated + user-confirmed |
| Data captured | Form answers | Conversation summary + extracted fields |
| Trigger | Forms → new response | Conversation start + flow input trigger |
| Downstream logic | Shared | Shared (unchanged) |

---

*Conversational intake design for the DevOps Triage automation. Maintained by Tommy Bui.*
