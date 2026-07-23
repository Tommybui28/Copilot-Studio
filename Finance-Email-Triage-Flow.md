# 📧 Finance Email Triage System — Copilot Studio + Power Automate

An AI-powered email triage flow that automatically **classifies, prioritises, flags, and routes** incoming Finance emails from a shared mailbox into the correct folders — with a manual-review safety net for sensitive cases.

> **Platform:** Microsoft Copilot Studio / Power Automate
> **Environment:** `Copilot - Pilot`
> **Mailbox:** `CopilotStudioTest@prostatecanceruk.org` (shared mailbox)
> **AI Model:** GPT (via AI Builder "Run a prompt")

---

## 📑 Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Prerequisites](#prerequisites)
4. [Build Steps](#build-steps)
5. [The Classification Prompt](#the-classification-prompt)
6. [Parse JSON Schema](#parse-json-schema)
7. [Routing (Switch) Cases](#routing-switch-cases)
8. [Mailbox Folder Structure](#mailbox-folder-structure)
9. [Known Gotchas & Fixes](#known-gotchas--fixes) ⭐
10. [Testing](#testing)
11. [Maintenance Notes](#maintenance-notes)

---

## Overview

Incoming emails to the Finance shared mailbox are analysed by an AI prompt that returns a strict JSON payload containing:

- **Top 3 category predictions** (ranked by confidence) with routing destinations
- **Urgency assessment** (High / Medium / Low)
- **Manual review flag** (complaints, cancellation threats, errors)
- **Financial email evaluation** (is it finance-related + confidence %)
- **Email summary** (1–2 sentence gist)

The flow then **routes** the email based on the Rank 1 destination, or **flags it for a human** if manual review is required.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  TRIGGER: When a new email arrives in a shared mailbox (V2)   │
│  → reads CopilotStudioTest inbox                              │
└───────────────────────────┬─────────────────────────────────┘
                            │ (Subject + Body)
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  RUN A PROMPT (AI Builder)                                    │
│  → classifies email, returns strict JSON                     │
│  ⚠️ MUST run under a LICENSED account (not shared mailbox)    │
└───────────────────────────┬─────────────────────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  PARSE JSON → converts text output into usable fields         │
└───────────────────────────┬─────────────────────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  CONDITION: manual_review/required == true ?                  │
└──────────────┬──────────────────────────┬───────────────────┘
              │ TRUE                      │ FALSE
              ▼                           ▼
   ┌────────────────────┐    ┌──────────────────────────────────┐
   │ Assign category     │    │ SWITCH on Rank 1                  │
   │ "Needs Review"      │    │ routing_destination               │
   │ (stays in Inbox)    │    │  ├─ Case 1..19 → move to folder   │
   │ + notify (optional) │    │  └─ Default   → Unclassified      │
   └────────────────────┘    └──────────────────────────────────┘
```

---

## Prerequisites

| Requirement | Notes |
|-------------|-------|
| Copilot Studio maker access | In the `Copilot - Pilot` environment |
| **AI Builder credits/capacity** | ⚠️ Allocated to the environment — required or prompt returns `403 Forbidden` |
| **Licensed account** for the prompt connection | Premium / AI Builder-enabled — **NOT** the shared mailbox |
| Full Access to the shared mailbox | For routing actions to reach it |
| Outlook categories created | e.g. `Needs Review` must exist in the mailbox |
| Destination folders created | The full folder tree must exist before routing |

---

## Build Steps

### 1. Trigger
- **Action:** `When a new email arrives in a shared mailbox (V2)`
- **Original Mailbox Address:** `CopilotStudioTest@...`
- **Folder:** `Inbox`
- **Importance:** Any
- **Only with Attachments:** No
- **Include Attachments:** Yes *(optional — only useful if you add attachment text extraction downstream)*

### 2. Run a Prompt
- **Action:** AI Builder → `Run a prompt`
- **Connection:** ⚠️ **A licensed work/service account** (see [Gotchas](#known-gotchas--fixes))
- **Inputs:** `Subject` → trigger Subject · `Body` → trigger Body
- **Output:** Text (JSON string)

### 3. Parse JSON
- **Content:** the prompt's `Text` output
- **Schema:** see [Parse JSON Schema](#parse-json-schema)

### 4. Condition (Manual Review)
- **Expression:**
  ```
  body('Parse_JSON')?['manual_review']?['required']
  ```
  `is equal to` `true`
- **True branch:** `Assigns an Outlook category` → `Needs Review` (stays in Inbox) + optional Teams/email notification
- **False branch:** Switch (below)

### 5. Switch (Routing)
- **On (expression):**
  ```
  body('Parse_JSON')?['top_3_categories'][0]['routing_destination']
  ```
- One **Case** per routing destination → move action to matching folder
- **Default:** move to `Unclassified` (never leave empty!)

---

## The Classification Prompt

> Full prompt used in the "Run a prompt" action. Placeholders `[Insert Subject Here]` / `[Insert Body Here]` are replaced by AI Builder **inputs** (`Subject`, `Body`) added via the `/` menu.

<details>
<summary>Click to expand the full prompt</summary>

```text
You are an advanced AI Email Classifier and Triage Assistant for a corporate Finance Department.
Your task is to analyze the Subject and Body of an incoming email, categorize it, detect its
urgency, determine if it requires manual review, assess whether it qualifies as a financial email,
and provide a concise summary.

### INPUT EMAIL TO CLASSIFY:
**Subject:** {Subject}
**Body:** {Body}

### STEP 1: URGENCY DETECTION
- High: Threats of service suspension, final overdue notices, legal/collection warnings,
  same-day deadlines, critical system failures, or urgent executive/management escalations.
- Medium: Standard invoices awaiting processing, routine supplier/customer queries, standard
  onboarding requests, standard month-end activities.
- Low: Informational (FYI), automated non-error notifications, supplier statements for
  reference/audit, newsletters.

### STEP 2: ROUTING & CATEGORY LIST  (provide TOP 3 ranked)
1. AP - Purchase Ledger
   - AP - Purchase Ledger / AP - Research Invoices
   - AP - Purchase Ledger / AP - No PO
   - AP - Purchase Ledger / AP - Approvals   (subfolder Facebook if applicable)
   - AP - Purchase Ledger / AP - Sent to Continia
   - AP - Purchase Ledger / AP - Answered Queries
2. AP - Reference
   - AP - Reference / AP - Supplier Statements
   - AP - Reference / AP - Remittance Advice
3. AR - Income
   - AR - Income / AR - Sales Invoice Requests
   - AR - Income / AR - Income Queries
   - AR - Income / AR - Refunds
4. Payments - Cards & Direct Debits
   - Payments - Cards & Direct Debits / Payments - Credit Cards
   - Payments - Cards & Direct Debits / Payments - Direct Debits
     (subfolders Premier Inn, Addison Lee, Trainline if applicable)
5. Admin & Setup
   - Admin & Setup / Systems - Notifications
   - Admin & Setup / Systems - New User Forms
   - Admin & Setup / Systems - New Supplier
6. Finance - Period End
   - Finance - Period End / Finance - Month End / Year End
   - Finance - Period End / PO close requests
7. External / Procurement Contracts
   - Routing Destination: Contracts Inbox  (Action: Auto-forward to Contracts)
8. Ad-Hoc Management Escalation
   - Routing Destination: Management Escalation

### STEP 3: MANUAL REVIEW ASSESSMENT
Set manual_review_required = true if:
1. Complaint emails (unhappy customers/suppliers, disputes, escalations)
2. Threats to cancel services (terminate contracts, cancel accounts, stop services)
3. Error emails (system bugs, failed payment alerts, technical errors, bounced processes)
Otherwise false.

### STEP 4: FINANCIAL EMAIL EVALUATION
- is_financial_email true/false (AP, POs, expenses, invoices, bank controls, ledgers)
- confidence_percentage 0-100
- financial reasoning

### STEP 5: EMAIL SUMMARY
1-2 sentence summary of the core message/intent/request.

### CRITICAL CONSTRAINTS
- Process-based routing using only the exact folder paths above.
- Do NOT route live incoming emails to Archive (Read-Only) folders.
- Always provide exactly three categories ranked by probability (Rank 1 highest).

### OUTPUT FORMAT
Return STRICTLY valid JSON only — no markdown fences, no text outside the JSON.
{
  "top_3_categories": [
    { "rank": 1, "category": "...", "routing_destination": "...",
      "confidence_percentage": 0, "reasoning": "..." },
    { "rank": 2, "category": "...", "routing_destination": "...",
      "confidence_percentage": 0, "reasoning": "..." },
    { "rank": 3, "category": "...", "routing_destination": "...",
      "confidence_percentage": 0, "reasoning": "..." }
  ],
  "urgency_assessment": { "overall_urgency": "Low", "urgency_reasoning": "..." },
  "manual_review": { "required": false, "reasoning": "..." },
  "financial_evaluation": { "is_financial_email": false,
      "confidence_percentage": 100, "reasoning": "..." },
  "email_summary": "..."
}
```

</details>

---

## Parse JSON Schema

Paste this into the **Parse JSON** action's schema field:

```json
{
    "type": "object",
    "properties": {
        "top_3_categories": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "rank": { "type": "integer" },
                    "category": { "type": "string" },
                    "routing_destination": { "type": "string" },
                    "confidence_percentage": { "type": "integer" },
                    "reasoning": { "type": "string" }
                },
                "required": [ "rank", "category", "routing_destination", "confidence_percentage", "reasoning" ]
            }
        },
        "urgency_assessment": {
            "type": "object",
            "properties": {
                "overall_urgency": { "type": "string" },
                "urgency_reasoning": { "type": "string" }
            }
        },
        "manual_review": {
            "type": "object",
            "properties": {
                "required": { "type": "boolean" },
                "reasoning": { "type": "string" }
            }
        },
        "financial_evaluation": {
            "type": "object",
            "properties": {
                "is_financial_email": { "type": "boolean" },
                "confidence_percentage": { "type": "integer" },
                "reasoning": { "type": "string" }
            }
        },
        "email_summary": { "type": "string" }
    }
}
```

---

## Routing (Switch) Cases

> ⚠️ **Match the `routing_destination` string, NOT the category name.** Cases must match **character-for-character**: regular hyphens `-` (not en-dashes `–`), spaces around every ` / `, and no trailing spaces.

| # | Case `Equals` value | Action → destination |
|---|---------------------|----------------------|
| 1 | `AP - Purchase Ledger / AP - Research Invoices` | Move → AP – Research Invoices |
| 2 | `AP - Purchase Ledger / AP - No PO` | Move → AP – No PO |
| 3 | `AP - Purchase Ledger / AP - Approvals` | Move → AP – Approvals |
| 4 | `AP - Purchase Ledger / AP - Sent to Continia` | Move → AP – Sent to Continia |
| 5 | `AP - Purchase Ledger / AP - Answered Queries` | Move → AP – Answered Queries |
| 6 | `AP - Reference / AP - Supplier Statements` | Move → AP – Supplier Statements |
| 7 | `AP - Reference / AP - Remittance Advice` | Move → AP – Remittance Advice |
| 8 | `AR - Income / AR - Sales Invoice Requests` | Move → AR – Sales Invoice Requests |
| 9 | `AR - Income / AR - Income Queries` | Move → AR – Income Queries |
| 10 | `AR - Income / AR - Refunds` | Move → AR – Refunds |
| 11 | `Payments - Cards & Direct Debits / Payments - Credit Cards` | Move → Payments – Credit Cards |
| 12 | `Payments - Cards & Direct Debits / Payments - Direct Debits` | Move → Payments – Direct Debits |
| 13 | `Admin & Setup / Systems - Notifications` | Move → Systems – Notifications |
| 14 | `Admin & Setup / Systems - New User Forms` | Move → Systems – New User Forms |
| 15 | `Admin & Setup / Systems - New Supplier` | Move → Systems – New Supplier |
| 16 | `Finance - Period End / Finance - Month End / Year End` | Move → Finance – Month/Year End |
| 17 | `Finance - Period End / PO close requests` | Move → PO Close Requests |
| 18 | `Contracts Inbox` | **Forward** → Contracts Inbox address |
| 19 | `Management Escalation` | Move → Management Escalation |
| — | **Default** | Move → Unclassified |

**Watch-outs:**
- #18 destination is literally **`Contracts Inbox`** (not the category name) and uses **Forward**, not Move.
- #19 destination is **`Management Escalation`** (not "Ad-Hoc Management Escalation").

---

## Mailbox Folder Structure

Create this tree in the shared mailbox **before** wiring the routing actions:

```
📥 Inbox
📁 AP - Purchase Ledger
   ├─ AP - Research Invoices
   ├─ AP - No PO
   ├─ AP - Approvals
   ├─ AP - Sent to Continia
   └─ AP - Answered Queries
📁 AP - Reference
   ├─ AP - Supplier Statements
   └─ AP - Remittance Advice
📁 AR - Income
   ├─ AR - Sales Invoice Requests
   ├─ AR - Income Queries
   └─ AR - Refunds
📁 Payments - Cards & Direct Debits
   ├─ Payments - Credit Cards
   └─ Payments - Direct Debits
📁 Admin & Setup
   ├─ Systems - Notifications
   ├─ Systems - New User Forms
   └─ Systems - New Supplier
📁 Finance - Period End
   ├─ Finance - Month End / Year End
   └─ PO close requests
📁 Management Escalation
📁 Unclassified          ← Switch Default target
```

Also create the Outlook **category** `Needs Review` (used by the manual-review branch).

---

## Known Gotchas & Fixes ⭐

The hard-won lessons from building this. Read before debugging.

### 1. `403 Forbidden` on "Run a prompt" 🚨
**Cause:** the prompt connection was set to the **unlicensed shared mailbox** (`CopilotStudioTest`), which has no AI Builder entitlement.
**Fix:** set the Run a prompt connection to a **licensed work/service account**. If it still 403s → the **environment has no AI Builder credits** (check Power Platform Admin Centre → Environment → Capacity).
> 💡 AI Builder credits are **separate** from Copilot Studio message credits.

### 2. "Move email (V2)" can't see the shared mailbox's folders 🗂️
**Cause:** Unlike the trigger, **Move email (V2) has no mailbox parameter** — it acts on the **connected account's own mailbox**, so the folder picker shows *your* Inbox (empty of these folders).
**Fixes (pick one):**
- ✅ **Graph via "Send an HTTP request" (Office 365 Outlook)** — explicitly names the mailbox:
  ```
  POST /v2.0/users('CopilotStudioTest@...')/messages/{Message Id}/move
  Body: { "destinationId": "<folderId>" }
  ```
- Connect the flow **as the shared mailbox** (requires sign-in → licence/conversion).
- Run under a **service account** with Full Access to the mailbox.

### 3. Getting folder IDs (picker won't browse shared mailbox) 🔑
Use the **same Outlook HTTP action** (reuses the working connection) with a **relative URI** and OData `users('address')` syntax:
```
GET /v2.0/users('CopilotStudioTest@...')/mailFolders/inbox/childFolders?$select=id,displayName&$top=100
```
For subfolders, drill in with the parent id:
```
GET /v2.0/users('CopilotStudioTest@...')/mailFolders('{PARENT_ID}')/childFolders?$select=id,displayName
```
> ⚠️ PowerShell/Graph (`Get-MgUserMailFolder`) returned **`404 ErrorItemNotFound / Default folder Root not found`** — because the personal Graph token lacked delegated access to the shared mailbox. The **in-flow HTTP action works** because it reuses the connector's authenticated access.

### 4. `"Sequence contains no elements"` on the Outlook HTTP action ⚠️
**Cause:** wrong URI format — usually passing a **full `https://graph.microsoft.com/...` URL** instead of a **relative path**, or using `users/address` instead of `users('address')`.
**Fix:** relative path + OData parentheses syntax (see #3).

### 5. Condition only offered the whole `manual_review` object 🎯
The dynamic-content picker exposed only the object, not the `required` boolean.
**Fix:** use the `fx` expression directly:
```
body('Parse_JSON')?['manual_review']?['required']
```

### 6. Switch cases silently fall to Default ⚠️
**Causes:** en-dash vs hyphen mismatch, missing spaces around ` / `, **trailing spaces**, or case sensitivity.
**Fix:** copy strings *directly* from the prompt's routing list; strip trailing spaces.

### 7. Parse JSON fails if model wraps output in ```` ```json ```` fences
**Fix:** add a **Compose** before Parse JSON to strip fences:
```
replace(replace(outputs('Run_a_prompt')?['body/text'], '```json', ''), '```', '')
```

### 8. "Invalid connection" / blank prompt dropdown
**Cause:** expired/invalid AI Builder connection token.
**Fix:** re-authenticate the connection (OAuth) with the correct licensed account.

---

## Testing

### Happy-path (routing) test emails

| Target | Subject | Body |
|--------|---------|------|
| Research Invoices | `Research invoice for approval - Project Nightingale` | "Attached research grant invoice for Project Nightingale, ref RES-2291, for processing." |
| No PO | `Invoice INV-4471 - no PO number` | "Invoice INV-4471 from Dell — no matching purchase order, please advise." |
| Approvals | `Invoice awaiting approval - £4,200` | "Supplier invoice needs internal sign-off before processing." |
| Sent to Continia | `Invoice submitted to Continia` | "Confirming invoice INV-8823 has been sent to Continia." |
| Answered Queries | `RE: Query on invoice INV-3310 - resolved` | "Query on INV-3310 now fully resolved. Closing off." |

### Manual-review test emails (should get `Needs Review` category)

| Type | Subject | Body |
|------|---------|------|
| Complaint | `Extremely unhappy with your service` | "Third time you've ignored my invoice query. I am furious and expect an immediate response." |
| Cancellation | `Cancelling our contract effective immediately` | "Due to repeated payment failures, we are terminating our supplier agreement." |
| Error | `SYSTEM ERROR - payment run failed` | "Automated alert: BACS payment batch failed with error 500. No payments processed." |

### Default (no-match) test emails

| Subject | Body |
|---------|------|
| `Lunch order for Friday team meeting` | "Send me your sandwich choices for Friday's team lunch." |
| `Office parking spaces update` | "Car park resurfaced next week, use the side entrance." |

### What to verify in Run History
1. **Run a prompt** → valid JSON, no 403
2. **Parse JSON** → green, fields populated
3. **Condition** → correct branch (`manual_review`)
4. **Switch** → correct case fired (or Default)
5. **Action** → email moved / category applied in Outlook

> 💡 **Debug tip:** temporarily add a **Compose** after Parse JSON showing
> `body('Parse_JSON')?['top_3_categories'][0]['routing_destination']` and
> `body('Parse_JSON')?['manual_review']?['required']` so you can see the exact
> decision values in each run. Delete once stable.

---

## Maintenance Notes

- **Folder IDs are stable** but change if a folder is deleted/recreated — re-run the childFolders HTTP call to refresh.
- **Prompt ↔ Switch coupling:** if you rename a folder in the prompt, update the matching Switch Case value too.
- **Credit usage:** each prompt run consumes AI Builder credits — scales linearly with email volume (~900/week). Monitor capacity.
- **Subfolders** (`Facebook`, `Premier Inn`, etc.) — the AI returns only the parent path; add extra logic if you need subfolder-level routing.
- **Consider a lookup-map** (Compose object mapping `routing_destination` → folder ID) instead of hardcoding IDs across 19 cases — easier to maintain.
- **Production hardening:** run under a dedicated **licensed service account** with MFA/Conditional Access; document in the governance risk register.

---

*Built for PCUK Finance • Copilot Studio Pilot • Maintained by the DevOps team.*
