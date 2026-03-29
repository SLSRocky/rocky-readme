# TODO.md - Rocky's Task List

Active tasks that need attention. Updated as tasks are added, completed, or blocked.

---

## Pending — Rocky Action

- [ ] **Zoom recording retrieval — easier access** — The download URLs from the Contact Center API require a bearer token, not browser login. Build a solution so Matt can access recordings without needing raw API URLs. Options to evaluate: (1) a Vercel proxy endpoint that accepts a recording ID and streams the file using Rocky's API token, (2) generate short-lived pre-authenticated links, (3) download and store in SharePoint once connected. Prioritize security — recordings contain sensitive client data.

---

- [ ] **Data Risk & Security Decision Log** — Create a formal document (in SLSRocky/rocky-readme) that records all security and data risk decisions made during Rocky's setup. Should include:
  - Each data access connection (what data, what scope, why, what was considered)
  - Risk mitigation decisions and the reasoning behind each choice
  - Rejected approaches and why (e.g. why we chose Application Access Policy for MS365)
  - Recording access decisions (proxy approach, channel restriction to Matt + Ally, etc.)
  - Format: organized by date, readable as a risk assessment document
  - Purpose: audit trail for future compliance/risk review questions

---

## Pending — Matt Action Required

- [x] **Zoom Phone scopes** — `phone:read:list_call_logs:admin` added and confirmed working (2026-03-28). 115 call log records accessible with full caller/callee detail, duration, result, recording/voicemail flags.

- [x] **MS365 connection** — Completed 2026-03-28. App registered, AAP applied, Mail.Send/Mail.Read/Calendars.Read confirmed working.

- [ ] **Ally Stratos introduction** — Create a new Discord channel for a capabilities showcase. Matt to plan what to demonstrate before the introduction.

---

- [ ] **Zoom Team Chat integration — recording delivery channel**
  - Add Zoom Team Chat scopes to Rocky's Marketplace app (`chat_message:write:admin` or equivalent)
  - Create a private Team Chat channel restricted to Matt + Ally only
  - When recording links are requested, post to that channel with an informative label per link:
    - Engagement ID, caller number, date/time (ET), agent name, queue, duration
  - Format: clean message block per recording, link downloads with formatted filename
  - Goal: replace Discord as the delivery channel for sensitive recording links

---

- [ ] **Calendar reminder — Rocky Production Secret expiry** — Once MS365 calendar access is confirmed, add an event to mdugan@slsct.org calendar 24 months from the secret creation date with the reminder: "Re-generate Rocky Production Secret — Azure portal → App registrations → Rocky AI Assistant → Certificates & secrets → New client secret"

---

## Pending — Rocky Action (once dependencies met)

- [x] **Weekly usage report — email delivery** — Enabled 2026-03-28. Monday 8 AM ET cron now emails report to mdugan@slsct.org from Rocky@slsct.org AND commits to GitHub.

- [x] **MS365 email hard limit update** — Updated 2026-03-28. Operation Hard Limits.txt updated with AAP confirmation.

---

## Future / Someday

- [ ] **Push alerts via ntfy.sh** — Set up critical alerting to Matt's cell phone using https://ntfy.sh. Use for things like: system errors, thresholds exceeded (queue wait times, call volume spikes), failed cron jobs, MS365 issues, etc. Lightweight, free, no app account required. Tip from Ben.

- [ ] Connect **LegalServer** (client case management)
- [ ] Connect **monday.com** (project management — Ally uses this)
- [ ] Set up **SharePoint** document storage site

---

## Completed

- [x] Discord channel configured (requireMention: false, typing indicator, no reactions)
- [x] GitHub connected (SLSRocky)
- [x] Zoom Contact Center + Reports API connected (read-only)
- [x] Vercel connected — Contact Center live dashboard deployed
- [x] Two-way SMS via Twilio (860-979-5961 ↔ 860-944-4703)
- [x] Operation Hard Limits.txt created and maintained in GitHub
- [x] README.txt session log created in GitHub
- [x] Nightly README cron (11:50 PM ET)
- [x] Weekly usage report cron (Mondays 8 AM ET) + sample report committed to GitHub
