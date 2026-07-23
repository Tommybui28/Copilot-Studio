# DevOps Triage — Automated Incident Intake, Triage & Reporting

> An end-to-end automation that turns a Microsoft Form submission into an AI-triaged
> incident: the **MsgBroker Assistant** (Copilot Studio agent) classifies the issue,
> attempts a safe resolution, routes unresolved tickets to the right owner, emails the
> submitter, and auto-generates + files a Word incident report into the **Dev Incident
> Reports** SharePoint library.

**Owner:** Tommy Bui — Junior Developer / AI Technical Lead, Prostate Cancer UK
**Platform:** Power Automate (cloud flow) + Copilot Studio agent + Microsoft Forms + SharePoint + Word Online (Business)
**Status:** ✅ Working (triage + routing + emails + report generation)

---

## Table of Contents

1. [What it does](#1-what-it-does)
2. [Architecture](#2-architecture)
3. [Components](#3-components)
4. [The Microsoft Form](#4-the-microsoft-form)
5. [The routing SharePoint list](#5-the-routing-sharepoint-list)
6. [The Copilot Studio agent](#6-the-copilot-studio-agent)
7. [The Power Automate flow — action by action](#7-the-power-automate-flow--action-by-action)
8. [The Word incident-report template](#8-the-word-incident-report-template)
9. [Email templates](#9-email-templates)
10. [Testing](#10-testing)
11. [Gotchas & fixes (read this!)](#11-gotchas--fixes-read-this)
12. [Design decisions](#12-design-decisions)
13. [Future improvements](#13-future-improvements)

---

## 1. What it does

Non-DevOps staff (Data Imports, Fundraising, Marketing, etc.) hit technical issues but
don't know who owns what. This flow gives them a **single front door** (a form) and:

- **Classifies** the issue using the agent's knowledge base (no need for the submitter to
  know the category).
- **Resolves** known, safe issues automatically and emails the submitter the fix.
- **Routes** anything it can't safely resolve to the correct owner:
  - MsgBroker → **Marcus**
  - Toca / AI → **Tommy**
  - DAREN + BA → **Aaliyah**
  - Unclear → **Tommy** (triage fallback)
- **Logs** every submission as a Word incident report in the Dev Incident Reports library —
  resolved *or* escalated — so nothing falls through the cracks.

---

## 2. Architecture

```
Microsoft Form ─► Power Automate cloud flow ─► MsgBroker Assistant (Copilot Studio)
(submitter)       (owns the trigger + glue)     (classify + attempt safe fix)
                                                             │
                                          returns structured JSON
                                                             ▼
                          ┌──────────────────────────────────────┐
                          │  resolved == true ?                    │
                          └──────────────────────────────────────┘
                            │ YES                      │ NO
                            ▼                          ▼
                    Email submitter          Email owner (Marcus/
                    WITH the solution        Tommy/Aaliyah) + email
                            │                submitter "forwarded"
                            └────────────┬─────────────┘
                                         ▼
                    Generate incident report (.docx from template)
                    → Upload to Dev Incident Reports library
                    → Set metadata (Category / Owner / Severity / Status)
                    → Attach report to the outgoing email(s)
```

> **Note:** The trigger is a **Power Automate / Microsoft Forms** trigger, *not* a native
> Copilot Studio trigger. Power Automate is the orchestrator; the agent is the brain it calls.

---

## 3. Components

| Component | Where | Purpose |
|---|---|---|
| Intake form | Microsoft Forms | Front door for non-DevOps staff |
| `DevOps Triage` flow | Power Automate / Copilot Studio | Orchestration |
| `MsgBroker Assistant` | Copilot Studio | Classification + resolution (reuses existing KB) |
| `Incident Routing` list | SharePoint | Category → Owner → Email mapping |
| Incident report template | SharePoint (Templates) | Word template with 16 content controls |
| Dev Incident Reports | SharePoint library | Final store for all generated reports |

---

## 4. The Microsoft Form

Structured fields so the agent gets clean input. Built **first** (the trigger can't select a
form that doesn't exist yet).

| Field | Type | Required | Notes |
|---|---|---|---|
| Your name | Short text | ✔ | Reporter name |
| Your team | Choice | ✔ | Data Imports / Fundraising / Marketing / Finance / Supporter Care / Other |
| Short summary | Short text | ✔ | One line — used for the report filename |
| Detailed description | Long text | ✔ | **The agent's main classification input** |
| Which capability / team / process is affected | Long text | ✔ | |
| Urgency | Choice | ✔ | Low / Medium / High / Critical |
| When did it start / how to reproduce | Long text | ✖ | Extra context |
| Screenshot / attachment | File upload | ✖ | Evidence |

**Settings:** turn on **Record name** (or add an email field) so the flow has a reply-to address.

> **Key design change:** we deliberately **do NOT ask the submitter to pick the issue
> category** — the agent classifies from the description. A non-DevOps user reporting
> "Facebook leads aren't importing" shouldn't need to know that's a MsgBroker issue.

---

## 5. The routing SharePoint list

List name: **`Incident Routing`**

| Column | Type | Notes |
|---|---|---|
| Title *(used as Category)* | Single line of text | ⚠️ Internal name stays `Title` even after renaming — see gotchas |
| OwnerName | Single line of text | e.g. Marcus Anderson |
| OwnerEmail | Single line of text | Plain text, **not** a Person column (easier to drop into "To") |

**Rows** (Category values must match the agent's output **exactly**):

| Title (Category) | OwnerName | OwnerEmail |
|---|---|---|
| MsgBroker | Marcus Anderson | marcus@… |
| Toca/AI | Tommy Bui | tommy@… |
| DAREN/BA | Aaliyah … | aaliyah@… |
| Unclear | Tommy Bui | tommy@… |

---

## 6. The Copilot Studio agent

**Reuse the existing `MsgBroker Assistant`** — do **not** build a new agent. It already has the
curated knowledge base (SQL objects, SharePoint docs). We only:

1. **Added past Dev Incident Reports to its KB** so it can recognise recurring issues.
2. **Called it from the flow** via the *Run an agent* action, returning structured output.

### Agent message (the triage prompt)

```
You are triaging a PCUK DevOps incident. Read the ticket below and use your
knowledge base to classify and (if safe) resolve it.

Decide the OWNER category based ONLY on the description — ignore any guess the
submitter made. Category must be one of: MsgBroker | Toca/AI | DAREN/BA | Unclear.
If genuinely ambiguous, use "Unclear".

If you can confidently and SAFELY resolve it, set resolved = true and fill the
diagnosis fields. NEVER suggest destructive actions (DB writes/deletes,
sp_rename, rerunning production jobs) — anything risky = resolved false, route
to owner.

TICKET
Reporter: {form: name}
Team: {form: team}
Summary: {form: short summary}
Description: {form: detailed description}
Capability affected: {form: capability affected}
When started / repro: {form: when did it start}
```

### Agent response settings

- **Response format:** `Text + JSON`
- **Request human assistance when unsure:** **OFF** (the flow owns escalation — leaving this
  on creates a competing escalation path that stalls the flow).
- **Structured output schema:**

```json
{
  "type": "object",
  "properties": {
    "resolved": { "type": "boolean" },
    "confidence": { "type": "string" },
    "category": { "type": "string" },
    "high_level_action": { "type": "string" },
    "immediate_actions": { "type": "string" },
    "root_cause_category": { "type": "string" },
    "root_cause_other": { "type": "string" },
    "estimated_time_to_fix": { "type": "string" },
    "root_cause_analysis": { "type": "string" },
    "recommendations": { "type": "string" },
    "recurring": { "type": "boolean" }
  }
}
```

> ⚠️ **Critical:** the agent's structured fields are exposed at
> `outputs('Run_an_agent')?['body/structuredOutput/<field>']` — **not** `body/structured` and
> **not** flat `body/`. This tripped us up repeatedly (see gotchas).

---

## 7. The Power Automate flow — action by action

Final order:

```
1.  Trigger: When a new response is submitted (Microsoft Forms)
2.  Get response details
3.  Compose: IncidentRef
4.  Run an agent (MsgBroker Assistant, structured output)
5.  Get items (Incident Routing, filtered by agent category)
6.  Compose: OwnerEmail
7.  Compose: OwnerName
8.  Compose: StatusValue
9.  Compose: RootCauseForReport
10. Populate a Microsoft Word template
11. Create file (Dev Incident Reports)
12. Update file properties
13. Condition: resolved == true
       ├─ TRUE  → Send email to submitter (solution) + attach report
       └─ FALSE → Send email to owner + Send email to submitter + attach report
```

### 2. Get response details
- **Form Id:** the intake form
- **Response Id:** `Response Id` (from trigger)
- ⚠️ Essential — the trigger only gives a response *ID*, not the answers.

### 3. Compose — `IncidentRef`
Expression:
```
concat('INC-', formatDateTime(utcNow(),'yyyyMMdd-HHmmss'))
```
→ e.g. `INC-20260723-143022`. Created early so both branches and the report share one reference.

### 4. Run an agent
- **Agent:** MsgBroker Assistant
- **Message:** the triage prompt (§6), with form fields inserted
- **Request human assistance:** OFF
- **Response:** Text + JSON, with the schema (§6)

### 5. Get items (routing lookup)
- **Site:** HUB-TechnologySolutions
- **List:** Incident Routing
- **Filter Query:**
  ```
  Title eq '<agent category from Run an agent>'
  ```
  ⚠️ Use **`Title`**, not `Category` (internal-name gotcha). Route on the **agent's** category,
  not any form field.

### 6 & 7. Compose — `OwnerEmail` / `OwnerName`
Grab the single matching row without forcing an Apply-to-each loop:
```
first(outputs('Get_items')?['body/value'])?['OwnerEmail']
first(outputs('Get_items')?['body/value'])?['OwnerName']
```
⚠️ Must be entered via the **Expression (fx)** tab, not typed into the plain box.

### 8. Compose — `StatusValue`
```
if(outputs('Run_an_agent')?['body/structuredOutput/resolved'], 'Resolved', 'Escalated')
```

### 9. Compose — `RootCauseForReport`
```
if(outputs('Run_an_agent')?['body/structuredOutput/resolved'],
   outputs('Run_an_agent')?['body/structuredOutput/root_cause_analysis'],
   concat('Escalated to ', outputs('OwnerName'), ' - pending diagnosis'))
```
Gives the report a sensible root-cause value on escalated tickets instead of a blank field.

### 10. Populate a Microsoft Word template
See §8 for the full 16-field mapping.

### 11. Create file
- **Site:** HUB-TechnologySolutions
- **Folder Path:** `/Dev Incident Reports`
- **File Name** (with filename sanitising):
  ```
  concat(outputs('IncidentRef'), '_msgb_',
         replace(replace(<short summary>, '/', '-'), ':', '-'), '.docx')
  ```
- **File Content:** the **Microsoft Word document** output from step 10 (⚠️ not the template file)

### 12. Update file properties
- **Id:** `ItemId` from Create file
- **Category:** agent `category`
- **Owner:** `OwnerName`
- **Severity:** form Urgency
- **Status:** `StatusValue`
- ⚠️ Only works if those columns exist in the library — add them or skip this step.

### 13. Condition — `resolved == true`
- Left: `outputs('Run_an_agent')?['body/structuredOutput/resolved']`
- Operator: is equal to
- Right: `true`

**TRUE branch:** Send email (V2) to submitter with the solution + attach report.
**FALSE branch:** Send email to owner (full details + agent diagnosis) **and** send email to
submitter ("forwarded to {OwnerName}"), both attaching the report.

**Attachments** (Advanced options on Send email):
- Name: `concat(outputs('IncidentRef'), '.docx')`
- Content: **Microsoft Word document** output from Populate template

---

## 8. The Word incident-report template

`MsgBroker_Incident_Report_Template.docx` — mirrors the real Dev Incident Report layout. Every
field is a **plain-text content control** whose Title/Tag = the flow field name.

### Full mapping (Populate a Microsoft Word template)

| Content control | Value | Type |
|---|---|---|
| `IncidentRef` | `IncidentRef` Compose | chip |
| `Status` | `StatusValue` Compose | chip |
| `Date` | `formatDateTime(utcNow(),'dd/MM/yyyy')` | expression |
| `Time` | `formatDateTime(utcNow(),'HH:mm')` | expression |
| `ReporterName` | form: Your name | chip |
| `ReporterTeam` | form: Your team | chip |
| `IncidentDescription` | form: Detailed description | chip |
| `ReporterDetails` | form: Responder's Email | chip |
| `CapabilityAffected` | form: Which capability affected | chip |
| `HighLevelAction` | `…/structuredOutput/high_level_action` | chip |
| `ImmediateActions` | `…/structuredOutput/immediate_actions` | chip |
| `RootCauseCategory` | `…/structuredOutput/root_cause_category` | chip |
| `RootCauseOther` | `…/structuredOutput/root_cause_other` | chip |
| `EstimatedTimeToFix` | `…/structuredOutput/estimated_time_to_fix` | chip |
| `RootCauseAnalysis` | `RootCauseForReport` Compose | chip |
| `Recommendations` | `…/structuredOutput/recommendations` | chip |

(`…` = `outputs('Run_an_agent')?['body/structuredOutput/…']`)

---

## 9. Email templates

**Solution (resolved):**
```
Hi {name},

Thanks for reporting "{summary}".

This looks like a known issue. Here's what's happening and how it's resolved:
{root_cause_analysis}

Recommended actions:
{recommendations}

Logged as {IncidentRef}. If this doesn't resolve it, reply and we'll escalate.
```

**Owner (escalation):**
```
Hi {OwnerName},

A new incident has been routed to you (category: {category}).
Reporter: {name} ({team})  |  Urgency: {urgency}
Summary: {summary}
Description: {description}
Capability affected: {capability}
Agent's preliminary assessment: {root_cause_analysis}
Recurring issue: {recurring}
```

**Forwarded (submitter):**
```
Hi {name},

Thanks for reporting "{summary}". I've forwarded this to {OwnerName}, who looks
after {category}. Logged as {IncidentRef}; they'll follow up.
```

---

## 10. Testing

Run two tickets to exercise both branches.

**TEST 1 — should resolve (TRUE):** a safe, informational "how does this work" question the
agent can answer from the KB (e.g. "Where do skipped Enthuse gifts end up?"). Expect: one
solution email + a report marked *Resolved* with full diagnosis.

**TEST 2 — should escalate (FALSE):** something requiring live investigation (e.g. "Facebook
donations missing this week"). Expect: owner + submitter emails + a report marked *Escalated*
with "pending diagnosis".

**Check in Run history:** agent `resolved`/`category`, Get items returned the right owner,
correct branch taken, report uploaded with correct filename + Status.

---

## 11. Gotchas & fixes (read this!)

These cost real debugging time — documenting so future-you (or the next dev) doesn't repeat them.

| # | Symptom | Cause | Fix |
|---|---|---|---|
| 1 | `Column 'Category' does not exist` in Get items filter | Renaming Title→Category only changes the **display** name; internal name stays `Title` | Use `Title eq '…'` in the Filter Query |
| 2 | Email "To" fails: `ParameterTypeConversionFailed`, shows literal `first(outputs(...))` | Expression typed into the **plain box** → stored as dead text | Re-enter via the **Expression (fx)** tab so it becomes a token |
| 3 | Compose outputs the formula text, not the value | Same as #2 but inside the Compose Inputs | Re-enter the `first(...)` via the fx tab inside the Compose |
| 4 | `if expects boolean, provided value is Null` | Wrong path to `resolved` (`?['structured']?['resolved']`) | Correct path is `body/structuredOutput/resolved` |
| 5 | `invalid reference to 'agentnode'` | Guessed the agent action's internal name | Use the real action name `Run_an_agent` (or whatever the Dynamic content picker generates) |
| 6 | Populate template invalid-expression error | Composes referenced a non-existent action name / wrong path | Fix agent action name + use `body/structuredOutput/…` |
| 7 | Generic `Body` / `body/structured…` chip labels | Cosmetic — expression-backed tokens show generic labels | Hover to confirm the true target; it works at runtime |
| 8 | Create file fails on some submissions | Illegal filename chars (`/ : * ? " < > \|`) in Short summary | Wrap summary in `replace(...)` to strip `/` and `:` |
| 9 | Corrupt/empty uploaded file | Wrong "File Content" in Create file | Must be the **Microsoft Word document** output from Populate template |
| 10 | Agent pauses/escalates on its own | "Request human assistance when unsure" was ON | Turn it OFF — the flow owns escalation |

**Golden rule:** any time you paste `first(...)`, `outputs(...)`, `if(...)`, `formatDateTime(...)`
etc., it **must** go through the **fx Expression tab**. Typing into the normal box stores it as
literal text.

---

## 12. Design decisions

- **Agent classifies, not the submitter.** Non-DevOps users don't know the category, so routing
  is driven by the agent's `category` output — the routing lookup runs *after* the agent.
- **Reuse the existing MsgBroker Assistant.** No second agent — reuses the KB, keeps governance
  and credit usage clean. Only a new topic/action + new KB content (incident reports) were added.
- **Report generation before the Condition.** So the finished document can be **attached** to the
  outgoing emails (an email can only attach a file that already exists).
- **Report runs on both branches.** Resolved *and* escalated tickets get logged — escalated ones
  mirror how requester-raised reports are filed with a "pending diagnosis" note.
- **Routing as a SharePoint list, not hardcoded.** Ownership changes = a row edit, not a redeploy.
- **Destructive-action guardrail.** The agent must never auto-instruct risky actions (DB writes,
  `sp_rename`, rerunning production jobs) to non-DevOps submitters — anything risky escalates.

---

## 13. Future improvements

- **Tighten Facebook routing:** currently the agent classifies Facebook *donations* as MsgBroker;
  the intended rule is *donations → Toca (Tommy)* vs *leads → MsgBroker (Marcus)*. Refine the
  agent's KB/instructions to enforce this distinction.
- **Report link in emails:** move report generation before the emails (done) and include the
  SharePoint "Link to item" so recipients can click straight to the logged report.
- **Critical-urgency Teams ping:** notify the owner in Teams immediately when Urgency = Critical.
- **Deflection metric:** report on Resolved vs Escalated counts for leadership — a clean measure
  of how many tickets the agent deflects.
- **Confidence gating:** only auto-send solutions on `high` confidence; medium/low → escalate.
- **Recurring-issue surfacing:** use the `recurring` flag + related past reports to flag repeat
  faults (e.g. the FacebookLeads schema-drift issue that recurred 7+ times).

---

*Auto-generated handover documentation for the DevOps Triage automation. Maintained by Tommy Bui.*
