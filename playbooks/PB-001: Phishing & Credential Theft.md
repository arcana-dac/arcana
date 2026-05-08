# PB-001: Phishing & Credential Theft

## Overview
This playbook outlines the end-to-end response process for phishing incidents, including credential harvesting, malicious attachments, and business email compromise (BEC).

It is designed to:
- Rapidly determine if an email is malicious
- Contain impacted users and accounts
- Identify scope across the organisation
- Prevent follow-on compromise (ATO, malware, data exfil)

---

## Trigger Conditions

Initiate this playbook when any of the following occur:

- User reports suspicious email
- Email security tooling alert (SEG, Defender, Proofpoint, etc.)
- Detection of known phishing indicators (IOCs)
- Reports of credential harvesting pages
- Suspicious inbox rules or login activity post-email

---

## Severity Guidelines

| Severity | Description |
|----------|------------|
| Sev3 | Single user report, no interaction |
| Sev2 | User clicked link or opened attachment |
| Sev1 | Credentials submitted or malware executed |
| Sev0 | Widespread phishing campaign or confirmed org-wide compromise |

---

## Objectives

- Identify whether the email is malicious
- Determine if users interacted (click, submit creds, execute payload)
- Contain compromised accounts or endpoints
- Identify scope across users and systems
- Remove phishing artifacts from environment
- Prevent recurrence

---

## Required Inputs

- Email headers (full)
- Email body (raw + rendered)
- URLs and attachments
- Affected user(s)
- Email logs (O365 / Google Workspace)
- Endpoint telemetry (if interaction occurred)

---

## Playbook Workflow

---

### 1. Initial Triage

📘 Runbook: [RB-001.1 Email Triage & Header Analysis](../runbooks/phishing/RB-001-1-email-triage-header-analysis.md)

- Extract and analyse email headers
- Validate sender authenticity (SPF/DKIM/DMARC)
- Identify spoofing or domain impersonation
- Extract URLs and attachments

**Decision Point:**
- If benign → close with documentation
- If suspicious/malicious → continue

---

### 2. URL & Attachment Analysis

📘 Runbook: [RB-001.2 URL Detonation & Analysis](../runbooks/phishing/RB-001-2-url-analysis.md)

- Detonate URLs in sandbox
- Analyse redirect chains
- Identify credential harvesting pages
- Check attachment hashes and behaviour

**Decision Point:**
- Credential harvesting → go to Step 3
- Malware delivery → escalate to:
  - [PB-003 Endpoint Malware Infection](../playbooks/PB-003-malware.md)

---

### 3. User Interaction Verification

📘 Runbook: [RB-001.3 Credential Exposure Validation](../runbooks/phishing/RB-001-3-credential-check.md)

- Determine if user:
  - Clicked link
  - Submitted credentials
  - Downloaded or executed file

Sources:
- Email click logs
- Proxy logs
- IdP logs
- User confirmation

**Decision Point:**
- No interaction → go to Step 5 (containment-lite)
- Interaction confirmed → continue

---

### 4. Account Containment

📘 Runbook: [RB-001.4 Account Lockdown (IdP/SSO)](../runbooks/phishing/RB-001-4-account-lockdown.md)

- Disable or lock affected account
- Revoke active sessions and tokens
- Reset password
- Enforce MFA re-registration

If suspicious login activity observed:

➡ Escalate to:
- [PB-002 Account Takeover](../playbooks/PB-002-ato.md)

---

### 5. Environment-Wide Hunting

📘 Runbook: [RB-001.5 Phishing Retro Hunt](../runbooks/phishing/RB-001-5-retrohunt.md)

- Search for:
  - Same sender
  - Same subject
  - Same URLs
  - Same attachment hash

Across:
- Email logs
- EDR telemetry
- Proxy logs

**Output:**
- List of all impacted users
- Campaign scope

---

### 6. Email Remediation

📘 Runbook: [RB-001.6 Email Removal & Blocking](../runbooks/phishing/RB-001-6-remediation.md)

- Remove emails from inboxes (tenant-wide)
- Block sender/domain
- Block URLs at:
  - Email gateway
  - Proxy / DNS
- Add detection rules

---

### 7. User Communication

📘 Runbook: [RB-001.7 User Notification](../runbooks/phishing/RB-001-7-user-comms.md)

- Notify affected users
- Provide:
  - What happened
  - Actions taken
  - Required next steps

Optional:
- Org-wide advisory (if campaign is broad)

---

### 8. Post-Incident Activities

📘 Related:
- PIR Template: [templates/pir-template.md](../templates/pir-template.md)
- CAN Report: [templates/can-report.md](../templates/can-report.md)

Actions:
- Document timeline
- Identify detection gaps
- Improve controls:
  - Email filtering
  - User awareness
  - Detection rules

---

## Escalation Paths

| Condition | Escalate To |
|----------|------------|
| Credential theft confirmed | PB-002 Account Takeover |
| Malware execution | PB-003 Endpoint Malware |
| Data access suspected | PB-005 Data Exfiltration |
| Privileged account involved | PB-009 Privilege Escalation |

---

## Outputs

- Incident ticket (completed)
- List of affected users
- Indicators of compromise (IOCs)
- Actions taken (CAN format)
- PIR (if Sev1+)

---

## Automation Opportunities

- Auto-extract URLs and attachments
- Auto-detonate links in sandbox
- Auto-search across tenant (retro hunt)
- Auto-remove phishing emails
- Auto-disable compromised accounts

---

## Notes

- Always assume credential reuse risk if credentials exposed
- Prioritise token/session revocation over password reset alone
- Retro hunting is critical — phishing is rarely isolated

---

## References

- MITRE ATT&CK:
  - T1566 (Phishing)
  - T1204 (User Execution)
- Internal KB:
  - Header analysis guide
  - Phishing detection patterns
