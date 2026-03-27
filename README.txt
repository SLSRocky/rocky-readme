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
2026-03-27 (Session 2 — SMS Fix & README Setup)
====================================================

FIXES & IMPROVEMENTS
- Contact Center dashboard date bug fixed: was caching yesterday's date, now always shows current ET day
  - Added export const dynamic = 'force-dynamic' to Next.js API route
  - New deployment URL: https://zoom-dashboard-gg76va651-slsrockys-projects.vercel.app

- Two-way SMS wake-up fixed:
  - Problem: Rocky's bot was posting the SMS relay message to Discord, which OpenClaw ignores (self-loop protection)
  - Fix 1: Created Discord webhook "SMS Relay" (separate identity) to post inbound SMS
  - Fix 2: Added allowBots: "mentions" to OpenClaw config so webhook mentions trigger Rocky
  - Result: Text 860-979-5961 → Rocky wakes up within seconds and replies to Matt's phone

AUTOMATION ADDED
- Nightly cron job: 11:50 PM ET (03:50 UTC) — Rocky wakes to update this README
  If no work was done that day, Rocky skips adding an entry and goes back to sleep

README LOG
- This file (README.txt) created in SLSRocky/rocky-readme (public repo)
- Auto-updated nightly and at end of each session
