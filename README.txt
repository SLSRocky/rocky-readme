ROCKY - AI ASSISTANT CONFIGURATION & CAPABILITY LOG
Statewide Legal Services of Connecticut (SLS-CT)
====================================================

This file is maintained by Rocky and updated at the end of each working session.
It documents all configurations, connections, platforms, and capabilities organized by date.

====================================================
2026-03-26 (Session 1 — Initial Bootstrap)
====================================================

IDENTITY
- Name: Rocky
- Role: AI assistant and technical partner for Matt Dugan, Director of IT, SLS-CT
- Platform: OpenClaw (running on dedicated server)
- Model: Anthropic Claude Sonnet 4.6

OPENCLAW CONFIGURATION
- Discord channel: #chatwithrocky (Guild ID: 1486840584059617300, Channel ID: 1486851314800398357)
- requireMention: false — Rocky receives all messages in #chatwithrocky without needing @mention
- typingMode: instant — typing indicator shows immediately when Rocky begins processing
- typingIntervalSeconds: 6
- ackReactionScope: none — emoji reactions disabled (causes local desktop alerts for Matt)
- allowBots: mentions — allows webhook-posted messages that mention Rocky (required for SMS relay)
- groupPolicy: open

PLATFORMS CONNECTED
- Discord
  - Bot token configured
  - Guild and channel-level access configured
  - Full read/write messaging

