# RB-001.3 Credential Exposure Validation

## Objective
Determine whether a user has interacted with a phishing email and whether credentials or sensitive data were exposed.

---

## Inputs

- User identity (email / username)
- Email indicators (URLs, timestamp)
- Email logs
- Identity provider logs (Okta, Azure AD, Google)
- Proxy / web logs
- Endpoint telemetry (optional)
- User confirmation (if available)

---

## Tools

- IdP:
  - Okta / Azure AD / Google Workspace
- Email platform:
  - Message trace / click tracking
- Proxy / network logs:
  - Zscaler, Netskope, etc.
- SIEM / log platform

---

## Steps

### 1. Check Email Interaction (Click Activity)

- Query email security logs:
  - Link click events
  - Time of click
- Validate:
  - Which URL was accessed
  - How many times

---

### 2. Confirm Web Access

- Check proxy / DNS logs:
  - Did user resolve or connect to phishing domain?
- Validate:
  - IP address
  - Timestamp alignment with email delivery

---

### 3. Identify Credential Submission

- Indicators:
  - Access to known credential harvesting page
  - POST requests to suspicious endpoints
- If available:
  - Inspect sandbox results from URL analysis

---

### 4. Analyse Identity Provider Logs

Look for:

- Login attempts shortly after click
- Successful login from:
  - New IP / geolocation
  - Unusual device
- MFA events:
  - Push approvals
  - MFA fatigue patterns

---

### 5. Check for Active Sessions

- Identify:
  - Existing active sessions
  - Token usage across services
- Look for:
  - API usage spikes
  - SaaS access anomalies

---

### 6. User Verification (if needed)

- Contact user:
  - Did they click?
  - Did they enter credentials?
- Cross-check with logs (do not rely solely on user memory)

---

## Decision Points

| Condition | Action |
|----------|--------|
| No click activity | Return to PB-001 Step 5 (retro hunt) |
| Click only (no creds) | Monitor + consider precautionary reset |
| Credentials likely exposed | Immediate account containment |
| Suspicious logins present | Escalate to PB-002 ATO |

---

## Output

- Interaction status:
  - `no interaction / click only / credential exposure`
- Timeline:
  - Email received
  - Click time
  - Login events (if any)
- Risk level:
  - Low / Medium / High
- Affected account(s)

---

## Automation Hooks

- Auto-query click logs
- Auto-correlate proxy + IdP logs
- Auto-flag impossible travel
- Auto-trigger account lockdown workflow

---

## Notes

- Users often underreport interaction—trust logs first
- Even without credential entry, session tokens may be exposed
- Treat all confirmed credential entry as compromised

---

## Related

- Playbook: [PB-001 Phishing](../../playbooks/PB-001-phishing.md)
- Escalation: [PB-002 Account Takeover](../../playbooks/PB-002-ato.md)
- Previous: [RB-001.1 Email Triage](./RB-001-1-email-triage.md)
