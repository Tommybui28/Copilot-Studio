# Risk Register – Copilot Studio & Cowork Agent Rollout

**Owner:** IT / AI Engineering
**Review cadence:** Quarterly (or on major model/licensing change)
**Rating key:** 🟢 Low | 🟡 Medium | 🔴 High

---

## 1. Cost & Commercial Risks

| ID | Risk | Likelihood | Impact | Rating | Mitigation | Owner |
|---|---|---|---|---|---|---|
| C1 | **Credit overrun via autonomous agents** (25 credits per trigger, always charged even with M365 Copilot licence) | High | High | 🔴 | Cap autonomous agents per environment; mandatory cost forecast before publish; PAYG alerts in Azure | IT / FinOps |
| C2 | **Shadow agent sprawl** – makers spin up PAYG agents in default environment without IT visibility | High | Medium | 🔴 | Restrict Environment Maker role; default environment locked down; monthly inventory audit | IT |
| C3 | **Anthropic / Opus model burns credits faster than GPT** for same task | Medium | Medium | 🟡 | Default to GPT for standard tasks; reserve Opus for coding/deep-reasoning use cases; require AI Engineer sign-off | AI Engineering |
| C4 | **Deep reasoning toggle left on by default** (adds ~10 credits per response) | Medium | Medium | 🟡 | Reasoning off by default; enable only where evidence of poor accuracy; document per-agent | Makers / AI Eng |
| C5 | **Pre-Purchase (P3) credits expire unused** at year-end | Medium | Medium | 🟡 | Start on PAYG for 60+ days to baseline; only move to P3 once usage is predictable | IT / Finance |
| C6 | **Cowork "Heavy" tasks (>1,500 credits each)** stack across a team | Medium | High | 🔴 | Per-user monthly caps in M365 Admin Center; educate users on Light vs Heavy patterns | IT |

---

## 2. Data & Security Risks

| ID | Risk | Likelihood | Impact | Rating | Mitigation | Owner |
|---|---|---|---|---|---|---|
| D1 | **Agent surfaces data the user shouldn't see** (over-permissioned SharePoint sites) | High | High | 🔴 | SharePoint 2.0 containment first; "Restricted SharePoint Search"; agent scoped to specific sites only | IT / SharePoint Admin |
| D2 | **Sensitive content (HR, finance, supporter data) leaked via generative answer** | Medium | High | 🔴 | DLP policies; sensitivity labels enforced; pilot agents read-only; Purview audit | IT / Compliance |
| D3 | **Email triage agent reads PII / supporter data** in Media & PR mailbox | High | High | 🔴 | DPIA before build; data minimisation; retention rules; explicit lawful basis review | DPO / AI Eng |
| D4 | **Anthropic / Opus model data residency outside EUDB** | Medium | High | 🔴 | Verify EUDB compliance per [Microsoft EUDB change log](https://learn.microsoft.com/en-us/privacy/eudb/change-log); disable model if non-compliant | AI Engineering / DPO |
| D5 | **Guest / external users access internal agents** | Low | High | 🟡 | Block guest access in environment security; security group gating on publish | IT |
| D6 | **Credentials / secrets embedded in agent flows or custom connectors** | Medium | High | 🔴 | Use Key Vault / managed identities; code review for custom connectors; no plaintext secrets in topics | Developers / IT |

---

## 3. Model & Accuracy Risks

| ID | Risk | Likelihood | Impact | Rating | Mitigation | Owner |
|---|---|---|---|---|---|---|
| M1 | **Hallucinated answers** from handbook agent (Sadie's "erratic results") | High | Medium | 🔴 | Enable reasoning; tighten knowledge source scope; add "I don't know" fallback topic; user feedback loop | Maker / AI Eng |
| M2 | **Stale knowledge** – SharePoint page updated, agent answers from cached/outdated grounding | Medium | Medium | 🟡 | Documented refresh cadence; owner accountable for source content | Business owner |
| M3 | **Email triage misclassifies urgent PR issue** as low priority | Medium | High | 🔴 | Human-in-the-loop for first 90 days; confidence thresholds; weekly accuracy sampling | Media & PR / AI Eng |
| M4 | **Model deprecation / version change** breaks agent behaviour | Medium | Medium | 🟡 | Subscribe to M365 Roadmap; regression test pack per agent; version-pin where possible | AI Engineering |
| M5 | **Prompt injection** from email content or SharePoint pages | Medium | High | 🔴 | System prompt hardening; instruction isolation; test with known prompt-injection payloads | AI Engineering |

---

## 4. Governance & Operational Risks

| ID | Risk | Likelihood | Impact | Rating | Mitigation | Owner |
|---|---|---|---|---|---|---|
| G1 | **No clear ownership** of published agents (maker leaves org) | High | Medium | 🔴 | Mandatory secondary owner per agent; orphan-agent quarterly sweep | IT |
| G2 | **Agents published org-wide without AI Engineer review** | Medium | High | 🔴 | Publishing pipeline gated; team-wide allowed by maker, org-wide requires AI Eng sign-off | IT / AI Eng |
| G3 | **No visibility of credit consumption per agent** – FinOps blind spot | High | Medium | 🔴 | Power Platform Admin Center analytics; monthly cost report by environment; tag agents by directorate | IT / FinOps |
| G4 | **Users don't know which tool to use** (Copilot Chat vs Cowork vs Studio agent) | High | Low | 🟡 | Single-page decision guide; intranet FAQ; training session per directorate | IT Comms |
| G5 | **Lack of audit trail** for sensitive agent interactions | Medium | High | 🔴 | Purview audit enabled; conversation transcripts retained per policy | Compliance |
| G6 | **AI Builder credits ending Nov 2026** – legacy flows break | Medium | Medium | 🟡 | Inventory all AI Builder dependencies now; migrate to Copilot Credits before deprecation | IT |

---

## 5. People & Change Risks

| ID | Risk | Likelihood | Impact | Rating | Mitigation | Owner |
|---|---|---|---|---|---|---|
| P1 | **Makers build agents they don't understand** the cost/risk of | High | Medium | 🔴 | Mandatory maker onboarding; credit-cost cheat sheet; environment maker training | AI Eng |
| P2 | **Reputational risk** – agent gives wrong info to an external Media/PR contact | Low | High | 🟡 | Internal-only for now; no external publishing without exec sign-off | Comms / IT |
| P3 | **Over-reliance on agents** for decisions that need human judgement (Media & PR triage) | Medium | High | 🔴 | Position agents as assistive only; human approval before action; KPI on human override rate | Sadie / Media & PR |
| P4 | **AI Engineering team becomes bottleneck** for approvals | Medium | Medium | 🟡 | Tiered review (low-risk = lightweight; high-risk = full review); template-based fast-track | IT Leadership |

---

## Top 5 risks to flag to Head of Technology
1. **C1 – Autonomous agent credit overrun** (Media & PR triage build)
2. **D1 – Over-permissioned SharePoint exposure** via agent grounding
3. **D3 – PII handling in email triage agent** (DPIA required)
4. **M3 – Misclassification of urgent PR issues**
5. **G2 – Ungoverned org-wide publishing**

---

## Next steps
- [ ] DPIA for Media & PR email triage agent
- [ ] Cost cap + alerting on PAYG subscription
- [ ] Publishing pipeline documented & enforced
- [ ] AI Engineer review template for new agents
- [ ] Anthropic / Opus enablement decision logged
