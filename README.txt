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
====================================================
2026-03-29 (Session 4 — MS365, SharePoint, LegalServer DEMO)
====================================================

MS365 — FULLY CONNECTED
- Azure App Registration: Rocky AI Assistant (App ID: 845c0e8f-5f38-455f-af05-6dcab3cf669e)
- Tenant: SLS-CT (56dc725a-74b1-4552-8c18-b403d5a22364)
- Client secret expiry: 2028-03-28 (calendar reminder set in mdugan@slsct.org)
- Permissions: Mail.Send, Mail.Read, Calendars.Read (application permissions, AAP-restricted)
- Mailbox scope group: rocky-ai-access@slsct.org (Rocky@slsct.org + mdugan@slsct.org only)
- Calendar event created: "ACTION REQUIRED: Re-generate Rocky AI Production Secret" — March 28, 2028, 9 AM ET, 7-day advance reminder
- Test email from Rocky@slsct.org confirmed working
- Weekly usage report cron updated to email mdugan@slsct.org on Mondays

SHAREPOINT — CONNECTED
- Site: Matt & Rocky (https://slsctorg.sharepoint.com/sites/MattRocky)
- Site ID: slsctorg.sharepoint.com,a2d8aa22-7c08-4c77-b271-a8e63e4bfd4c,4d8e0e54-9487-4efe-8ad5-97d6aada7547
- Access: Sites.Selected (write to MattRocky site only — scoped, not tenant-wide)
- Security: Private M365 group, hidden from address book, search indexing disabled
- Credential drop workflow: Matt drops credentials in SharePoint → Rocky reads + saves → Rocky deletes file
- Credential drop template created: _credential-drop-template.txt in Documents library

LEGALSERVER DEMO — CONNECTED
- Site: https://slsct-demo.legalserver.org
- API User: RockyAPI (Bearer token, 1-year expiry: 2027-03-29)
- Credentials delivered via SharePoint credential drop (file deleted after reading)
- Access: Read-only all matters + write to Case 25-0381304 only
- Confirmed working: v1 + v2 matter search, case note read/write, note editing via UUID
- Total records in demo site: 376,418 matters

LEGALSERVER TESTING
- Read all 5 notes on Case 25-0381304 (Morticia XXXAdams) ✓
- Edited case note subject via UUID ✓
- Restored original subject + appended Program Disposition Code line ✓
- Discovery: Program Disposition Code not exposed in v1 or v2 API after case is reopened
- Research finding: Generic Outgoing API Block on Close Case form could capture it at close time
- date_opened filter NOT supported in v1/v2 search API — requires Reports API

HARD LIMITS ADDED
- Limit #8: LegalServer writes — DEMO: Case 25-0381304 only | LIVE: Case 25-0383515 only (when connected)
- Limit #9: All data reports/dashboards must be behind an auth wall (SharePoint or email).
  Dashboard requests require a conversation about data sensitivity first.

PENDING / TODO (as of end of day)
- LegalServer Reports walkthrough: Matt to introduce report IDs
- Program Disposition Code automation: design + build
- LegalServer live site connection: pending demo validation
- ntfy.sh integration: noted (Ben's suggestion, future)

====================================================
2026-03-30 (Session 5 — RTC Weekly Reports Pipeline, LegalServer LIVE, Mail.ReadWrite)
====================================================

PROJECT: RTC WEEKLY REPORTS — BUILT & AUTOMATED
- Full automated ETL pipeline built: Zoom Contact Center API → SQLite → 3 chart reports → SharePoint upload → draft email
- Variable logs endpoint (variable_logs) confirmed working for RTC Hotline call flow data
- Zoom API cap handling implemented: re-pull in 4-hour windows if any day returns 10,000 records
- Timezone handling: Zoom returns UTC; pipeline converts to ET (EDT UTC-4, EST UTC-5, DST-aware)
- Business hours filter: 8:50 AM–4:45 PM ET, excluding 12:00–1:00 PM lunch (matches RTC Hotline screener schedule)

CHART SPECS LOCKED IN (Call Flow Chart)
- External Inbound: single full-width bar (dark gray #404040), total count in white centered
- Inbound / Vet xFer / Non-Area / Eligible: diverging bars — English right (blue #2E75B6), Spanish left (orange #ED7D31)
- X-axis: symmetric, range = max language-stage value × 1.25 per side
- Title, legend, total calls text box (light yellow, bottom-right), figure 13×6 white background
- On Hold: permanently omitted (feature phased out, always zero)

AUTOMATION ADDED
- Weekly cron (job ID: c1f7d2df): every Monday 7:00 AM ET
  Pulls prior Mon–Sun data, builds 3 charts, uploads to SharePoint, creates draft email in Rocky@slsct.org
  First automated run: Monday April 6, 2026 (will cover March 30 – April 5)
- Monthly cron (job ID: 5a0dce0e): first Monday of each month, 7:00 AM ET
  Same pipeline, full prior-month date range, DST-aware split within month
  First automated run: Monday April 6, 2026 (will cover March 2026)

EMAIL TEMPLATE CONFIRMED
- Subject: RTC Hotline Data Reports - [start date] - [end date]
- To: Jan Chiaretto, Aletheia Stratos, Jason Becker, Alexis Smith, Jamey Bell, JKelleher@ctlegal.org, Giovanna Shay, Anne Louise Blanchard
- CC: Angela@ctbarfdn.org, Emma@ctbarfdn.org, Carolyn@ctbarfdn.org
- From: MDugan@slsct.org (draft only — Matt reviews and sends)

BUG FOUND IN EXISTING MACRO (RTCReport-Macro.txt)
- Location: "CONVERT TO EST" section; Line: FilterTime.Formula = "=I1-TIME(5,0,0)"
- Bug: Zoom portal exports are already in ET — macro subtracts 5 extra hours, shifting filter window to ~1:50 PM–9:45 PM ET
- Impact: All historical RTC reports have understated morning call counts (stages 2–5 roughly halved)
- Rocky's automated pipeline uses the CORRECT filter (bug fixed in new pipeline)
- Action required: Matt to manually fix macro and re-run historical reports

CONFIRMED DATA (week of 3/23–3/29, with corrected filter)
- Total External Inbound: 506 | Inbound: Eng=341, Esp=36 | Vet xFer: Eng=16, Esp=0
- Non-Area: Eng=48, Esp=2 | Eligible: Eng=159, Esp=19 | On Hold: 0/0 (omitted)

MS365 — PERMISSION EXPANDED
- Mail.ReadWrite scope added (in addition to existing Mail.Send, Mail.Read, Calendars.Read)
- Required for draft email workflow: Rocky creates draft in Rocky@slsct.org inbox → Matt reviews and sends

LEGALSERVER LIVE — NOT YET CONNECTED
- Site: https://slsct.legalserver.org
- Status: Pending — awaiting demo site validation before connecting
- Write restriction when connected: Case 25-0383515 only
- Note: Only the DEMO site (slsct-demo.legalserver.org) is currently connected

SHAREPOINT
- Security-Privacy-Decisions.txt created in MattRocky SharePoint site
  Decision #1 logged: phone numbers stripped from SharePoint/email-delivered reports (privacy)
  Cron scheduled to audit full history and add all decisions organized by date (10 PM)

PENDING / TODO (updated)
- Matt to fix RTCReport-Macro.txt and re-run historical RTC reports
- TownIndex (~TownIndex.xlsx) in SharePoint: add zip 06516 (West Haven) and 06514 (Hamden) as Eligible
- LegalServer Reports walkthrough: Matt to introduce report IDs
- Program Disposition Code automation: design + build
- Zoom Phone scopes: Matt action needed in Marketplace (phone:read:list_call_logs:admin)
- Ally Stratos introduction: Matt to plan timing

====================================================
2026-03-31 (Session - RTC Reports Pipeline Complete + SLS Analysis)
====================================================

RTC HOTLINE REPORTS - FULLY AUTOMATED
- Weekly cron: every Monday 7 AM ET -> covers previous Mon-Sun
- Monthly cron: first Monday of month 7 AM ET -> covers previous calendar month
- Both pipelines: cap-handled ETL, 3 PDF charts, SharePoint upload, draft email in MDugan inbox
- First automated runs: Monday April 6, 2026
- Historical backfill: 12 weeks (12/29/25-3/22/26) completed with corrected time filter
- Charts: Call Flow (diverging bars), Eligible Zip Codes (pie), Non-Eligible by Town (bar)
- API cap handling: if any day = 10k records, auto re-pulls in 4-hour UTC windows then deletes temp data

BUG FOUND - EXCEL MACRO (RTCReport-Macro.txt)
- Bug: macro subtracts TIME(5,0,0) from portal timestamps already in ET (double-shift)
- Effect: filter applied to 1:50 PM-9:45 PM instead of intended 8:50 AM-4:45 PM
- All historical reports understated stages 2-5 by approx 2x
- Fix: change "=I1-TIME(5,0,0)" to "=I1" in CONVERT TO EST section
- Rocky's pipeline uses the corrected filter

SHAREPOINT - ORGANIZED
- Folders created: RTC Hotline Reports/Weekly, RTC Hotline Reports/Monthly,
  RTC Historical Reports, SLS Main Hotline Reports, RTC Reference Files
- Report Menu: Rocky - Report Menu.txt (root of MattRocky SharePoint site)
- Security-Privacy-Decisions.txt updated with Decisions #1-5

SLS MAIN HOTLINE - LEGAL ISSUE SELECTION REPORT (SLS-1)
- First-tier legal issue pie chart built (Housing 56%, Family 20%, etc.)
- March 2026 chart in SharePoint -> SLS Main Hotline Reports/
- 16-month chart (Dec 2024-Mar 2026) in progress as background job

TOWNINDEX GAPS / NON-ELIGIBLE COUNT DISCREPANCY
- 67 zip codes in TownIndex have no routing_type (Hartford, Bridgeport, Stamford, etc.)
- Macro mis-classifies these as Non-CT -> inflated count (577) vs Rocky's IVR-based count (50)
- Root cause: Rocky trusts IVR routing signal; macro trusts TownIndex lookup
- Resolution pending: Matt reviewing RTC flow JSON before TownIndex update
- West Haven (06516) and Hamden (06514) patched in Rocky's pipeline

HARD BOUNDARIES UPDATED
- LegalServer LIVE: confirmed NOT yet connected (DEMO only - corrected)
- Email: draft-only permanently, never auto-send
- Temp data (/tmp/rtc/): deleted after every pipeline run (Security Decision #2)

PENDING
- Matt to review RTC Hotline flow JSON -> update TownIndex
- LegalServer LIVE connection (pending demo validation)
- Staff onboarding + offboarding checklists


=======================================================
2026-04-01 (Session 6 — SLS Main Hotline 16-Month Chart, LegalServer Demo Search, reMarkable Attempt)
=======================================================

SLS MAIN HOTLINE - LEGAL ISSUE SELECTION REPORT (COMPLETE)
- 16-month chart (Dec 2024 – Mar 2026) finalized and uploaded to SharePoint
  File: SLS Main Hotline - Legal Issue Selections - 1224 - 0326.pdf
  Location: SharePoint -> SLS Main Hotline Reports/
  Total unique calls: 40,235 that reached legal issue selection
- Chart specs locked: pie chart with % label per slice + call count table below
  Legend brackets show option number (e.g. "Housing [1]"), no footer/watermark
  Data: first-tier selections only (0=Operator excluded)

LEGALSERVER DEMO - SEARCH CAPABILITY CONFIRMED
- Searched demo site for client "Harold" -> found Harold Bennett (25-0381271)
- New Haven, DOB 01/16/1952, veteran, disabled, income eligible, LSC eligible
- Case summary PDF generated and uploaded to SharePoint root
- LegalServer DEMO search capability confirmed working end-to-end

REMARKABLE INTEGRATION - ATTEMPTED (BLOCKED)
- Attempted connection using unofficial cloud API
- One-time code generated from my.remarkable.com but all known endpoints returned 404
- reMarkable appears to have migrated API infrastructure; community docs are outdated
- Research cron scheduled (job id: 48fda8af): fires 10 PM ET to research current working
  endpoints from GitHub/community sources; findings to be posted in Discord before retry
- Status: BLOCKED pending endpoint research results

DISCORD SESSION ISSUE
- Main Discord session (b3fa17ec) became unresponsive ~17:42 UTC (API rate limited,
  840 messages / 2.7MB context)
- Matt switched to webchat to diagnose; /reset issued in Discord to clear context

SECURITY DECISION #6 - 24-HOUR RETENTION FOR ACTIVE REPORTS
- For long-running pulls under active review: data may be held up to 24 hours
- PII still stripped at ingestion; logged in Security-Privacy-Decisions.txt in SharePoint

PENDING / TODO (updated)
- reMarkable integration: wait for research cron results, retry with confirmed endpoint
- Historical RTC PDF backfill (12 weeks): cron job b020acd1 errored; needs re-run
- Matt to review RTC Hotline flow JSON -> update TownIndex
- LegalServer LIVE connection (pending demo validation)
- Staff onboarding + offboarding checklists
==============================================================================
2026-04-02 (Session 7 — AssetPanda Integration)
==============================================================================

PLATFORM CONNECTED: ASSETPANDA
- API base: https://api.assetpanda.com/v2
- Bearer token stored securely in tools/assetpanda.md (may expire; regenerate from AP UI)
- Credentials delivered via SharePoint credential drop (file deleted after read)
- Entity IDs mapped: Assets (208780), Offices (208781), Categories (208782),
  Employees (208783), Software Licenses (208784), Rooms (208785), Inventory (209396)

PROJECT: TECHNOLOGY EQUIPMENT IMPORT (AssetPanda)
- Source: Technology Equipment Inventory-FINAL.xlsx (292 asset records)
- Field mapping locked: Barcode #, Description, Category, Model, Serial #,
  Dell Service Tag, Dell Express Service Code, Assigned Staff, Location, Status,
  Purchase Date, Cost, Purchase From, Funding Source
- Status mapping locked: Available, Assigned to Employee, Not In Use,
  Out for Repair, Disposed
- Office mapping: Staff Offsite -> Remote Location; all others -> SLS Main Office
- Name fixes applied: Aletheia Stratos, Janice Chiaretto, Jason Becker, Wendy Vazquez

IMPORT METHOD: CSV (not API) — changed mid-session
- Discovery: AssetPanda depreciation sub-fields (date_placed_in_service,
  period_of_life, salvage_value, original_cost) do NOT save via API — silently ignored
- Solution: generate CSV matching AssetPanda's import format; import via UI
- CSV columns include: Old SLS Tag #, Asset ID, Category, Description, Model,
  Serial #, Dell Service Tag, Dell Express Service Code, Current Employee,
  Current Office, Current Room, Status, Purchase From, Purchase Date, Cost,
  End of Life Date, Date Scanned, Date Placed in Service, Original Cost Basis,
  Total Months of Life (60), Salvage Value (0)

TEST IMPORT RESULTS (20 records via AssetPanda UI CSV import)
- 17 imported successfully
- 3 skipped — blank Asset ID (barcode missing), Status=Disposed
- Decision: exclude blank-barcode rows from full import

EXCLUSION RULES (FINAL)
- Barcode = 9999 -> exclude (old "discard" marker)
- Match in September 2025 - Junked Assets.csv -> exclude
- Blank barcode -> exclude (can't set unique Asset ID)
- Total excluded: ~54 rows; Total to import: ~238 records

FIXED ASSETS JOIN
- Sheets "Computers" AND "Furniture" both parsed for purchase data
- Join key: Barcode # in inventory <-> BARCODE # in fixed assets
- original_cost = Cost; date_placed_in_service = Purchase Date
- Total Months of Life = 60; Salvage Value = 0; Funding Source = LSC

OTHER WORK
- Zoom user count confirmed: 43 active users via API
- Calendar appointment created: "Test Staging Pipeline" Thu Apr 2 at 8:45 AM ET

STATUS
- Test import (17/20 records) pending staff verification at SLS
- Once approved: generate full 238-record CSV, upload to SharePoint, Matt imports via UI
- Script: /home/aiadmin/.openclaw/workspace/assetpanda_import.py (API version, keep for reference)
- Probe records to delete from AP UI: DEPR-PROBE, DEPR-PROBE-2, DEPR-TEST

PENDING / TODO (updated)
- Staff to verify 17 test import records in AssetPanda
- Matt to update Excel files: fix duplicate barcode 9999, add missing laptop purchase data
- Matt to clear test records from AssetPanda UI before full import
- Generate and upload full import CSV once files are updated
- reMarkable integration: research cron (job 48fda8af) fired overnight — check results

==============================================================================
2026-04-07 (Session 8 — RTC Unified Pipeline, LegalServer LIVE, AssetPanda Full Import)
==============================================================================

LEGALSERVER LIVE — CONNECTED
- LIVE bearer token provided by Matt and validated for read access against the v1 API (HTTP 200)
- Observed LIVE matter count: approximately 386,218
- SharePoint RTC Hotline Reports folder populated with LegalServer reference files:
  - Case Referral API URL
  - Case Demographics API URL
  - Project LS Reports

RTC REPORTING — UNIFIED AUTOMATION BUILT
- New durable local script created: rtc_automation/rtc_unified_pipeline.py
- Unified pipeline now handles end-to-end RTC reporting:
  - 3 Zoom RTC PDFs
  - 1 LegalServer RTC Referrals & Demographics PDF
  - SharePoint uploads
  - Outlook draft email creation
- Weekly test run for 2026-03-30 to 2026-04-05 completed successfully end-to-end
- Monthly March 2026 rerun started using the unified pipeline
- Durable plan confirmed: replace prior split RTC logic with the unified pipeline and repoint weekly/monthly cron jobs to it

LEGALSERVER RTC PDF CAPABILITY ADDED
- Built local Python reporting pipeline: project-ls-reports/build_ls_reports.py
- Standard RTC Referrals & Demographics PDF spec locked in:
  - Filename pattern: SLS RTC Referrals & Demographics Report - MMDD - MMDD.pdf
  - Plain professional format, no cover page, no methodology page, no footer
  - Raw LFxferOffice values preserved as columns
  - Household size and number-under-18 cross-tabs bucketed as 0, 1, 2, 3, 4, 5+
  - Unknown row removed from both bucketed cross-tabs
  - Page subtitle uses Report Run:
- Included cross-tabs locked in:
  - LFXferOffice x Case Status
  - LFXferOffice x Ethnicity
  - LFXferOffice x Gender
  - LFXferOffice x Total Household Size
  - LFXferOffice x Number of People under 18
- Weekly LegalServer PDF for 2026-03-30 to 2026-04-05 generated locally and uploaded to SharePoint
- Going forward, Matt wants this LegalServer PDF included in weekly and monthly RTC report emails

SHAREPOINT / MS365 AUTH
- SharePoint site permissions confirmed working; upload failures traced to Graph token issuance using a stale client secret
- New MS365 client secret provided by Matt and validated by successful SharePoint upload
- Future requirement identified: persist current MS365 / SharePoint app credentials in durable secure config storage for future sessions

RTC MACRO PATCH
- Patched RTCMacro-04072026.txt so non-eligible calls now reference RTC-NonArea-ENG / RTC-NonArea-ESP instead of relying on TownIndex.xlsx for non-eligible classification
- Patched file uploaded to SharePoint: RTC Hotline Reports/RTCMacro-04072026.txt
- Validation note: inferred non-eligible queue naming should still be checked in Excel

ASSETPANDA — FULL IMPORT COMPLETED
- Durable local builder created: assetpanda_build_full_csv.py
- Script workflow now:
  - downloads source files from SharePoint
  - applies exclusion rules
  - joins fixed-asset purchase data onto inventory by barcode / Asset ID
  - outputs a true AssetPanda UI import CSV matching the approved test template
  - uploads the corrected CSV back to SharePoint
- Important correction: earlier full-import CSV format was wrong (JSON-payload style rather than AssetPanda UI import format); corrected process is now locked in
- Final successful import outcome:
  - 236 rows imported successfully via AssetPanda UI
  - 56 rows excluded by design
- SharePoint deliverable used for successful import:
  - AssetPanda/AssetPanda Full Import - Corrected.csv
- Locked import rules/spec:
  - 26-column CSV matching AssetPanda Test Import - 20 Records.csv
  - Exclude rows where Asset ID = 9999, blank barcode / Asset ID, or barcode appears in September 2025 - Junked Assets.csv
  - Staff Offsite maps to Current Office = Remote Location and Current Room = Staff Offsite
  - Purchase Date, Cost, and Purchase From are joined from 2025 Fixed Asset Records.xlsx when available
  - All rows include Total Months of Life = 60 and Salvage Value = 0
  - For rows missing matched purchase date, Date Placed in Service defaults to 01/01/2008
- Earlier in the session, stale employee names were also corrected in the SharePoint source workbook so matching succeeded:
  - Ally Stratos -> Aletheia Stratos
  - Jan Chiaretto -> Janice Chiaretto
  - Jason Backer -> Jason Becker
  - Wendy Velazquez -> Wendy Vazquez

==============================================================================
2026-04-08 (Session 9 — Durable Credential Standardization, MS365 Persistence Repair, Monday.com Connection)
==============================================================================

MS365 / SHAREPOINT PERSISTENCE REPAIR
- Root cause confirmed: fresh sessions lacked one durable authoritative MS365 credential source; runtime config did not carry Graph credentials and the tooling note was stale
- Created durable local config file: /home/aiadmin/.openclaw/workspace/.openclaw/ms365.env
- Durable config now stores tenant ID, client ID, current client secret, mailbox scope group,
  default mailbox, SharePoint site URL/site ID, drive ID, and TownIndex item ID
- Added shared helper module: tools/ms365_config.py
- Updated tools/ms365.md to reference durable local config instead of embedding credentials
- Repointed active scripts to use the shared durable MS365 config helper:
  - rtc_backfill/run_backfill.py
  - rtc_automation/rtc_unified_pipeline.py
  - project-ls-reports/build_ls_reports.py
  - assetpanda_build_full_csv.py
  - project-ls-reports/build_ls_reports_snapshot.py

MS365 ACCESS VERIFIED
- Graph token issuance: HTTP 200
- Mailbox read for mdugan@slsct.org: HTTP 200
- Calendar read: HTTP 200
- SharePoint site access: HTTP 200
- Calendar event creation: HTTP 201
- App roles present in token:
  - Mail.Read
  - Mail.ReadWrite
  - Mail.Send
  - Calendars.Read
  - Calendars.ReadWrite
  - Sites.Selected
  - User.Read.All
- Deleted temporary verification event: Rocky MS365 persistence test
- Created Matt calendar reminder: call Amanda and confirm
  - 2026-04-09 at 9:00 AM America/New_York
  - 30-minute appointment with 15-minute reminder

CREDENTIAL STANDARDIZATION EXPANDED
- Moved GitHub token into durable local config: /home/aiadmin/.openclaw/workspace/.openclaw/github.env
- Moved Vercel token into durable local config: /home/aiadmin/.openclaw/workspace/.openclaw/vercel.env
- Moved AssetPanda API base, access token, client ID, and client secret into durable local config:
  /home/aiadmin/.openclaw/workspace/.openclaw/assetpanda.env
- Added shared helper module: tools/assetpanda_config.py
- Added shared helper module: tools/legalserver_config.py
- Moved LegalServer demo/live site and bearer tokens into durable local config:
  /home/aiadmin/.openclaw/workspace/.openclaw/legalserver.env
- Updated tools/github.md, tools/vercel.md, tools/legalserver.md, and relevant active scripts
  to reference durable local config rather than inline secrets
- Remaining cleanup note: stale historical secret references may still exist in prior chat/session
  transcripts and cron payload text; active workspace tooling now points at durable config

MONDAY.COM — CONNECTED
- Monday account URL saved: https://sls-ct.monday.com/
- Added durable local config: /home/aiadmin/.openclaw/workspace/.openclaw/monday.env
- Added local Monday tooling files:
  - tools/monday.md
  - tools/monday_config.py
  - tools/monday_lookup.py
  - tools/monday_inventory.json
- Matt's Monday token was briefly stored for first verification, then removed from durable config
  at Matt's request because the token was tied to Matt's account
- Ally Stratos's token then stored in durable local config for ongoing Monday access
- Verified Monday API access under Ally's identity:
  - Aletheia Stratos
  - astratos@slsct.org
- Initial Monday access inventory captured:
  - 11 visible workspaces
  - 100 visible boards in first-pass inventory sample
  - visibility confirmed across Outreach & Education, Chat Navigation, SLS Tech, EITA,
    Pro Bono, Training, Grant Management, Grants management, and Outreach Documents
- Durable production rule approved by Matt: Ally is authorized for both read and write work in Monday.com

MONDAY DOCUMENTATION OUTPUTS
- Generated structured inventory JSON locally: tools/monday_inventory.json
- Generated Monday lookup helper for future queries and raw GraphQL use: tools/monday_lookup.py
- Generated PDF inventory report: Monday Access Inventory - Ally Stratos.pdf
- Uploaded PDF to SharePoint Documents/Monday folder:
  Monday Access Inventory - Ally Stratos.pdf

SECURITY NOTE
- Both Matt and Ally pasted Monday tokens into Discord during setup
- Tokens should be treated as potentially exposed and rotated when practical

SHAREPOINT — ALLYROCKY CONNECTED
- New dedicated SharePoint collaboration site connected for Rocky + Ally:
  - Site: AllyRocky
  - Access model: Rocky read/write, Ally read/write
  - Sensitivity posture: same as MattRocky; safe for sensitive internal / client-capable information
  - Delivery rule: organizational-data outputs remain SharePoint/email only; no public links
- Rocky app (`Rocky AI Assistant`) was granted site-scoped write access via Microsoft Graph under `Sites.Selected`
- Access verified end-to-end:
  - Site resolved successfully via Graph
  - Default document library confirmed: Documents
  - Site ID confirmed: slsctorg.sharepoint.com,782e0b3b-ddb6-43d9-9b01-8477e603a1c4,4d8e0e54-9487-4efe-8ad5-97d6aada7547
  - Drive ID confirmed: b!OwsueLbd2UObAYR35gOhxFQOjk2HlP5OitWX1qradUfjPgSW4Zz2TqkP86QNqBtC
- Minimal write test completed successfully, then cleanup performed:
  - Created `Rocky Access Test.txt`
  - Deleted the test file after verification

====================================================
2026-04-09 (Operational Hard Limits, Ally SMS approval, Discord health-monitor change)
====================================================

TWILIO / SMS BOUNDARY EXPANDED
- Matt approved Ally Stratos's cell phone number `774-994-0064` as an additional allowed SMS contact
- Updated `Operation Hard Limits.txt` so Rocky may use that number for:
  - outbound SMS
  - inbound SMS
- Hard-limits wording updated from Matt-only SMS access to an approved-number model
- First Ally intro text was successfully sent through the approved Twilio Messaging Service

MS365 ACCESS SCOPE PREPARED FOR ALLY
- Verified Exchange-side Application Access Policy includes Ally mailbox access for Rocky's app
- Confirmed `Test-ApplicationAccessPolicy` granted access for `astratos@slsct.org`
- Live Graph mailbox/calendar calls from Rocky were still blocked at session time, consistent with Microsoft propagation/cache delay
- Added an automated OpenClaw cron recheck to test Ally mailbox and calendar access again after propagation

OPENCLAW / DISCORD CONFIG CHANGE
- Updated OpenClaw config: `channels.discord.healthMonitor.enabled: false`
- Purpose: reduce suspected duplicate Discord reply behavior by disabling Discord-only health-monitor auto-restarts
- Gateway restart was initiated so the new Discord health-monitor setting could take effect

DOCUMENTATION / HANDOFF OUTPUTS
- Created a polished Ally intro handout summarizing Rocky's connected systems, capabilities, and hard guardrails
- Delivered review drafts to Matt's mailbox, including revised PDF versions for the 2026-04-10 Ally introduction

====================================================
2026-04-10 (Session 10 — Ally WordPress Wiki Integration and First Echo KB Workflow)
====================================================

ALLY CHANNEL / COLLABORATION MODEL
- In the dedicated Ally Discord channel, Matt instructed Rocky to treat messages there as coming from Ally unless something clearly indicates otherwise, while still verifying sensitive or write approvals when needed
- Ally introduced a WordPress-based resource wiki workflow spanning four Echo Knowledge Base post types:
  - `epkb_post_type_1` = wiki (staff)
  - `epkb_post_type_2` = SLSBoard
  - `epkb_post_type_3` = PB Attorney
  - `epkb_post_type_4` = Rocky Test

WORDPRESS / ECHO KNOWLEDGE BASE DISCOVERY
- Confirmed WordPress REST API availability at `https://wiki.slsct.org/wp-json/`
- Confirmed standard WordPress endpoints and custom Echo Knowledge Base post types are exposed via REST metadata
- Determined that public KB item endpoints returned empty arrays without authenticated/plugin-context access, so the live workflow would need to follow the actual Echo KB admin process rather than public REST alone
- Confirmed the wiki is managed through Echo Knowledge Base plus related Echo plugins, including custom link article behavior and category-specific workflows

SHAREPOINT / PDF HANDOFF
- Generated a PDF memo for Ally summarizing WordPress wiki integration recommendations
- Uploaded the memo to the AllyRocky SharePoint site for Ally’s use

TASK-SPECIFIC APPROVAL AND MAILBOX ACCESS
- Matt explicitly approved Rocky, in the Ally Discord channel, to access Ally’s mailbox for the `Document for WIKI` task and upload the attached document to the Rocky Test knowledge base
- Using Microsoft Graph application access, Rocky successfully searched Ally’s mailbox, found the email from Matt, and retrieved the attached `Wiki Directions.docx`

FIRST LIVE ECHO KB WORKFLOW TEST
- Logged into WordPress successfully as the `rocky` user and validated updated permissions for Rocky Test after Ally changed the role/capability settings
- Confirmed the needed Echo KB workflow for PDF-style resources is:
  1. upload the file to the WordPress Media Library
  2. copy the file URL
  3. create a `Custom Link Article` in the target knowledge base
  4. assign the appropriate category (for this test: `PDFs`)
- First created a test Rocky Test article titled `Wiki Directions` linked to the uploaded DOCX file
- Then corrected the workflow to match wiki requirements by converting the DOCX to PDF locally, uploading `Wiki-Directions.pdf` to the Media Library, and updating the Rocky Test custom link article to point to the PDF instead
- Final linked file path used by the Rocky Test article:
  - `http://wiki.slsct.org/wp-content/uploads/2026/04/Wiki-Directions.pdf`

NEW OPERATIONAL RULES FROM ALLY WIKI WORK
- If a source document is already a PDF, upload it directly
- If a source document is not a PDF, convert it to PDF first and upload the PDF version instead before creating the KB link article
- Never delete WordPress articles or pages in the Echo Knowledge Base workflow
- User/page access permissions inside Echo KB should not be changed as part of routine document posting work

IMPLEMENTATION NOTES
- This workflow currently runs as a hybrid:
  - Microsoft Graph tokens/app credentials for mailbox access
  - local Python scripting for document extraction and PDF conversion
  - local scripted WordPress web-session workflow for Media Library upload and Echo KB custom link article creation
- The successful first end-to-end Ally wiki task established a repeatable baseline for future document posting into Rocky Test and, by extension, the other Echo Knowledge Base sections once permissions and categories are confirmed

MONDAY.COM — DAILY CALL SUMMARY BUILD (ALLY)
- Matt approved Rocky to create a monday.com tracking structure in the `SLS Tech` workspace for Zoom Contact Center daily metrics
- Created folder: `Daily Call Summary`
- Created boards:
  - `Agent Stats`
  - `Queue Stats`
- Initial board design was tested and then iterated with Ally in real time

AGENT STATS BOARD
- Kept as a flat daily log structure
- Daily rows continue to represent per-agent daily totals from the Zoom dashboard source

QUEUE STATS BOARD — REDESIGNED WITH ALLY
- Ally revised the preferred layout after seeing the initial structure
- Final working direction:
  - one group per queue
  - parent item per week, labeled like `Week of Apr 6, 2026`
  - one subitem per day
  - subitems hold the actual daily values
  - parent items are intended to summarize the subitems via monday formulas
- Confirmed Queue Stats subitem board columns now use numeric storage for:
  - `Calls Per Queue`
  - `Average Wait Seconds`
- Daily queue data for 2026-04-10 was written into the subitems successfully

AUTOMATION / SYNC WORK
- Built local updater script: `/home/aiadmin/.openclaw/workspace/monday_daily_call_summary_sync.py`
- Built cron wrapper: `/home/aiadmin/.openclaw/workspace/cron_monday_daily_call_summary.sh`
- Configured weekday 6 PM ET cron schedule for the sync job
- Sync uses the existing Zoom dashboard API as the data source:
  - `https://zoom-dashboard-gg76va651-slsrockys-projects.vercel.app/api/zoom-data`
- Token-minimal design used:
  - one dashboard API fetch
  - minimal monday reads for board/item lookup
  - only necessary monday writes

KNOWN MONDAY LIMITATION / FORMULA ISSUE
- Ally created parent formula columns on Queue Stats for weekly summary values
- Inspection showed the monday formula definitions were pointing at each other instead of the subitem numeric columns
- Rocky confirmed the correct intended logic:
  - `Total Calls` = SUM of subitems `Calls Per Queue`
  - `Average Wait Seconds` = AVERAGE of subitems `Average Wait Seconds`
- Rocky also confirmed the monday API path available in-session could populate the subitems but could not reliably rewrite the parent formula definitions, which appear to be handled client-side / UI-side for this board setup
- Ally is handling the monday UI formula configuration; Rocky will continue populating the daily subitems at 6 PM on weekdays

LEGALSERVER REOPEN AUTOMATION TEST
- Matt approved additional DEMO-site write testing on case `25-0381369` for Program Disposition Code reopen automation
- Built and deployed Vercel webhook app: `legalserver-pdc-reopen`
- Live endpoint:
  - `https://legalserver-pdc-reopen.vercel.app/api/reopen-hook`
- Configured the LegalServer `ReOpen` process Generic Outgoing API Block to POST JSON containing:
  - `caseId`
  - `matterUuid`
  - `programDispositionCode`
  - `reopenedAt`
  - optional `reopenedBy`
- Implemented shared-secret verification via `x-rocky-secret` header stored in Vercel environment configuration

LEGALSERVER API FINDINGS AND CAPABILITY ADDED
- Confirmed that the ReOpen Generic Outgoing API Block does expose Program Disposition Code at reopen time, making reopen-time capture a viable automation path
- Correct LegalServer note API paths/fields established for this workflow:
  - read: `GET /api/v2/notes?matter_uuid=<uuid>`
  - update: `PATCH /api/v2/notes/<note_uuid>`
  - note identifier: `note_uuid`
- Updated note-selection logic to target only the newest `Reopened Case` note by actual timestamp
- Changed the automation to append the captured Program Disposition Code line to the note body, not the subject
- Final appended line format now uses Eastern Time formatting, for example:
  - `Program Disposition Code captured at reopen: 180 Caller Didn't Pursue/No Adv Con | reopenedAt=04/10/2026 11:54:47 AM EDT`

TEST CLEANUP / STATUS
- LegalServer notes for this workflow cannot be deleted through the available API, so prior test notes on case `25-0381369` were preserved and archived by renaming their subjects
- Final DEMO test succeeded: the newest `Reopened Case` note on case `25-0381369` now receives the Program Disposition Code line in the note body after reopen
- Matt requested that future coding work default to Claude when available; an attempted Claude ACP handoff failed because Claude local auth/CLI was not available yet

====================================================
2026-04-13 (Session 11 — RTC Cron Repair, Demographics Rule Fix, Claude ACP Restored)
====================================================

RTC AUTOMATION / CRON REPAIR
- Diagnosed why the Monday RTC weekly draft did not appear: both active RTC cron jobs had Discord announce delivery configured without a required recipient target
- Repaired weekly RTC cron `c1f7d2df-4dc3-4569-b794-6811600e4d51` and monthly RTC cron `5a0dce0e-756a-4e1b-8805-760dc8f6d1af`
- Set the required Discord delivery target to:
  - `channel:1486851314800398357`
- Manually reran the weekly RTC flow for `2026-04-06` through `2026-04-12`
- Verified a new Outlook draft was created in `MDugan@slsct.org` with subject:
  - `RTC Hotline Data Reports - April 6, 2026 - April 12, 2026`
- Confirmed the weekly SharePoint uploads completed successfully
- Resolved a runtime import blocker during rerun by supplying the workspace `PYTHONPATH`

RTC REPORTING LOGIC / OUTPUT CHANGES
- Moved the RTC Call Flow legend from lower-right to lower-left in all active generators to prevent overlap with chart content
- Updated all active report generators:
  - `rtc_automation/run_weekly_rtc_reports.py`
  - `rtc_automation/run_monthly_rtc_reports.py`
  - `rtc_automation/rtc_unified_pipeline.py`
- Investigated a March LegalServer demographics mismatch in `Number of People under 18`
- Confirmed the source total was 94 while the report showed 91 because 3 records had blank / N/A `family_under_18` values that were being bucketed as `Unknown`
- Per Matt's instruction, changed the reporting rule so blank / N/A under-18 values are treated as `0` going forward
- Verified the corrected March under-18 totals now match the source cross-tab:
  - `0=62, 1=17, 2=8, 3=6, 4=1, 5+=0` (total 94)
- Identified the 3 affected March case IDs:
  - `26-0402886`
  - `26-0412884`
  - `26-0409660`
- Regenerated the weekly RTC draft set successfully with the formatting change
- A March monthly rerun was started again after one interrupted attempt; next session should confirm final completion state before assuming the monthly draft finished cleanly

CLAUDE ACP / OPENCLAW CAPABILITY RESTORED
- Restored one-off Claude ACP execution inside OpenClaw
- Installed missing ACP runtime locally:
  - `acpx@0.5.2`
- Installed the Claude ACP adapter package locally:
  - `@zed-industries/claude-agent-acp@0.21.0`
- Confirmed the adapter requires host-side `ANTHROPIC_API_KEY` for non-interactive runs
- Pulled the updated Anthropic key from the MattRocky SharePoint site (`Documents/Claude Token.txt`)
- Stored the key in local sensitive env-file storage:
  - `/home/aiadmin/.openclaw/workspace/.openclaw/anthropic.env`
- Wired `ANTHROPIC_API_KEY` into the OpenClaw user systemd gateway service and restarted it
- Verified successful one-off Claude ACP execution with result:
  - `CLAUDE ACP TEST OK`

CLEANUP NOTE
- Current working state places `ANTHROPIC_API_KEY` directly in the OpenClaw user systemd service for reliability
- Recommended follow-up: move that secret reference to a cleaner env-file pattern and remove the raw key from the unit file

====================================================
2026-04-16 (Session 14 — Family report workflow locked in, LegalServer LIVE reporting validated)
====================================================

LEGALSERVER LIVE REPORTING
- Validated a working LegalServer LIVE reporting path using export `load=6823`
- Confirmed Rocky can pull and analyze a constrained LIVE export for Family reporting work
- Important field limitation documented for future report logic:
  - export exposes `last` and combined `name`
  - no clearly separate first-name field was present in the tested export

CAPABILITIES ADDED
- New durable report workflow added: `LS-1 — Family Case Data Request`
- Rocky can now generate the Family case report from the validated LegalServer LIVE export logic and produce both:
  - dated Excel workbook output
  - matching PDF summary output
- Local Family report builder updated to use the current run date dynamically instead of a hardcoded cutoff date
- SharePoint report catalog updated to include `LS-1 — Family Case Data Request` with the workflow, formatting, and storage location documented

DELIVERABLES / OPERATING STATE
- Generated refreshed outputs for `2026-04-16`:
  - `family-case-report-2026-04-16.xlsx`
  - `family-case-data-request-2026-04-16.pdf`
- Uploaded both final outputs to the MattRocky SharePoint site
- Saved the deduped 490-case comparison list used during validation to SharePoint for reference

====================================================
2026-04-20 (Session 15 — RTC weekly run, Funnel Stage 2 logic investigation)
====================================================

RTC REPORTING
- Weekly RTC reporting run for `2026-04-13` through `2026-04-19` completed successfully after re-running the unified pipeline with the workspace `PYTHONPATH` set correctly
- SharePoint uploads completed for all four weekly report PDFs and a matching Outlook draft was created for Matt
- No new external platform connection was added, but the current durable RTC reporting path was confirmed working for this cycle:
  - `rtc_automation/rtc_unified_pipeline.py`

FUNNEL REPORTING / STAGE 2
- Funnel Stage 2 local logic was refined for investigation work and durable reruns
- Created checkpointed helper scripts and a saved checkpoint file so long Zoom `variable_logs` pulls can resume reliably during slow runs and transient `502` failures
- Important reporting rule established for future Stage 2 reconciliation work:
  - day bucketing must use `America/New_York` local-day boundaries rather than UTC midnight windows
- Important source-of-truth finding established:
  - Zoom export/filter results do not fully match the `contact_center/variable_logs` API for SLS Stage 2, so export-based workflows may be required when exact workbook matching matters

====================================================
2026-04-15 (Session 13 — TenacitOS dashboard evaluation and full rollback)
====================================================

TENACITOS DASHBOARD EVALUATION
- Rocky evaluated TenacitOS as a possible OpenClaw dashboard for Matt
- Confirmed host compatibility for an install attempt:
  - OpenClaw reachable and healthy
  - Node.js `v22.22.0`
  - npm `10.9.4`
- Cloned TenacitOS into `/home/aiadmin/.openclaw/workspace/mission-control`
- Created local config, installed dependencies, built the app, and brought it up as a local-only service on `127.0.0.1:3000`
- Created and enabled a local systemd service: `mission-control.service`

SOURCE PATCHING / FINDINGS
- Patched the TenacitOS source to remove hardcoded Claude-specific wording in the About page
- Replaced incompatible session-listing logic with direct reads from `/home/aiadmin/.openclaw/agents/*/sessions/*.jsonl`
- Testing showed the dashboard remained too incomplete and mismatched for this OpenClaw environment

FINAL STATUS
- Matt decided not to keep the dashboard
- Rocky removed `/home/aiadmin/.openclaw/workspace/mission-control`
- Matt completed the privileged cleanup needed to remove the systemd service
- Cleanup verification confirmed:
  - `mission-control.service` absent
  - nothing listening on port `3000`
- Net result: no new persistent configuration, platform, service, or capability was retained from this evaluation

====================================================
2026-04-14 (Session 12 — Exchange archive audit, QMD memory backend, Ally routing investigation)
====================================================

MS365 / EXCHANGE ONLINE - APP-ONLY POWERSHELL AUDIT ENABLED
- Rocky now has a working app-only Exchange Online PowerShell audit path for tenant mailbox inspection
- Required Exchange app auth model documented and validated:
  - Entra application permission: `Exchange.ManageAsApp`
  - Exchange RBAC role assignment: `View-Only Recipients`
- New operational capability added: Rocky can query Exchange mailbox recipient/archive state without delegated interactive login
- Audit completed successfully across 53 Exchange mailboxes
- Generated CSV of user mailboxes missing archive enablement and uploaded it to the MattRocky SharePoint site

OPENCLAW MEMORY - QMD BACKEND CONNECTED
- OpenClaw memory backend switched to QMD and verified working end-to-end
- Durable working configuration established in `openclaw.json`:
  - `memory.qmd.includeDefaultMemory = false`
  - explicit collections for `MEMORY.md` and `memory/*.md`
- New capability added: semantic memory search is now backed by QMD with vector search available
- Important design rule locked in: workspace Markdown files remain the canonical memory source; QMD is the rebuildable index/search layer

ALLY / OPENCLAW ISOLATION WORK
- Created separate OAuth profile for Ally's Codex use while preserving Matt's profile
- Created isolated OpenClaw agent `ally` with its own workspace:
  - `/home/aiadmin/.openclaw/workspace/ally`
- Seeded Ally continuity file so the separate agent can maintain its own operating context

DISCORD ROUTING FINDING
- Confirmed a likely OpenClaw routing/binding issue for per-channel non-ACP Discord agent routing in this build
- Tested and corrected top-level route matching for Ally's Discord channel, restarted gateway, and cleared sticky main-session routing state
- Despite the corrected route, fresh channel traffic still instantiated a new `main` session instead of `ally`
- Current conclusion locked in: normal peer-based top-level routing appears insufficient or buggy for this use case; likely next step is source-level patching or a different Discord bot/runtime approach

====================================================
2026-04-21 (Session 13 — LegalServer LIVE CH7 webhook completed)
====================================================

LEGALSERVER LIVE - CH7 WEBHOOK FLOW COMPLETED
- Final deployed webhook endpoint confirmed working:
  - `https://legalserver-ch7-sync.vercel.app/api/ch7-note-hook`
- Matt confirmed the end-to-end CH7 flow now creates the note correctly in LegalServer LIVE
- Durable working Generic Outgoing API block configuration established:
  - Method: `POST`
  - URL: `https://legalserver-ch7-sync.vercel.app/api/ch7-note-hook`
  - Raw header syntax required in HTTP Headers:
    - `x-rocky-secret: rocky-ch7-test-2026-04-21`
  - Parameter to send:
    - `caseId` = `Matter/Case ID#`
  - `caseId` must not be marked as a path parameter
  - `Store response in a note?` must remain unchecked
  - `JSON params to pluck from response for note body` must remain blank
  - `dryRun` may be used for preview testing only and must be removed or false for real note creation

LEGALSERVER LIVE API SYNTAX LEARNED
- Successful `POST /api/v2/notes` create payload shape confirmed for LIVE:
  - `module` must be `matter`
  - `module_id` must be the numeric matter/case id, not the matter UUID
  - `note_type` is required and accepted as:
    - numeric id `100189`
    - object with `lookup_value_id`
    - object with `lookup_value_uuid`
- Rejected note type forms confirmed:
  - `note_type_id`
  - `note_type_uuid`
  - stringified numeric `"100189"`
- Read-only fields that must not be sent on create:
  - `date_posted`
  - `active`
  - `note_was_emailed`
  - `note_was_messaged`
  - `note_has_document_attached`
- Final working payload pattern established:
  - `subject`: `C4L Bankruptcy – Standard Intake Questions`
  - `note_type`: `100189` (`Case Notes`)
  - `is_html`: `true`
  - `allow_etransfer`: `true`

CH7 SOURCE / PARSING FINDINGS
- Correct LegalServer report source URL pattern confirmed:
  - `https://slsct.legalserver.org/modules/report/api_export.php?load=6922&api_key=395ea1f0-e290-449c-9a6c-a4ffd291fa71&filter[matter_identification_number]=<CASE_ID>`
- Important finding: the CH7 report response arrives as XML embedded inside a single `raw` field, not as already-parsed columns
- Rocky updated the app to parse the XML `<row>` content from `raw`, extract the fields, and format the note body correctly

FORMATTING / OUTPUT DECISIONS LOCKED IN
- Note title: `C4L Bankruptcy – Standard Intake Questions`
- Include `Case ID: <case number>` near the top of the note
- Create the note as HTML for reliable rendering in LegalServer
- Use simple HTML paragraph formatting with escaped values for stable output

SAFE OPERATING POSTURE
- The webhook project is functional and deployed
- Keep the allowlist restriction to case `25-0383515` until Matt explicitly approves broader scope
- Preserve webhook secret authentication if the flow is generalized later

====================================================
2026-04-22 (Session 14 — CH7 generalized, C4L webhook deployed, Vercel hosting clarified)
====================================================

LEGALSERVER LIVE - CH7 WEBHOOK GENERALIZED
- Removed the baked-in single-case fallback from the deployed CH7 note webhook
- New durable behavior:
  - `ALLOWED_LIVE_CASE_IDS` is enforced only when explicitly set and non-empty
  - otherwise the webhook allows any case through while preserving shared-secret authentication
- Production behavior verified after removing the lingering Vercel env var allowlist:
  - `https://legalserver-ch7-sync.vercel.app/api/ch7-note-hook`
- Result: the CH7 webhook is no longer restricted to the original test case `25-0383515`

LEGALSERVER LIVE - C4L QUESTIONS WEBHOOK ADDED
- Built and deployed a second LegalServer LIVE webhook app for C4L intake questions
- New Vercel project and production endpoint:
  - project: `legalserver-c4l-sync`
  - endpoint: `https://legalserver-c4l-sync.vercel.app/api/c4l-note-hook`
- Durable production configuration established:
  - shared-secret auth via `x-rocky-secret`
  - source report URL pattern based on LegalServer export `load=7021`
  - LegalServer note type id `100189` (`Case Notes`)
- Working Generic Outgoing API block pattern documented for Matt:
  - Method: `POST`
  - URL: `https://legalserver-c4l-sync.vercel.app/api/c4l-note-hook`
  - Parameter: `caseId = Matter/Case ID#`
  - `caseId` is not a path parameter
  - response-note storage remains disabled

FORMATTER CAPABILITY EXPANDED
- Updated and redeployed both LegalServer webhook apps so blank/self-closing XML answers are preserved and rendered explicitly as:
  - `Answer: (no response)`
- This behavior now applies to both deployed endpoints:
  - `https://legalserver-ch7-sync.vercel.app/api/ch7-note-hook`
  - `https://legalserver-c4l-sync.vercel.app/api/c4l-note-hook`
- Result: Rocky now preserves question visibility even when the upstream LegalServer report returns blank values

VERCEL HOSTING IDENTITY CLARIFIED
- Confirmed the Vercel account/scope currently hosting Rocky's deployed apps:
  - username: `slsrocky`
  - team/scope: `slsrockys-projects`
  - account email: `rocky@slsct.org`
- Confirmed active hosted projects in that scope include:
  - `legalserver-ch7-sync`
  - `legalserver-c4l-sync`
  - `legalserver-pdc-reopen`
  - `zoom-dashboard`

====================================================
2026-04-23 (Session 15 — OCR capability added, local PaddleOCR stack stabilized)
====================================================

OCR CAPABILITY ADDED
- Rocky evaluated OCR options for local toolbox use and selected PaddleOCR as the primary recommendation
- Recommendation rationale locked in:
  - strongest practical all-around default for local OCR
  - self-hostable
  - better layout/table handling than Tesseract
  - Apache 2.0 licensing avoids Surya GPL concerns
- Secondary/fallback tools noted:
  - `Tesseract`
  - `OCRmyPDF` for PDF-oriented workflows

LOCAL OCR RUNTIME CONFIGURATION
- Durable isolated OCR environment created at:
  - `/home/aiadmin/.openclaw/workspace/.venv-paddleocr`
- Initial latest-package install path was tested and rejected due to runtime prediction failure on this host:
  - `paddleocr 3.5.0`
  - `paddlepaddle 3.3.1`
- Stable working compatibility stack established instead:
  - `paddleocr 2.7.3`
  - `paddlepaddle 2.6.2`
  - `numpy 1.26.4`

REUSABLE OCR WRAPPER ADDED
- New wrapper script created:
  - `/home/aiadmin/.openclaw/workspace/scripts/paddleocr_run.py`
- Wrapper accepts an image path and returns structured JSON including:
  - combined extracted text
  - per-line text
  - confidence values
  - bounding boxes
- Durable usage pattern:
  - `/home/aiadmin/.openclaw/workspace/.venv-paddleocr/bin/python /home/aiadmin/.openclaw/workspace/scripts/paddleocr_run.py <image_path>`

VALIDATION
- Smoke test succeeded on a generated sample image
- Verified OCR output correctly returned: `Hello OCR 123`

====================================================
2026-04-24 (Session 16 — SharePoint OCR workflow connected end-to-end)
====================================================

SHAREPOINT OCR WORKFLOW ADDED
- Rocky completed a full OCR job directly from the private MattRocky SharePoint site and wrote the extracted text back to the same SharePoint folder
- Source file processed:
  - `Documents/OCR/pb case status.png`
- Output file created:
  - `Documents/OCR/pb case status.txt`
- Result: Rocky can now take an image dropped into the SharePoint OCR folder, extract text locally, and return a `.txt` file to SharePoint

SHAREPOINT CONNECTION CONFIRMED
- Rocky accessed the MattRocky SharePoint site via Microsoft Graph using the durable local MS365 configuration already established in this runtime
- Confirmed working against the SharePoint OCR folder without needing an interactive re-auth step during the task

OCR PIPELINE VALIDATED IN PRODUCTION WORKFLOW
- Rocky used the existing local PaddleOCR wrapper to process the SharePoint image and upload the extracted text output back to SharePoint
- This validates the practical end-to-end path:
  - SharePoint file retrieval
  - local OCR processing
  - SharePoint text-file upload
- Result: Rocky now has a working operational OCR workflow for files placed in the private SharePoint OCR folder

====================================================
2026-04-25 (Session 17 — OpenClaw 2026.4.24, GPT 5.5 default, Session Radar MVP)
====================================================

OPENCLAW UPDATE COMPLETED
- Updated OpenClaw from `2026.4.9` to `2026.4.24`
- Confirmed gateway health after the update and restart
- Verified the installed gateway and npm latest version both reported `2026.4.24`

GPT 5.5 ENABLED AS DEFAULT
- Confirmed GPT 5.5 availability in both model catalogs:
  - `openai-codex/gpt-5.5`
  - `openai/gpt-5.5`
- Set the default model to `openai-codex/gpt-5.5` to match Matt's existing working OpenAI Codex OAuth path
- Verified new sessions should use GPT 5.5 by default
- Operational note: existing sessions may continue showing/using the prior model until reset or fresh session start

DISCORD ELEVATED EXEC ACCESS ENABLED FOR MATT
- Enabled elevated host command access from Discord for Matt's Discord user only
- Verified elevated exec works from the Discord session by running OpenClaw status checks
- Security caveat documented: Discord group policy still reports warnings while groupPolicy remains open; future hardening should allowlist Discord group access while keeping elevated access tightly scoped

AUTH / MEMORY / SESSION PRACTICES CONFIRMED
- Confirmed the active Discord channel/session was bound to Matt's OpenAI Codex OAuth profile rather than Ally's
- Confirmed QMD remains the active memory backend and vector search layer for workspace memory files
- Reviewed and locked in lightweight operating practices to keep long-running sessions healthy:
  - sub-agents for larger tasks
  - TaskFlow/cron for durable background work
  - QMD search for targeted memory retrieval
  - file checkpoints before resets
  - fresh sessions after major work blocks
  - concise summaries instead of raw logs

ROCKY SESSION RADAR MVP BUILT
- Built a read-only local browser dashboard for monitoring OpenClaw session size and reset risk
- Project location:
  - `/home/aiadmin/.openclaw/workspace/session-radar/`
- Default run command:
  - `cd /home/aiadmin/.openclaw/workspace/session-radar && npm start`
- Default local URL:
  - `http://127.0.0.1:8787/`
- MVP features:
  - lists OpenClaw sessions from local session stores
  - shows agent, session type, model/auth, token usage, percent full, age, staleness, and reset recommendation
  - refreshes automatically every 20 seconds
  - includes filters for agent, risk, age/staleness, and text search
  - includes sorting by risk, age, size, and agent/name
  - shows checkpoint/reset buttons as disabled placeholders for future safer write-enabled phases
- Performance improvement made during build:
  - replaced slow `openclaw sessions --all-agents --json` API approach with direct local session-store reads
  - reduced session API response time from roughly 45 seconds to near-instant local reads
- UI layout fix completed after screenshot feedback:
  - widened the table
  - added horizontal scrolling
  - removed sticky header clipping
  - allowed session names/keys to wrap cleanly
- Matt confirmed the dashboard looked perfect after the layout fixes

NEXT LIKELY SESSION RADAR WORK
- Make the dashboard durable as a service so it survives command-session exits
- Add Discord threshold alerts with dedupe/cooldowns
- Later add protected checkpoint/reset actions after confirming a safe OpenClaw reset mechanism

====================================================
2026-04-26 (Session 18 — GPT 5.5 runtime confirmation, QMD checks, Session Radar stabilization)
====================================================

GPT 5.5 RUNTIME CONFIRMED AFTER RESET
- Confirmed the active Rocky runtime/session was using `openai-codex/gpt-5.5`
- Confirmed the Discord channel/session was bound to Matt's OpenAI Codex OAuth profile (`mdugan@slsct.org`) rather than Ally's profile
- Confirmed no additional reset was needed because the live session was already using Matt's profile and GPT 5.5

QMD MEMORY BACKEND VERIFIED
- Confirmed QMD remains the active memory/search backend for Rocky's workspace memory
- Verified the main agent QMD index was present, vector search was enabled, and the index was not dirty
- Confirmed the operational practice of using QMD targeted retrieval, file checkpoints, sub-agents, TaskFlow/cron, and fresh sessions to keep main sessions lighter

SESSION RADAR READ-ONLY MVP STABILIZED
- Continued and validated the Rocky Session Radar read-only local dashboard at:
  - `/home/aiadmin/.openclaw/workspace/session-radar/`
  - `http://127.0.0.1:8787/`
- The dashboard shows OpenClaw session rows with agent, type, model/auth, token usage, percent full, age, staleness, risk, and reset recommendation
- Confirmed auto-refresh, filtering, sorting, and disabled placeholder checkpoint/reset buttons for future safer write-enabled phases
- Reworked the session API to read local session stores directly instead of using the much slower CLI session listing path
- Observed the direct local read approach returning session data near-instantly instead of taking roughly 45 seconds
- Applied UI layout fixes after Matt's screenshot feedback:
  - wider table and horizontal scrolling
  - removed sticky header clipping
  - fixed column sizing
  - session names/keys wrap cleanly instead of being cut off
- Matt confirmed the dashboard looked perfect after these fixes

NEXT SESSION RADAR WORK
- Make Session Radar durable as a service so it survives command-session exits
- Add Discord threshold alerts with dedupe/cooldowns
- Later add protected checkpoint/reset actions after confirming a safe OpenClaw reset mechanism


====================================================
2026-04-27 (Session 19 — RTC automation repairs and Discord OAuth persistence)
====================================================

RTC MONTHLY REPORT CRON FIXED
- Investigated Matt's concern that the full-month RTC monthly report was running weekly instead of only monthly
- Found the monthly cron job was misconfigured as `0 7 1-7 * 1`, causing repeated runs in the first week when Mondays matched
- Corrected `rtc-monthly-reports` to run only on the first day of each month at 7:00 AM ET:
  - `0 7 1 * *`
- Updated the job description to document that it covers the previous full month
- Verified the job remained enabled and next scheduled for 2026-05-01 at 7:00 AM ET

RTC WEEKLY REPORT PIPELINE REPAIRED
- Matt reported the weekly RTC draft for 2026-04-20 through 2026-04-26 was missing the LegalServer referrals/demographics PDF
- Root cause: the maintained weekly script only generated and attached the three Zoom PDFs, while the older unified pipeline still had the LegalServer report logic
- Created a corrected Outlook draft for Matt with all four weekly report attachments:
  - `RTC Hotline - Call Flow Report - 0420 - 0426.pdf`
  - `RTC Hotline - Eligible Zip Codes - 0420 - 0426.pdf`
  - `RTC Hotline - Non-Eligible Zip Codes by Town - 0420 - 0426.pdf`
  - `SLS RTC Referrals & Demographics Report - 0420 - 0426.pdf`
- Uploaded all four PDFs to SharePoint under `RTC Hotline Reports/Weekly/`
- Patched `rtc_automation/run_weekly_rtc_reports.py` so future weekly drafts include the LegalServer referrals/demographics PDF automatically
- Updated the weekly RTC cron prompt/description to use the maintained weekly script and verify four attachments
- Verified the corrected latest Outlook draft had exactly four attachments and the Python compile check passed

RTC REPORT SCHEDULES STANDARDIZED TO 7 AM ET
- Matt asked for RTC report crons to run at 7 AM instead of 11 AM
- Updated `rtc-weekly-reports` from `0 11 * * 1` to `0 7 * * 1` in `America/New_York`
- Confirmed `rtc-monthly-reports` was already `0 7 1 * *` after the monthly cron repair
- Verified both RTC report jobs are enabled and scheduled for 7:00 AM ET

DISCORD CODEX OAUTH PERSISTENCE FIXED
- Investigated why the `#matt-chatwithrocky` Discord session kept reverting to Ally's OpenAI Codex OAuth profile after reset/new-session flows
- Found the persistent session registry entry for Matt's Discord channel had an automatic auth override pointing to Ally's profile
- Backed up the session registry and changed only Matt's Discord channel session entry to use Matt's OpenAI Codex OAuth profile as a manual/user override
- Verified the active session reported `openai-codex/gpt-5.5` with Matt's `mdugan@slsct.org` Codex OAuth profile
- Left Ally's separate session/channel configuration unchanged

====================================================
2026-04-28 (Session 20 — LegalServer PB Case Status Report OCR automation)
====================================================

LEGALSERVER PB CASE STATUS REPORT OCR WORKFLOW BUILT
- Built a production-style local poller for LegalServer PB Case Status Report processing:
  - `/home/aiadmin/.openclaw/workspace/ocr_project/pb_status_poller.py`
- Default mode is dry-run/no writes; live writes require explicit `--execute`
- Poller reads the LegalServer report API for same-day `slsct_pro_bono_case.pdf` documents, downloads each PDF, extracts text, validates the `PRO BONO CASE STATUS REPORT` header, verifies case-number match, parses key fields, and prepares/executes final actions

LEGALSERVER REPORT + DOCUMENT CONNECTIONS CONFIRMED
- Confirmed LegalServer report API access works with Rocky's live bearer token
- Confirmed queued report rows expose document ID, matter/case ID, case number, creation date, and document profile link
- Confirmed PDF download works through:
  - `/modules/document/download.php?id={document_id}`
- Uploaded an XSLT normalization helper to SharePoint OCR folder, but confirmed the LegalServer report API continued returning the generic XML shape; poller now robustly parses that generic XML directly

LIVE WRITE CAPABILITIES CONFIRMED ON APPROVED TEST CASES
- Confirmed document completion marker:
  - `PATCH /api/v2/documents/{document_uuid}` with `type.lookup_value_id = 3322657`
  - Sets document type to `PB Case Status Report` so successful rows drop from the queue
- Confirmed case closing date update after Matt added the needed permission:
  - `PATCH /api/v2/matters/{matter_uuid}` with `date_closed`
- Confirmed PB summary note creation:
  - `POST /api/v2/notes`
  - Subject/body begin with `PB Attorney Brief Summary`
- Confirmed pro bono timeslip creation after Matt added the needed permission:
  - `POST /api/v2/timeslips`
  - Uses `03 Pro Bono`, activity type `Case Activity`, PB activity codes, Ben Franklin caseworker lookup, and no Activity Details
- Confirmed case alert creation through the LegalServer UI form endpoint when Core API support was not found:
  - `/matter/alert/edit/{matter_id}`

PB TIME RULES LOCKED IN
- If total time is 7 hours or less, create one timeslip on the closing date
- If total time is 8 hours or more, split into chunks of 7 hours or less going backward consecutively from the closing date
- Do not create timeslips before the case open date
- If there are not enough eligible days, skip timeslips and create alert:
  - `PB TIME NOT ENTERED - NOT ENOUGH DAYS`
- If calculated service dates are older than LegalServer's 60-day limit, skip timeslips and create alert:
  - `PB TIMESLIPS NOT ADDED - TOO FAR BACK IN TIME`

ACTIVITY CODE MAPPING ADDED
- Counsel and Advice: `3319753`
- Limited Action: `3319754`
- Negotiated Settlement without litigation: `3319755`
- Negotiated Settlement with litigation: `3319756`
- Administrative Agency Decision: `3319757`
- Court Decision: `3319758`
- Other / Explain: `3319759`
- Extensive Service: `3319760`

END-TO-END VALIDATION COMPLETED
- Happy-path live test succeeded on Matt-created test case `25-0388775`:
  - close date updated
  - PB summary note created
  - three pro bono timeslips created
  - document type changed last
  - queue dropped to zero
- Too-far-back alert-path live test succeeded on test case `25-0384784`:
  - close date and PB note handled
  - old timeslips skipped
  - alert `PB TIMESLIPS NOT ADDED - TOO FAR BACK IN TIME` created
  - document type changed last
  - queue dropped to zero

SCHEDULING PLAN DOCUMENTED
- Added scheduling notes in:
  - `/home/aiadmin/.openclaw/workspace/ocr_project/SCHEDULING.md`
- Recommended free/cheap approach: run locally on the Rocky/OpenClaw host every 5 minutes
- Next decision: whether to enable recurring dry-run monitoring first or approved recurring `--execute` automation

REMAINING IMPROVEMENTS
- Add true OCR fallback for scanned PDFs
- Refine duplicate/idempotency matching if production data may already contain legitimate PB notes or timeslips
- Decide whether and when to schedule the poller automatically
