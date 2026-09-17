# Copilot Studio + Cowork — Current Setup & Billing Overview

This document outlines the current configuration of the Copilot Studio pilot environment, Cowork, and associated billing plans, for reference by the development team.

## Administration

- **System Administrator (Copilot pilot environment):** James Nelson
- Role assignments (Basic Users, Environment Makers, etc.) must be requested through James Nelson.

## Pilot Environment Users

- Current pilot users are managed via a security group named **"Copilot Test Users"**.
- This security group is located and managed in the **Admin Center**, owned by James Nelson.

## Billing Plan

- **Plan type:** Pay As You Go
- **Scope:** Currently enabled for **Cowork only** — not yet enabled for Copilot Studio.
- **Cowork credit allocation:** Limited to a selected set of users (not organization-wide).

## Agent Deployment & Management

- Deployment of agents is handled jointly by **James Nelson + Junior Dev**.
- Agents can be:
  - Published
  - Saved
  - Developed
- Management of agents and workflows follows **Aaliyah's AI Agent Request Form** as the governing process.
- Credit allowance allocation is managed in the **Admin Center**, and can be sorted/assigned via **security groups**.
- **Per-user allocation** functionality is **pending** — not yet available, awaiting Microsoft.

## Monitoring Credit & Usage

| Item | Where to view |
|---|---|
| Cowork credit usage | Admin Center |
| Agent usage | Power Platform Admin Center |
| Flow usage | Power Platform Admin Center |

## Notes for Future Developers

To fully support this environment, future developers should familiarise themselves with:

- Locating each of the above areas (Admin Center vs. Power Platform Admin Center) independently.
- Estimated costs **per credit**, **per flow**, and **per agent**.
- Cost/consumption differences across **model usage** (e.g., different LLMs).
- The process for **enabling Anthropic models** for users.
