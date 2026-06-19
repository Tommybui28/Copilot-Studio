# Copilot Studio Governance Framework (Draft)

> **Status:** Draft based on internal testing and current Microsoft documentation reviewed on 19 June 2026.
>
> **Important note:** This document intentionally reflects **tested behaviour in our tenant** where that behaviour is more useful for governance than relying on Microsoft documentation alone.

---

## 1. Purpose

This framework defines how Microsoft Copilot Studio should be used in the organisation, including:

- which users can access Copilot Studio, create agents, and publish them
- the role of the **Default environment** versus controlled build environments
- the difference between creating agents in **Microsoft 365 Copilot** and **Copilot Studio**
- why **Copilot Credits** matter and how consumption can be controlled
- which actions require **environment admin** privileges rather than only **Environment Maker** permissions

Microsoft states that Copilot Studio uses Power Platform environments and recommends using a **non-default production environment** for agents intended for production. citeturn1search7turn1search43

---

## 2. Core tested findings for our tenant

The following points are based on real testing carried out in our tenant and should be treated as the working governance baseline.

### 2.1 Default environment is required for Copilot Studio access

Testing showed that when users were removed from the **Environment Maker** role in the **Default environment**, they were no longer able to access Copilot Studio properly, even when they were granted **Environment Maker** in another environment such as a test environment. In practice, the Copilot Studio link stopped working for those users.

This aligns with Microsoft guidance that the **Default environment** is the common environment every employee has access to, and that environment security/governance for maker access must be carefully controlled rather than assuming other environments can fully replace it for first-run access patterns. citeturn1search43turn1search7

**Governance position:**

> Users who need to access Copilot Studio must retain the required maker-level rights in the **Default environment**. Removing all maker access from the Default environment can break Studio access, even where users have permissions elsewhere.

### 2.2 Environment Maker does not equal org-wide sharing authority

Testing also showed that users with **Environment Maker** could create agents but **could not share an agent to the whole organisation**. That action required higher privileges, and only environment admins were able to complete it.

Microsoft documents that **Editor** permissions allow a user to edit, configure, share, and publish content, but that agent sharing can also be limited through **Managed Environments** controls. Microsoft also documents agent sharing rules which allow admins to limit how broadly makers can share agents. citeturn1search8turn1search38

**Governance position:**

> Environment Makers can author agents, but **broad sharing**, especially **org-wide sharing**, must be treated as an admin-controlled activity.

---

## 3. Environment model

## 3.1 Default environment

The **Default environment** must remain part of the Copilot Studio operating model.

Microsoft states that every employee in the organisation has access to the **Default environment**, and that this environment should be secured and governed rather than ignored. Microsoft also recommends enabling **Managed Environments** features and clearly communicating the intended use of the Default environment. citeturn1search43turn1search41

### Default environment policy

- The Default environment is required to support baseline Copilot Studio access for approved makers. citeturn1search43turn1search7
- Users who need to build agents must retain the necessary authoring rights in the Default environment. *(This point is based on internal testing.)*
- The Default environment should **not** be the long-term home for production-grade agents, even though it remains operationally important for access. Microsoft recommends a **non-default production environment** for production agents. citeturn1search7turn1search43
- Governance controls such as **Managed Environments**, **sharing limits**, and **DLP/data policies** should be used to reduce risk in the Default environment. citeturn1search41turn1search38turn1search52

## 3.2 Controlled build / production environment

Agents that are intended for wider use, structured testing, or production deployment should be built and managed in a **separate controlled environment** (for example, a test, pilot, or production environment).

Microsoft states that environments are the place where agents, apps, and flows are stored and managed, and explicitly notes that production agents should use a **non-default production environment**. citeturn1search7

### Controlled environment policy

- Use a separate environment for pilot, test, or production agents. citeturn1search7
- Restrict maker access to approved users only. Microsoft recommends assigning authoring permissions using security roles such as **Environment Maker** only to users who need to author agents. citeturn1search44
- Enable **Managed Environments** features so sharing and governance controls can be enforced. citeturn1search41turn1search38
- Apply **DLP/data policies** to restrict connectors, channels, and data movement where appropriate. citeturn1search52turn1search45

---

## 4. Access model

### 4.1 Minimum practical access rule

Based on tenant testing, users who need to use Copilot Studio to create agents should have:

1. the required maker permissions in the **Default environment**; and
2. the required maker permissions in the approved target environment where agents will be built and managed.

Microsoft guidance confirms that authors need a suitable environment security role such as **Environment Maker** to author agents. citeturn1search44

### 4.2 Authoring role

Microsoft recommends assigning the **Environment Maker** role to users who need to author agents, unless another predefined or custom role is used instead. citeturn1search44

**Governance position:**

- **Environment Maker** = can build agents in that environment. citeturn1search44
- **No Environment Maker (or equivalent)** = user should not be treated as an approved author in that environment. citeturn1search44
- In practice, removing Default environment maker rights can also block portal access entirely. *(Based on internal testing.)*

---