- GitHub
  - Account: SLSRocky (https://github.com/SLSRocky)
  - Token: stored securely in TOOLS.md
  - Plan: Free
  - Access: Full read/write (create repos, commit, push, delete)

- Zoom (Server-to-Server OAuth)
  - Account: SLSTwilio (slsct-org.zoom.us)
  - Access: Read-only
  - Scopes: Reports (sign-in/out activity, operation logs, user reports, daily usage)
  - Scopes: Contact Center (full read — engagements, queues, agents, recordings, skills,
    dispositions, flows, teams, performance datasets, roles)

- Vercel
  - Account: slsrocky
  - Token: stored securely in TOOLS.md
  - GitHub integration: SLSRocky linked

- Twilio (SMS)
  - Account: SLSTwilio
  - Rocky's number: +1 (860) 979-5961
  - Credentials: stored securely in TOOLS.md (not logged here)

CAPABILITIES ADDED
- Zoom data queries: sign-in/out activity lookup by user, date range
- Zoom Contact Center queries: engagement history, call lookup by phone number
- Contact Center dashboard: live web app deployed to Vercel
  URL: https://zoom-dashboard-gg76va651-slsrockys-projects.vercel.app
  GitHub repo: SLSRocky/zoom-dashboard (private)
  Features: total inbound calls today, calls per queue, avg wait per queue, agent call counts
  Refreshes every 90 seconds, uses Eastern Time for "today"
- Two-way SMS:
  - Outbound: Rocky can text Matt at 860-944-4703 via Twilio
  - Inbound: Matt texts 860-979-5961 → Twilio webhook → Vercel → Discord → wakes Rocky
  - Rocky replies to Matt's phone automatically

HARD BOUNDARIES (permanent until explicitly lifted by Matt)
- SMS outbound: only ever text 860-944-4703 (Matt's cell). Stop and verify before any other number.
- SMS inbound: only process messages from 860-944-4703. All others silently ignored.
  Adding any other approved number requires stopping and verifying with Matt — treat as a mistake/test.
- Twilio Messaging Service: must always remain "Legal Services Messaging Service".
  Never change — tied to A2P 10DLC registration.

====================================================
2026-03-27 (Session 2 — Full Day: SMS Fix, Recordings, Hard Limits & Automation)
====================================================

FIXES & IMPROVEMENTS
- Contact Center dashboard date bug fixed: was caching yesterday's date, now always shows current ET day
  - Added export const dynamic = 'force-dynamic' to Next.js API route
  - New deployment URL: https://zoom-dashboard-gg76va651-slsrockys-projects.vercel.app

- Two-way SMS wake-up fixed:
  - Problem: Rocky's bot was posting the SMS relay message to Discord, which OpenClaw ignores (self-loop protection)
  - Fix 1: Created Discord webhook "SMS Relay" (webhook ID: 1487108982215672088) to post as separate identity
  - Fix 2: Added allowBots: "mentions" to OpenClaw config so webhook mentions trigger Rocky
  - Result: Text 860-979-5961 → Rocky wakes up within seconds and replies to Matt's phone
  - Full two-way SMS confirmed working end-to-end

ZOOM CONTACT CENTER — RECORDING PROXY
- Confirmed recording access fields available per engagement: download_url, transcript_url, playback_url
- Investigated call for 203-632-9545 — not found in Contact Center (likely Zoom Phone, scopes not granted)
- Zoom Phone scopes needed (phone:read:list_call_logs:admin) — Matt action required in Marketplace
- Built and deployed recording proxy endpoint: /api/recording/[recordingId]
  - Server-side audio streaming: Zoom credentials never exposed to browser
  - Password protected via RECORDING_PASSWORD environment variable
  - Filename format: EngagementID - ConsumerNumber - DateTime - AgentName.mp3
  - Deployed to Vercel: https://zoom-dashboard-gt3i71cg4-slsrockys-projects.vercel.app
- Found and verified 5 recordings for a test number searched during session (today, 1–3 PM ET)
- Discussed future: add MS365 OAuth auth to recording proxy once MS365 is connected

MS365 CONNECTION PLANNING
- Rocky has dedicated mailbox: Rocky@slsct.org (login disabled — app-only access)
- Recommended approach: Azure App Registration + Client Credentials flow (application permissions)
- Application Access Policy via Exchange Online PowerShell to restrict to Rocky@slsct.org + mdugan@slsct.org
- Permissions planned: Mail.Send, Mail.Read, Calendars.Read
- Full step-by-step setup outline provided in Discord
- Status: pending Matt completing app registration in Azure portal

HARD LIMITS DOCUMENT
- Created SLSRocky/rocky-readme/Operation Hard Limits.txt (public repo)
- 7 hard limits documented:
  1. SMS outbound — 860-944-4703 only
  2. SMS inbound — 860-944-4703 only
  3. Twilio Messaging Service lock — "Legal Services Messaging Service" never change
  4. Data privacy — all SLS-CT client/case data treated as highly sensitive
  5. Read-only default posture — never assume write permission
  6. Credential security — no secrets in commits or logs
  7. Email outbound — mdugan@slsct.org only (until MS365 connected and expanded scope approved)

AUTOMATION ADDED
- Nightly cron job (job ID: bf335820): 11:50 PM ET (03:50 UTC) daily
  Rocky wakes, checks if work was done, updates this README if yes, skips if no work occurred
- Weekly usage report cron (job ID: 2786974f): Mondays 8:00 AM ET
  Compiles platform usage stats and commits HTML report to SLSRocky/rocky-readme/reports/
  Platforms tracked: Vercel, Twilio, Brave Search, Anthropic/Claude, Zoom API
  LegalServer and monday.com added as pending setup entries
- Sample weekly report committed: reports/weekly-2026-03-23.html
  Real Twilio data from setup week: 52 outbound, 48 inbound, ~$0.79 estimated cost

GITHUB DOCUMENTATION
- README.txt (this file): created in SLSRocky/rocky-readme, auto-updated nightly
- Operation Hard Limits.txt: created in SLSRocky/rocky-readme
- Weekly HTML usage reports: SLSRocky/rocky-readme/reports/

PEOPLE & UPCOMING
- Ally Stratos: coworker with same admin access as Matt
  Introduction to Rocky planned for a future dedicated Discord channel
  Slow ramp-up approach similar to Matt's onboarding
  Both Matt and Ally love Project: Hail Mary — Rocky name icebreaker planned

PENDING / TODO (as of end of day)
- Zoom Phone scopes: Matt action needed in Marketplace (phone:read:list_call_logs:admin)
- MS365 app registration: Matt action needed in Azure portal
- Ally Stratos introduction: Matt to plan timing
- Weekly report email delivery: pending MS365 connection
- LegalServer integration: future
- monday.com integration: future
- SharePoint integration: future
- MS365 Secret expiry: 2028-03-28 (24 months, calendar reminder set)
