# Copilot Studio + Cowork — Current Setup & Billing Overview

## Purpose of This Document

This document is intended as a **handover/reference guide** for the current configuration of the Copilot Studio pilot environment, Cowork, and their associated billing plans. It covers who owns what, how access and credits are managed today, known gaps/pending items, and what the next developer should get up to speed on to take this forward.

---

## 1. Administration & Ownership

| Area | Owner | Notes |
|---|---|---|
| System Administrator — Copilot pilot environment | **James Nelson** | Single point of contact for role assignments (Basic User, Environment Maker, etc.) |
| Agent deployment (publish/save/develop) | **James Nelson + Junior Dev** | Joint responsibility — no self-service publishing outside this pairing currently |
| Agent/workflow request intake | **Aaliyah** (via AI Agent Request Form) | Governs what gets built and prioritised |

**Action for next dev:** Get added as a named contact alongside James Nelson early on, so there's no single point of failure on admin tasks.

---

## 2. Pilot Environment & Users

- Pilot users are managed as a single **security group: "Copilot Test Users"**.
- This group lives in the **Admin Center** (owned/managed by James Nelson).
- Adding/removing pilot users is done by editing group membership — there is currently **no self-service request process** for end users to join; requests should route through James Nelson.

**Things worth clarifying/documenting further:**
- Whether there's a formal process (or just informal requests) for getting added to "Copilot Test Users".
- Whether membership changes take effect immediately or require a licence/role sync delay — worth testing and noting here, as this trips people up.

---

## 3. Billing Plan

- **Plan type:** Pay As You Go (PAYG)
- **Scope today:** Enabled for **Cowork only** — **Copilot Studio itself is not yet on a billing plan**. This means Copilot Studio usage/publishing may currently be riding on trial/included capacity rather than metered PAYG — worth confirming with Microsoft/finance before scaling usage.
- **Cowork credit allocation:** Currently restricted to a **limited, named set of users** rather than everyone in the pilot group. Worth keeping a running list of who has been granted credits and why, since this isn't yet manageable per-user natively (see below).

**Action for next dev:** Confirm with finance/procurement what triggers Copilot Studio being switched onto a billing plan, and what the cost impact would be — this is likely to happen once the pilot graduates to a wider rollout.

---

## 4. Agent Deployment & Management

- Agents can be **published, saved, and developed** — this flow currently sits with James Nelson + Junior Dev only.
- All new agent/workflow work should be logged and tracked through **Aaliyah's AI Agent Request Form** — this is the governance mechanism to avoid ad-hoc/unapproved agent sprawl.
- **Credit allowance** is allocated in the **Admin Center**, and can currently only be sorted/assigned **at the security group level**, not per individual user.
- **Per-user credit allocation is a pending Microsoft feature** — not yet available. Keep an eye on Microsoft release notes/roadmap (Power Platform release plans) for when this ships, as it will likely change how credits are managed here.

---

## 5. Monitoring Credit & Usage

| Item | Where to view | Notes |
|---|---|---|
| Cowork credit usage | **Admin Center** | Tracks consumption against the PAYG Cowork plan |
| Agent usage | **Power Platform Admin Center** | Per-agent session/message volume |
| Flow usage | **Power Platform Admin Center** | Power Automate flow runs tied to agents |

**Gap to fill in:** There's currently no single dashboard combining Cowork spend + agent/flow usage — worth checking whether a Power BI report or Admin Center export can be set up so cost and usage can be reviewed together, rather than checking two separate portals.

---

## 6. Known Gaps / Pending Items (Watchlist)

- **Copilot Studio not yet on a billing plan** — only Cowork is metered today.
- **Per-user credit allocation** — pending from Microsoft; currently group-level only.
- **No formal onboarding process** documented yet for joining the "Copilot Test Users" group.
- **No unified cost/usage reporting** across Admin Center and Power Platform Admin Center.

---

## 7. Notes for Future Developers

Before taking ownership of this environment, get comfortable with:

- **Navigating both admin surfaces independently:** Microsoft 365 Admin Center (Cowork credits, licensing) vs. Power Platform Admin Center (agents, flows, environments).
- **Cost model fundamentals:** estimated cost per credit, per flow run, and per agent session — get actual figures from Microsoft's current Copilot Studio/Cowork pricing page, as these change.
- **Model usage differences:** cost and behaviour differences between the default model(s) and any premium/alternative models (e.g., enabling **Anthropic models** for users) — confirm licensing prerequisites before turning this on.
- **Governance process:** familiarise yourself with Aaliyah's AI Agent Request Form so new agent work stays tracked and approved.
- **Escalation path:** James Nelson is the go-to for admin/role issues; Microsoft support/FastTrack (if available) for platform-level billing or feature questions (e.g., per-user allocation timelines).

---

## Change Log

| Date | Change | Author |
|---|---|---|
| 2026-09-17 | Initial version documenting Copilot Studio + Cowork setup and billing | Tommy Bui |
