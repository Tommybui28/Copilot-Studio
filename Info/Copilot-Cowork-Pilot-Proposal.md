# Proposal: Piloting Microsoft Copilot Cowork via Security Group

**Prepared by:** Tommy Bui
**Date:** 28 July 2026
**Status:** Draft for review

---

## 1. Purpose

This proposal outlines a plan to pilot **Microsoft Copilot Cowork** at PCUK by scoping access to a small, controlled **security group**, rather than enabling it tenant-wide. The pilot will run alongside our existing **Copilot Studio trial**, using our current credit allowance to evaluate real-world usage before committing to a permanent billing model.

---

## 2. Background

- We are currently on a **Copilot Studio trial**, which includes a **625,000 Copilot Credit allowance**.
- This trial ends on **16 August 2026**.
- Copilot Cowork is a usage-based Microsoft 365 Copilot feature (an "AI teammate" that executes multi-step tasks — sending emails, building documents, scheduling meetings, posting in Teams — rather than just chatting).
- Cowork is **off by default** at the tenant level and must be explicitly enabled by an admin via a spending policy in the Microsoft 365 admin center.
- **Important:** Enabling Cowork automatically enables **Anthropic as a subprocessor** for our tenant, since Cowork uses Anthropic's Claude models (alongside GPT‑5.5) under the hood. This is a required part of activation — there is no way to enable Cowork without this.

---

## 3. Proposed Approach: Pilot via Security Group

Rather than opening Cowork to **All users**, we will scope it to a **Specific group** using the access group setting in the Copilot spending policy:

1. **Create a dedicated security group** (e.g. `Copilot-Cowork-Pilot`) in Microsoft 365 admin center under *Teams & groups*, with an owner and a defined list of pilot members.
2. In **Admin Center → Copilot → Cost Management → Spending Policy → Edit access groups**, select **Specific groups** (rather than *All users*) and assign the new security group.
3. This ensures only pilot members can consume Copilot Credits via Cowork, giving us full control over exposure and spend during the trial.
4. This mirrors our standard practice of testing new automations in a controlled group before wider rollout (as we've done with the email triage and incident reporting flows).

> Note: the *Specific users* option is currently marked "Coming soon" in the admin UI — group-based scoping is the only method available today.

---

## 4. Funding the Pilot

- The pilot will draw from our **existing 625,000 credit trial allowance** — no additional billing setup is required to get started.
- We will **monitor credit consumption closely** throughout the pilot, tracking:
  - Total credits consumed by the pilot group
  - Average credits per task/user
  - Task types driving the highest consumption (e.g. document generation, browser-based tasks, research tasks)
  - Burn rate relative to the 16 August trial end date

---

## 5. Decision Point: Before Trial Ends (16 August 2026)

Using the usage data gathered during the pilot, we will compare the two available billing models for ongoing Cowork usage and select whichever is more cost-effective for our expected volume:

| Billing Option | Description | When it tends to make sense |
|---|---|---|
| **Pay-As-You-Go (PayGo)** | Billed per Copilot Credit consumed via an Azure subscription (~$0.01/credit) | Lower or unpredictable/variable usage; avoids upfront commitment |
| **Copilot Credit Pre-Purchase Plan (P3) / Capacity Packs** | Bulk credits purchased upfront at a set volume | Consistent, higher-volume usage where bulk pricing reduces per-credit cost |

**Recommendation timeline:** Complete usage analysis by **~11–13 August 2026**, allowing time to raise a recommendation and get sign-off before the 16 August trial cutoff.

---

## 6. Governance & Guardrails During Pilot

- **Monthly spending limit** set on the pilot's spending policy to prevent runaway credit usage.
- **Alerts configured** to notify admins when spend hits defined thresholds.
- **Discoverability left off** for non-pilot users, so Cowork does not appear or get requested outside the pilot group during testing.
- All pilot activity remains **auditable via Microsoft Purview**, consistent with our existing AI governance practices.

---

## 7. Next Steps

1. Confirm pilot group membership and nominate an owner.
2. Create the `Copilot-Cowork-Pilot` security group in the admin center.
3. Enable Cowork scoped to this group via the spending policy (Specific groups).
4. Confirm Anthropic subprocessor enablement is acknowledged/accepted.
5. Run pilot for [X weeks] with weekly credit usage check-ins.
6. Present usage findings and billing recommendation ahead of 16 August 2026.

---

## 8. Open Questions for Sign-off

- Who should be included in the initial pilot group (proposed: AI/automation team + a small number of representative business users)?
- What monthly spending cap should we set for the pilot policy?
- Who owns the go/no-go decision on PayGo vs. Pre-Purchase Plan post-pilot?