## 5. Sharing and publishing model

### 5.1 What Environment Makers can do

Environment Makers should be treated as **agent authors/builders**.

In practical terms, this means they can:

- create and edit agents in approved environments, where they hold the necessary role. citeturn1search44
- test and manage agent content within the scope of their permissions. citeturn1search8turn1search23

### 5.2 What Environment Makers should not be assumed to do

Environment Makers should **not** be assumed to have unrestricted ability to:

- share agents to the **whole organisation**
- expose agents broadly without admin oversight
- override environment-level governance controls

Microsoft documents that sharing can be constrained through **Managed Environments** and that admins can limit how broadly agents are shared using rules for Editors, Viewers, individuals, and security groups. citeturn1search8turn1search38

### 5.3 Admin role in sharing

Based on tenant testing, **org-wide sharing** should be treated as an **admin-only** action.

This practical approach is consistent with Microsoft documentation showing that sharing governance is centrally controllable and that admins can enforce sharing limits at environment level. citeturn1search38turn1search8

**Governance position:**

> Environment Makers may create agents, but **sharing to the whole organisation** must be reserved for approved admins.

---

## 6. Copilot Studio tenant trial context

This framework assumes the organisation is using the **tenant trial version of Copilot Studio** rather than relying on the **per-user Copilot Studio license** model.

Microsoft documents that a trial gives users access to create agents, but trial restrictions apply. Microsoft also notes that the trial allows users to create agents and test them, while publishing has limitations depending on the scenario. citeturn1search58turn1search59

**Practical governance note:**

- For this framework, the **per-user license** model is intentionally **out of scope**.
- Governance should instead focus on:
  - environment roles
  - admin controls
  - sharing controls
  - runtime credit/capacity behaviour

---

## 7. Microsoft 365 Copilot agents vs Copilot Studio agents

This is a key distinction and should be clearly understood.

## 7.1 Agents created in Microsoft 365 Copilot

Microsoft’s pricing and licensing guidance says that **Microsoft 365 Copilot** includes access to Copilot Studio capabilities for internal agent creation scenarios, and that employee-facing usage scenarios by licensed Microsoft 365 Copilot users can be included without separate billed Copilot Credit charges in those supported internal cases. citeturn1search32turn1search2

### Typical characteristics

- Used mainly for **internal employee-facing scenarios**. citeturn1search2turn1search32
- Built for lighter-weight internal productivity scenarios compared with full Copilot Studio implementations. citeturn1search32turn1search1
- Commonly tied to the user’s authenticated Microsoft 365 identity. Microsoft notes that internal employee-facing usage under Microsoft 365 Copilot is included when the agent runs in the authenticated licensed user’s identity, subject to fair use. citeturn1search2

### Governance interpretation

Use Microsoft 365 Copilot agent creation where the requirement is:

- internal-only
- lower complexity
- lower governance overhead
- no need for broader external publication or advanced platform controls

## 7.2 Agents created in Copilot Studio

Microsoft describes Copilot Studio as the full agent-building platform for supported channels, premium connectors, agent actions, orchestration, and environment-based governance. citeturn1search1turn1search32

### Typical characteristics

- Built inside Power Platform / Dataverse environments. citeturn1search7
- Designed for more advanced use cases, channels, actions, and governance. citeturn1search1turn1search32
- Runtime usage is measured using **Copilot Credits**. citeturn1search57turn1search2

### Governance interpretation

Use Copilot Studio where the requirement is:

- structured build environments
- controlled publishing and sharing
- advanced agent actions / connectors / orchestrations
- production support and admin oversight

---

## 8. Why Copilot Credits are needed

Microsoft describes **Copilot Credits** as the metered currency used to pay for Microsoft Copilot consumption across the tenant. Copilot Credits can be used by Copilot Studio agents and other Copilot workloads. citeturn1search57

Credits are required because runtime activity in Copilot Studio is billed according to what the agent actually does. Microsoft’s billing documentation gives examples such as:

- **Classic answer** = 1 Copilot Credit. citeturn1search2
- **Generative answer** = 2 Copilot Credits. citeturn1search2
- **Agent action** = 5 Copilot Credits. citeturn1search2
- **Tenant graph grounding** = 10 Copilot Credits. citeturn1search2
- **Content processing tools** = 8 Copilot Credits per page. citeturn1search2
- **Voice scenarios** are also metered. citeturn1search2

---

## 9. Credit pricing and purchase options

Microsoft currently documents the following official Copilot Credit purchase options:

- **Pay-as-you-go** = **$0.01 per credit**, billed through Azure. citeturn1search57turn1search36
- **Copilot Credit Capacity Pack** = **$200 per tenant per month** for **25,000 Copilot Credits**. citeturn1search57
- **Copilot Credits Pre-Purchase Plan** = annual prepaid commitment with tiered discounts. citeturn1search57turn1search35

Microsoft also states that unused capacity does **not** roll over to the next month. citeturn1search59

---

## 10. What does and does not consume credits

### 10.1 Activities that consume credits

