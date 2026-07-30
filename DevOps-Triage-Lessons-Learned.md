# Lessons Learned — DevOps Triage Automation

> A record of the key architectural decisions, dead-ends, and the pivot that fixed them, from
> building the AI-powered incident triage system at Prostate Cancer UK. Written both as a team
> reference and a personal engineering log.

**Author:** Tommy Bui — Junior Developer / AI Technical Lead, PCUK
**System:** Microsoft Forms / Copilot Studio agent + Power Automate + SharePoint
**Related docs:** `DevOps-Triage-Flow.md`, `DevOps-Triage-Conversational.md`

---

## TL;DR

We built an incident-triage system three ways before landing on the right architecture. The
biggest lesson: **don't nest an AI agent call inside a flow that is itself invoked from an agent
conversation.** It causes an authentication prompt that can't render, and the flow stalls
silently. The fix was to **decouple intake from processing** using an event-driven trigger
(SharePoint "when a file is created"), which returned us to a proven, working pattern and
delivered a cleaner design as a bonus.

---

## The journey (what we tried, in order)

### V1 — Form → Flow → Agent (worked ✅)
- Microsoft Form as the front door.
- Power Automate flow triggered on form submission.
- Flow called the agent ("Run an agent") to classify + diagnose.
- Routed, generated a Word report, uploaded to SharePoint, emailed owners.
- **This worked perfectly** — because there was only ONE agent in the chain, no nesting.

### V2 — Chatbot → Flow → Agent (stalled ❌)
- Replaced the form with a conversational agent front door (better UX).
- Reused the V1 flow as-is — including its internal "Run an agent" action.
- **Result:** the flow stalled every time in Teams. A "Connect to continue" consent card was
  required but could not render inside the agent conversation.
- Root cause: **agent → flow → agent nesting.** The nested agent's connection needed interactive
  consent that has nowhere to surface mid-conversation.

### V2.5 — Move classification into the agent/topic (partially worked, then hit format issues)
- Removed the nested "Run an agent" from the flow.
- Tried to classify using a Prompt tool, then a "Create generative answers" node in the topic.
- **Problem 1:** a rules-only prompt doesn't scale (a new platform like GoFundMe wouldn't be
  recognised) — we needed KB-grounded classification, not hardcoded keywords.
- **Problem 2:** the "Create generative answers" node is built to produce rich prose, and cannot
  be forced to return a single clean category label. `PredCategory` came back as a 300-word
  troubleshooting essay instead of `Toca/AI`, so routing (a SharePoint filter) couldn't use it.

### V3 — Decoupled, event-driven processing (the fix ✅)
- Split the system into **intake** and **processing**:
  - **Intake** (chatbot or form): gather details, attempt self-service, create the report .docx,
    upload to the Dev Incident Reports library. Nothing else.
  - **Processing** (new flow): triggered by **"When a file is created"** in the library. Reads the
    report, calls the agent to classify + produce structured JSON, routes to the owner, emails.
- The agent call now runs in a normal **automated-flow context** (not inside a live conversation),
  so maker-owned connections work and there is no consent prompt. This is the same context V1 ran
  in — the proven working pattern.

---

## Key lessons

### 1. Never nest agent → flow → agent
The single most expensive lesson. If a flow is invoked from within an agent conversation, any
connector inside it that requires interactive authentication (especially the **Agents /
"Run an agent"** connector) will demand a consent card that **cannot render in that context**, and
the flow stalls with no useful error. Keep agent calls in **automated/event-triggered flows**, not
in flows called live from a chat.

### 2. "Create generative answers" is for answers, not classification
It is designed to return rich, grounded prose and will ignore instructions like "respond with only
the label." For a machine-readable single value (needed for routing/filters), use a **Prompt tool**
with a constrained output — or, better, let a downstream processing flow's agent return
**structured JSON**.

### 3. Classification needs KB grounding, not hardcoded rules
Hardcoding routing keywords ("Facebook donations → Toca") doesn't scale — a new platform breaks it.
The agent should **consult the knowledge base** (data-flow docs, error logs, past incidents) to
determine ownership, then fall back to "Unclear" when the KB is genuinely silent. Unknown cases
route to a human, who then adds them to the KB — so the system **improves over time**.

### 4. Decoupling beats coupling
Separating **intake** from **processing** via an event trigger delivered benefits well beyond
fixing the bug:
- One processing engine serves **multiple channels** (chatbot, form, or a manually-created report).
- Intake isn't blocked by processing failures; reports can be **reprocessed**.
- Each half is simpler to build, test, and hand over.
- It's a clean **separation-of-concerns / event-driven** design.

### 5. Connections must be maker/service-account owned
For any flow that runs on behalf of users, set connections to **"use this connection"** (maker or a
dedicated service account), not "provided by run-only user" — otherwise end users get connection
prompts. Prefer a **service account / shared mailbox** over a personal identity so it survives
password/role changes and sends email from an appropriate address.

### 6. When you change the front door, re-review the whole pipeline
The nesting bug was introduced precisely at the form → chatbot switch. Reusing the downstream flow
"as-is" was efficient, but changing the entry point should have triggered a review of whether
downstream components (the internal agent call) were still correctly placed.

---

## Smaller gotchas worth remembering

| Symptom | Cause | Fix |
|---|---|---|
| `Column 'Category' does not exist` | Renaming a SharePoint column keeps the original **internal** name | Use `Title eq '…'` in the filter |
| Expression shows as literal text / `ParameterTypeConversionFailed` | Typed into the plain box, not evaluated | Enter via the **fx Expression** tab |
| `if expects boolean, provided Null` | Wrong path to the agent output | Correct path was `body/structuredOutput/<field>` |
| `invalid reference to 'agentnode'` | Guessed the agent action's internal name | Use the name the dynamic-content picker generates |
| Corrupt uploaded file | Wrong "File Content" in Create file | Use the **Microsoft Word document** output from Populate template |
| Choice variable into a Text input | Type mismatch (Copilot choice vs flow Text) | Convert with `Text(Topic.Urgency)` at the mapping |
| Flow not selectable in topic | Flow still a draft | **Publish** the flow first |
| Trigger loops forever | "When a file is created OR modified" + the flow updates the file | Use **"created" only**, or guard on a Status field |

---

## The final architecture (V3)

```
INTAKE (any channel)                         PROCESSING ENGINE (event-driven)
────────────────────                         ─────────────────────────────────
Chatbot / Form
  → gather details                           Flow 2: "When a file is created"
  → attempt self-service                       in Dev Incident Reports
  → create report .docx                        → read report content
  → upload to library         ───────────►     → Run an agent (classify + JSON,
  → done                                          KB-grounded)
                                               → Get items (route on category)
                                               → update report metadata
                                               → email owner + submitter
```

**Why it works:** the agent runs in an automated flow, not inside a live conversation — no nested
auth, no consent card, proven pattern. Intake and processing are independent and individually
testable.

---

## What I'd do differently next time

1. **Prototype the auth context early.** Test a nested agent call in the *real* deployment surface
   (Teams) before building the whole pipeline around it — not just in the maker's test pane.
2. **Choose the classification mechanism up front** based on whether the output is consumed by a
   human (prose is fine) or a machine (needs structured/constrained output).
3. **Default to decoupling** for any "capture → process → notify" workflow. Event triggers age
   better than tightly-coupled synchronous chains.
4. **Confirm the platform/experience** (classic vs new Copilot Studio) at the start — behaviours
   around variables, topics, and triggers differ enough to change the design.

---

*Lessons-learned log for the DevOps Triage automation. Maintained by Tommy Bui.*
