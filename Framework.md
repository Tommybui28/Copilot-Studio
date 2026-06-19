# Copilot Studio Governance Policy

## 1. Purpose

This policy defines how Copilot Studio is governed, including:

- User access and agent creation
- Environment structure
- Sharing controls
- Copilot Credit usage

---

## 2. Governance Principles

- Access to Copilot Studio must be controlled
- The Default environment is required and cannot be removed
- Production agents must not be built in the Default environment
- Org-wide sharing is restricted to admins
- Credit usage must be monitored and controlled

---

## 3. Environment Policy

### 3.1 Default Environment (Mandatory)

- The Default environment is required for Copilot Studio access
- Users must retain **Environment Maker (or equivalent)** in Default
- Removing this access may block access to Copilot Studio *(tenant-tested behaviour)*

**Policy:**
- Do NOT remove all Environment Makers from Default
- Apply governance using:
  - Managed Environments
  - DLP policies
  - Monitoring

---

### 3.2 Controlled Build / Production Environment

- All production/pilot agents must be built outside Default
- Use dedicated environments (e.g. Test / Pilot / Prod)

**Policy:**
- Only approved users assigned **Environment Maker**
- Enforce:
  - Sharing restrictions
  - DLP policies
  - Admin oversight

---

## 4. Roles and Access

### Environment Maker

- Can:
  - Create and manage agents
- Cannot:
  - Share agents to entire organisation

**Policy:**
- Environment Maker = build role only

---

### Admin

- Required for:
  - Org-wide sharing
  - Environment governance
  - Data and connector control

**Policy:**
- Org-wide sharing is admin-only

---

## 5. Sharing Policy

| Action | Environment Maker | Admin |
|--------|-----------------|------|
| Create agent | ✅ | ✅ |
| Share with individuals | ✅ | ✅ |
| Share broadly | ⚠️ Limited | ✅ |
| Share to entire organisation | ❌ | ✅ |

**Policy:**
- Org-wide sharing must be performed by admins
- Sharing behaviour controlled via environment settings

---

## 6. M365 Copilot vs Copilot Studio

### M365 Copilot Agents

- Internal use only
- Lower complexity
- Uses user identity
- Typically no additional runtime cost (internal scenarios)

---

### Copilot Studio Agents

- Full Power Platform / Dataverse solution
- Supports connectors, actions, automation
- Uses Copilot Credits at runtime

---

**Policy:**
- Use M365 Copilot for internal productivity
- Use Copilot Studio for governed or production use

---

## 7. Copilot Credits

### What consumes credits

- Agent responses
- Generative answers
- Actions / workflows
- AI processing

---

### What does NOT consume credits

- Assigning roles (e.g. Environment Maker)
- Building agents without runtime usage

---

### Pricing (reference)

- Pay-as-you-go: $0.01 per credit
- Capacity pack: $200/month (25,000 credits)

---

## 8. Monitoring and Control

Admins must:

- Monitor:
  - Tenant usage (Power Platform Admin Center)
  - Agent usage (Analytics)

- Control:
  - Who can build (Environment Maker role)
  - Who can share (Admin-controlled)

---

## 9. Final Governance Position

- Default environment is required for access
- Environment Maker is a build role only
- Org-wide sharing is admin-controlled
- Copilot Studio usage is consumption-based
- Production agents must be built in controlled environments