Credits are consumed when an agent performs billable runtime work such as:

- returning responses
- generating answers
- running actions
- performing grounding
- processing documents/content
- using voice features

Microsoft’s rates documentation and billing FAQ both confirm that active runtime behaviour is what drives credit usage. citeturn1search2turn1search59

### 10.2 Activities that do not inherently consume credits

The following should **not** be treated as billing events by themselves:

- assigning a user to the **Environment Maker** role
- building/configuring an agent without billable runtime usage
- simply granting or removing permissions

The runtime billing model focuses on billable agent activity rather than static role assignment. citeturn1search2turn1search14

---

## 11. Monitoring and controlling usage

### 11.1 Tenant / environment monitoring

Microsoft states that Power Platform admins can monitor Copilot Studio capacity and consumption in the **Power Platform admin center** under **Licensing > Products > Copilot Studio**. The summary views include prepaid capacity, pay-as-you-go usage, and environment-level consumption trends. citeturn1search13

### 11.2 Agent-level monitoring

Microsoft also states that each agent’s **Analytics** page in Copilot Studio can show:

- billed Copilot Credits
- billing trends
- cost distribution
- monthly credit limits / used vs remaining

citeturn1search14

### 11.3 Governance controls to limit usage

To limit unexpected consumption, the organisation should:

- use controlled build/production environments rather than relying only on the Default environment. citeturn1search7
- restrict authoring rights to approved makers only. citeturn1search44
- reserve org-wide sharing to admins. *(Based on internal testing, supported by sharing governance controls.)* citeturn1search38turn1search8
- use **Managed Environments** to control sharing. citeturn1search41turn1search38
- use **DLP/data policies** to restrict non-approved connectors and channels. citeturn1search52turn1search45
- regularly review tenant and per-agent analytics. citeturn1search13turn1search14

---

## 12. Recommended governance model

### Default environment

- Keep the Default environment in scope because it is operationally required for Copilot Studio access in our tested setup. *(Internal tested behaviour.)*
- Do **not** attempt to fully remove all maker rights there if approved users still need Studio access.
- Apply governance controls there rather than trying to eliminate its use entirely. Microsoft recommends securing the Default environment instead of ignoring it. citeturn1search43turn1search41

### Controlled build environment

- Maintain a separate **test/pilot/production** environment for actual agent build and release activity. citeturn1search7
- Limit maker access to approved staff only. citeturn1search44
- Treat org-wide sharing as admin-controlled. *(Internal tested behaviour, aligned with sharing governance controls.)* citeturn1search38turn1search8

### Use-case split

- Use **Microsoft 365 Copilot agent creation** for lower-complexity internal employee scenarios. citeturn1search32turn1search2
- Use **Copilot Studio** for advanced, governed, environment-based agent development. citeturn1search1turn1search7

---

## 13. Short version / executive summary

- The **Default environment is required in practice** for Copilot Studio access in our tenant. Removing maker rights there can block access even if a user has rights in another environment. *(Tested internally; environment governance context supported by Microsoft docs.)* citeturn1search43turn1search7
- **Environment Maker** should be treated as the author/build role, not the full governance/admin role. Microsoft recommends it for authoring permissions. citeturn1search44
- **Org-wide sharing** should be admin-controlled. Tenant testing showed Environment Makers could not share to the whole organisation, and Microsoft provides environment-level sharing controls that support this governance model. citeturn1search38turn1search8
- **Microsoft 365 Copilot** is best for lighter internal agent scenarios, while **Copilot Studio** is the full governed platform for advanced use cases. citeturn1search32turn1search1
- **Copilot Credits** pay for runtime activity such as answers, actions, grounding, and voice. Microsoft documents pricing such as **$0.01 per PayGo credit** and **$200/month for 25,000 credits** in the capacity pack model. citeturn1search57turn1search36

---

## 14. Sources

- Microsoft Copilot Studio licensing guide (June 2026). citeturn1search18
- Copilot Studio licensing. citeturn1search1
- Assign licenses and manage access to Copilot Studio. citeturn1search25
- Work with Power Platform environments in Copilot Studio. citeturn1search7
- Secure the default environment. citeturn1search43
- Managed Environments overview. citeturn1search41
- Limit sharing. citeturn1search38
- Control how agents are shared. citeturn1search8
- Share agents with other users. citeturn1search23
- Secure your Copilot Studio projects. citeturn1search44
- Security and governance. citeturn1search45
- Configure data policies for agents. citeturn1search52
- Manage Copilot Studio credits and capacity. citeturn1search13
- View agent’s billing consumption. citeturn1search14
- Billing rates and management. citeturn1search2
- Copilot Credits overview. citeturn1search57
- FAQ for Copilot Studio billing and licensing. citeturn1search59
- Get access to Copilot Studio. citeturn1search58
- Microsoft 365 Copilot / Copilot Studio pricing page. citeturn1search32
- Copilot Studio licensing guidance. citeturn1search35
- Azure Copilot Studio pricing. citeturn1search36
