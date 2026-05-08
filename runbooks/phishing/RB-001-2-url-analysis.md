# RB-001.2 URL Detonation & Analysis

## Objective
Determine whether embedded URLs or attachments in an email are malicious, including credential harvesting, malware delivery, or redirection chains.

---

## Inputs

- URLs extracted from email
- Attachments (if present)
- Email timestamp
- User interaction data (if available)

---

## Tools

- Sandbox:
  - AnyRun / Hybrid Analysis / Joe Sandbox
- URL analysis:
  - URLScan
  - VirusTotal
- Browser (isolated VM only)
- Threat intel platforms
- Proxy logs (optional)

---

## Steps

---

### 1. Normalize & Expand URLs

- Extract all URLs from email:
  - Raw links
  - Obfuscated links (HTML encoded, shortened)
- Expand:
  - URL shorteners (bit.ly, tinyurl, etc.)
- Decode:
  - Base64 / URL encoding if present

**Output:**
- Clean list of URLs for analysis

---

### 2. Passive Reputation Check

Check each URL/domain:

- VirusTotal
- URLScan (previous scans)
- Domain age (WHOIS)

**Indicators:**
- Newly registered domains
- Low reputation / flagged URLs
- Domains mimicking legitimate services

---

### 3. Detonate URL in Sandbox

- Open URL in sandbox environment
- Observe:

  - Redirect chain
  - Final landing page
  - Network calls
  - File downloads
  - Script execution

---

### 4. Analyse Redirect Behaviour

Track:

- Initial URL → intermediate redirects → final destination

**Indicators:**
- Multiple redirects (especially through legit services)
- Redirect to credential harvesting page
- Geo/IP-based conditional redirects

---

### 5. Identify Credential Harvesting

Look for:

- Fake login pages (O365, Google, Okta, etc.)
- Form fields capturing:
  - Username
  - Password
- POST requests to attacker-controlled domains

**Validation:**
- Compare HTML structure vs real login page
- Check domain mismatch

---

### 6. Analyse Attachments (if present)

- Upload to sandbox
- Extract:
  - File hash (MD5/SHA256)
  - Behaviour:
    - Process execution
    - Network calls
    - Persistence

**Indicators:**
- Macro execution (Office docs)
- Droppers / loaders
- Suspicious child processes

---

### 7. Check for Payload Delivery

Determine if URL:

- Directly downloads file
- Triggers drive-by download
- Drops secondary payload

---

### 8. Correlate with Internal Logs (Optional)

- Proxy logs:
  - Did users access final URL?
- EDR:
  - Any file execution tied to URL?

---

## Decision Points

| Condition | Action |
|----------|--------|
| No malicious behaviour | Return to PB-001 Step 1 (reassess) |
| Suspicious but inconclusive | Monitor + proceed to credential validation |
| Credential harvesting confirmed | Proceed to RB-001.3 |
| Malware delivery confirmed | Escalate to PB-003 Endpoint Malware |

---

## Output

- URL classification:
  - benign / suspicious / malicious
- Indicators:
  - Final destination URL
  - Domains/IPs
  - File hashes (if applicable)
- Behaviour summary:
  - Redirect chain
  - Credential harvesting (yes/no)
  - Malware delivery (yes/no)

---

## Automation Hooks

- Auto-expand URLs
- Auto-submit to sandbox
- Auto-enrich with threat intel
- Auto-extract redirect chain
- Auto-flag known phishing kits

---

## Notes

- Always use sandbox or isolated environment—never open directly
- Many phishing URLs are time-limited or single-use
- Legitimate services (Google Docs, Dropbox) are often abused as redirectors
- Credential harvesting pages may only trigger once per session

---

## Related

- Playbook: [PB-001 Phishing](../../playbooks/PB-001-phishing.md)
- Previous: [RB-001.1 Email Triage](./RB-001-1-email-triage.md)
- Next: [RB-001.3 Credential Exposure Validation](./RB-001-3-credential-check.md)
- Escalation: [PB-003 Endpoint Malware](../../playbooks/PB-003-malware.md)
