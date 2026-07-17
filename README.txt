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

====================================================
2026-04-29 (Session 21 — LegalServer note apps, PDC live hook, OCR/PB automation enabled)
====================================================

LEGALSERVER C4L NOTE APP HARDENED AND REPOINTED
- Troubleshot C4L notes that were creating title-only notes when the source report returned no usable fields
- Patched the C4L webhook to fail safely with HTTP 422 `source_report_empty` instead of creating empty/title-only notes
- Repointed the C4L production app to Matt's rebuilt LegalServer report:
  - report load `7120`
  - filter uses `matter_identification_number`
  - production endpoint remains `https://legalserver-c4l-sync.vercel.app/api/c4l-note-hook`
- Cleaned rebuilt-report output formatting:
  - omitted internal fields
  - stripped trailing numeric field IDs from labels
  - rendered boolean values as `Yes` / `No`
  - preserved blank answers as `(no response)`
- Verified dry-run output for the rebuilt report sample returned a clean generated note body with 26 question/answer lines

LEGALSERVER BANKRUPTCY / CH7 NOTE APP HARDENED AND REPOINTED
- Repointed the Bankruptcy/CH7 production app to Matt's rebuilt LegalServer report:
  - report load `7087`
  - filter uses `matter_identification_number`
  - production endpoint remains `https://legalserver-ch7-sync.vercel.app/api/ch7-note-hook`
- Patched the CH7 app to fail safely with HTTP 422 `source_report_empty` instead of creating title-only notes when the source report has no usable fields
- Cleaned rebuilt-report output formatting:
  - omitted internal database/matter ID fields
  - preserved blank/self-closing XML fields as `(no response)`
  - rendered boolean values as `Yes` / `No`
- Verified dry-run output for the rebuilt report sample generated a clean note body with the expected fields

PDC REOPEN WEBHOOK PREPARED FOR LEGALSERVER LIVE
- Updated project:
  - `/home/aiadmin/.openclaw/workspace/legalserver-pdc-reopen/`
- Removed the prior demo/test-case allowlist enforcement so the webhook can be used from LIVE LegalServer
- Updated environment resolution to prefer LIVE LegalServer configuration while retaining generic/legacy fallbacks
- Added LIVE LegalServer Vercel production environment configuration for the app
- Deployed production endpoint:
  - `https://legalserver-pdc-reopen.vercel.app/api/reopen-hook`
- Simplified the appended note line so it now writes exactly:
  - `Program Disposition Code: <code>`
- Verified the allowlist gate was removed without performing a LegalServer write during smoke testing

OCR / PB CASE STATUS REPORT WORKFLOW FINALIZED AND AUTOMATED
- Updated the OCR/PB poller write order so Case Status is now the final completion step:
  1. update closing date
  2. create or skip PB summary note
  3. set document type to `PB Case Status Report`
  4. create or skip timeslips / alerts
  5. set Case Status to `PB Case Status Returned`
- Confirmed LegalServer Case Status lookup:
  - `PB Case Status Returned` = `3322690`
- Strengthened safety and idempotency behavior:
  - allocation failures other than `not_enough_days` now stop before any writes
  - exact existing PB timeslips are skipped on retry
  - non-matching PB-looking timeslips still block to prevent duplicates
- Processed live queued case `24-0373550` successfully through the OCR/PB workflow:
  - close date set
  - PB summary note created
  - document type changed
  - four PB timeslips totaling 27.0 hours created
  - Case Status set to `PB Case Status Returned`
  - queue verified empty afterward
- Enabled free local automation with system cron on the Rocky/OpenClaw host:
  - cron runs wrapper every 2 minutes
  - wrapper enforces Eastern Time business hours, Monday-Friday 9:00 AM through 4:59 PM
  - inside business hours it runs `ocr_project/pb_status_poller.py --execute`
  - logs to `ocr_project/out/poller_runs/cron.log`
- Verified cron service is enabled and active
- Manual wrapper test processed case `24-0373581` successfully with status `executed`

====================================================
2026-05-01 (Session 22 — RTC monthly report run and monday daily summary hardening)
====================================================

RTC MONTHLY REPORTS — APRIL 2026 RUN COMPLETED
- The scheduled `rtc-monthly-reports` automation ran for the previous full calendar month: April 1, 2026 through April 30, 2026
- Generated the monthly RTC Hotline report set and uploaded the PDFs to SharePoint under the Monthly reports folder:
  - Call Flow Report — `0401 - 0430`
  - Eligible Zip Codes — `0401 - 0430`
  - Non-Eligible Zip Codes by Town — `0401 - 0430`
- Created a draft email in Matt's mailbox with subject:
  - `RTC Hotline Data Reports - April 1, 2026 - April 30, 2026`
- Confirmed the draft was created only, not sent, and that the expected attachments were present
- Verified the Zoom Contact Center fetch handled high-volume days correctly by splitting capped 10,000-record days into smaller UTC windows
- April run summary:
  - 156,377 Zoom Contact Center records fetched
  - 1,770 external inbound calls
  - Inbound: 1,605 ENG / 165 ESP
  - Eligible: 927 ENG / 92 ESP
  - Non-Area: 196 ENG / 11 ESP

MONDAY DAILY CALL SUMMARY AUTOMATION HARDENED
- Continued hardening the monday.com daily call summary workflow used for call-center reporting boards
- Confirmed canonical board structure for both Queue Stats 2026 and Agent Stats 2026:
  - one group per queue/operator
  - one parent item per month
  - one day subitem per day
- Updated the daily sync/repair workflow so repair runs can automatically prune known legacy monday.com structures:
  - old `Week of ...` queue parent items
  - old flat dated agent items such as `YYYY-MM-DD - Staff Name`
- Cleaned leftover legacy items from Queue Stats 2026 and Agent Stats 2026 after approval from Ally in the dedicated Ally channel
- Documented the canonical monday daily summary rules and repair workflow in:
  - `/home/aiadmin/.openclaw/workspace/ally/MONDAY_DAILY_CALL_SUMMARY_SPEC.md`


====================================================
2026-05-04 (Session 23 — Weekly RTC reports and platform usage report)
====================================================

RTC WEEKLY REPORT AUTOMATION RUN COMPLETED
- The scheduled `rtc-weekly-reports` automation ran for the previous Monday-Sunday period: April 27, 2026 through May 3, 2026
- Generated the weekly RTC Hotline report set and uploaded all four PDFs to SharePoint under `RTC Hotline Reports/Weekly/`:
  - `RTC Hotline - Call Flow Report - 0427 - 0503.pdf`
  - `RTC Hotline - Eligible Zip Codes - 0427 - 0503.pdf`
  - `RTC Hotline - Non-Eligible Zip Codes by Town - 0427 - 0503.pdf`
  - `SLS RTC Referrals & Demographics Report - 0427 - 0503.pdf`
- Created a draft email in Matt's mailbox with subject:
  - `RTC Hotline Data Reports - April 27, 2026 - May 3, 2026`
- Confirmed the draft was created only, not sent, and that exactly four attachments were present
- Verified Zoom Contact Center cap handling during the run; 2026-04-27 hit the 10,000-record cap and was split into 4-hour windows
- Weekly run summary:
  - 41,457 Zoom Contact Center records fetched
  - 614 external inbound calls
  - 313 eligible calls
  - 51 non-eligible calls

WEEKLY PLATFORM USAGE REPORT RUN COMPLETED
- The scheduled weekly usage report automation generated and committed:
  - `reports/weekly-2026-04-27.html`
- Pulled live usage data where available and emailed the report from `Rocky@slsct.org` to `mdugan@slsct.org`
- Committed report to `SLSRocky/rocky-readme`:
  - commit `c4de334`
- Reported platform status for the week:
  - Twilio: 158 SMS messages, $1.42 SMS cost, $4.37 total weekly spend
  - Vercel: account/project API access live; detailed usage endpoint blocked by plan/API limit (`plan_upgrade_required`)
  - Brave Search: usage totals unavailable from current tooling/API access
  - Anthropic/Claude: reporting endpoints unavailable with current credentials; admin key needed



====================================================
2026-05-05 (Session 24 — RTC report fixes, April funnel repair, and usage cron cleanup)
====================================================

RTC REPORT AUTOMATION HARDENED
- Fixed weekly and monthly RTC ZIP/town mapping so report town distributions use real caller ZIP fields only:
  - `zipcode.Digits`
  - `ESP zipcode.Digits`
  - `Zip Code`
- Excluded routing ZIP variables such as `CLS-ZipCode`, `GHLA-ZipCode`, and `NHLAA-ZipCode` from town mapping after they caused an Eligible ZIP report to show `Non-CT 100%`
- Regenerated and re-uploaded corrected Apr 27-May 3 weekly RTC PDFs to SharePoint and created a new corrected Outlook draft for Matt
- Added the missing `SLS RTC Referrals & Demographics Report` attachment to monthly RTC report generation
- Updated the monthly RTC cron/job flow so future monthly drafts expect and attach all four report PDFs
- Fixed RTC demographics crosstabs so preferred rows are kept but additional observed categories are appended instead of dropped; this preserved a `Trans woman` Gender row and corrected April demographic totals
- Regenerated/re-uploaded April monthly RTC PDFs and created fresh four-attachment Outlook drafts after each correction
- Relevant code commits in the workspace/reporting repo:
  - `916fea8 Fix RTC eligible ZIP town mapping`
  - `8cba227 Add monthly RTC referrals report attachment`
  - `5d8aff9 Include observed demographic categories in RTC reports`

APRIL FUNNEL REPORT AUTOMATION REPAIRED
- Ran the April 2026 Monthly Funnel Report workflow and created the `Updated Monthly Funnel Report` Outlook draft for Matt
- Fixed the monthly funnel job so chart PDF rendering is month-aware instead of hard-coded for prior months
- Fixed April funnel chart line breaks by backfilling blank current-year history columns from existing monthly output workbooks before plotting
- Regenerated, visually verified, uploaded, and replaced corrected April SLS and RTC funnel chart PDFs on the existing Outlook draft
- Relevant code commits in the workspace/reporting repo:
  - `9eb337b Fix monthly funnel report drafting`
  - `b784dfd Fix funnel chart history line breaks`

ZOOM CONTACT CENTER AD HOC REPORTING CAPABILITY USED
- Produced aggregate-only Zoom Contact Center pie chart reports for SLS Main Hotline Main Menu choices from Jan. 1, 2025 through May 5, 2026
- Created an Outlook draft for Matt with two PDF attachments:
  - CT Benefits vs Gov Benefits choices
  - Full first-level Main Menu choices
- Refined the all-choices pie chart to use a non-overlapping legend layout and replaced the attachment on the existing draft

CRON CONFIGURATION CLEANUP
- Removed the `Weekly Usage Report` Gateway cron job after Matt said the Rocky usage weekly reports were no longer useful
- Deleted cron job id `2786974f-3e94-4273-9a28-f61a582b0c10`



====================================================
2026-05-06 (Session 25 — AssetPanda Dell serial cleanup)
====================================================

ASSETPANDA DELL ASSET DATA CLEANUP COMPLETED
- Built and used a targeted AssetPanda audit/update helper for Dell hardware records, including assets identified by product/manufacturer text such as `Dell`, `Latitude`, and `Optiplex`
- Found 143 matching Dell-related assets for review
- Updated 132 assets so Dell serial values were copied into the dedicated Dell fields:
  - Dell Service Tag
  - Dell Express Service Code
- Preserved AssetPanda uniqueness constraints and skipped blocked duplicate Service Tag values rather than forcing unsafe writes
- Known skipped/blocked monitor records where AssetPanda rejected non-unique Service Tags:
  - asset IDs 212, 233, 236, 238, and 252
- Matt approved ignoring the remaining no-serial/manual-fix edge cases after the bulk cleanup
- Local audit/helper artifacts were kept under:
  - `/home/aiadmin/.openclaw/workspace/tmp/assetpanda_dell_tags/`
  - `/home/aiadmin/.openclaw/workspace/tmp/assetpanda_dell_service_tags.py`

CAPABILITY ADDED / CONFIRMED
- Rocky can now audit AssetPanda asset records for Dell-identifying product data and safely populate Dell-specific serial fields in bulk, with uniqueness/error handling and audit output for follow-up review

====================================================
2026-05-08 (Session 26 — Conflict Check Email design and MS365 shared mailbox scope planning)
====================================================

CONFLICT CHECK EMAIL WORKFLOW REVIEWED
- Reviewed the SharePoint `Conflict Check Email` folder in the MattRocky site, including:
  - `Project Conflict Check Email.docx`
  - `Email Body.docx`
- Confirmed the requested workflow design:
  - LegalServer staff action/API block calls a Rocky-hosted webhook
  - webhook fetches current case details from LegalServer
  - generated email includes client full name, DOB, and adverse party names
  - test mode forces recipient to `mdtech01@gmail.com` before any external-agency rollout
  - CC should include the primary assignment email and case email address
  - subject should be `Emergency conflict check, please`
- Recommended implementation pattern: small Vercel webhook, shared-secret validation, LegalServer read, MS365 Graph send, minimal logging with no client/adverse-party details in logs

MS365 / GRAPH MAILBOX SCOPE CHECK
- Performed a harmless Microsoft Graph send test using the proposed sender `slstransfers@slsct.org`; no message was sent
- Graph returned `404 ErrorInvalidUser`, showing Rocky's app could not currently resolve/send as that mailbox through its existing access scope
- Documented the next Exchange Online PowerShell steps for Matt:
  - add `slstransfers@slsct.org` to `rocky-ai-access@slsct.org`
  - verify group membership
  - test the existing Rocky app access policy against app id `845c0e8f-5f38-455f-af05-6dcab3cf669e`
- Status: planning/validation only; no production webhook or external-email sending capability was deployed yet


====================================================
2026-05-11 (Session 27 — RTC weekly report run)
====================================================

RTC WEEKLY REPORT AUTOMATION RUN COMPLETED
- The scheduled `rtc-weekly-reports` automation ran for the previous Monday-Sunday period: May 4, 2026 through May 10, 2026
- Generated the weekly RTC Hotline report set and uploaded all four PDFs to SharePoint under `RTC Hotline Reports/Weekly/`:
  - `RTC Hotline - Call Flow Report - 0504 - 0510.pdf`
  - `RTC Hotline - Eligible Zip Codes - 0504 - 0510.pdf`
  - `RTC Hotline - Non-Eligible Zip Codes by Town - 0504 - 0510.pdf`
  - `SLS RTC Referrals & Demographics Report - 0504 - 0510.pdf`
- Created a draft email in Matt's mailbox with subject:
  - `RTC Hotline Data Reports - May 4, 2026 - May 10, 2026`
- Confirmed the draft was created only, not sent, and that exactly four attachments were present
- Verified Zoom Contact Center cap handling during the run; no days required cap-splitting
- Weekly run summary:
  - 37,725 Zoom Contact Center records fetched
  - 522 external inbound calls
  - 239 eligible calls
  - 54 non-eligible calls
- Confirmed the temporary `/tmp/rtc` workspace was removed by the pipeline cleanup step

CAPABILITY STATUS
- No new platform connection or configuration was added today; this was a successful routine run of the existing RTC weekly reporting capability

====================================================
2026-05-12 (Session 28 — slstransfers shared mailbox email capability verified)
====================================================

MS365 / GRAPH SHARED MAILBOX SEND CAPABILITY VERIFIED
- Checked Rocky's Microsoft Graph access for `slstransfers@slsct.org` after the shared mailbox scope work from the conflict-check email planning
- Confirmed Graph can access the `slstransfers@slsct.org` mailbox folders/messages even though direct user-object lookup returned `404`, consistent with shared/hidden mailbox behavior
- Verified Rocky can send email from `slstransfers@slsct.org` via Microsoft Graph with approved test sends:
  - internal test to `mdugan@slsct.org`: Graph returned `202 Accepted`
  - one-time approved external test to `mdtech01@gmail.com`: Graph returned `202 Accepted`
- This confirms the `slstransfers@slsct.org` sender is available for the planned conflict-check email workflow once the production workflow is built and separately approved

BOUNDARY / SAFETY NOTE
- The external Gmail send was a one-time approved test only
- Standing email boundary remains: Rocky sends only to `mdugan@slsct.org` by default unless Matt explicitly approves a specific different recipient/action

====================================================
2026-05-13 (Session 29 — LegalServer Conflict Check Email webhook built and configured)
====================================================

CONFLICT CHECK EMAIL WEBHOOK BUILT AND DEPLOYED
- Created the Vercel project `legalserver-conflict-check-email` for the LegalServer emergency conflict-check email workflow
- Production endpoint configured at:
  - `https://legalserver-conflict-check-email.vercel.app/api/conflict-check-email`
- Added shared-secret validation with the required `x-rocky-secret` request header
- Implemented request validation for LegalServer-provided fields including client name, DOB, adverse party data, primary assignment email, case email, and selected conflict-check destination
- Added safe adverse-party parsing for LegalServer formatted text:
  - strips `AP:` labels
  - ignores metadata lines such as `State: CT`
  - joins multiple adverse parties with `; `

MS365 EMAIL GENERATION / FORMATTING ADDED
- Connected the webhook to Microsoft Graph sendMail using the approved `slstransfers@slsct.org` sender configuration
- Implemented HTML email output with:
  - subject `Emergency conflict check, please`
  - bold `Client:`, `DOB:`, and `Adverse Party:` labels
  - a blank line between DOB and Adverse Party
- Test-send mode was used first with `mdtech01@gmail.com`, then removed before live-routing configuration

LIVE ROUTING CONFIGURED, THEN SAFELY PAUSED
- Added live recipient routing support:
  - always TO `slstransfers@slsct.org`
  - CLS routes to `clsalltransfersfromsls@ctlegal.org`
  - NHLAA routes to `NHLAAReferrals@nhlegal.org`
  - CC includes primary assignment email and case email
- Added fail-safe handling for unknown/unmapped `conflict_check_email` values so the webhook reports a missing mapped recipient rather than guessing or falling back to a test address
- Enabled live routing after Matt approved it, then later paused actual sending while Matt moves the LegalServer API call to a second form so the selected conflict-check destination is saved before the API block runs
- Current production state at end of day:
  - `CONFLICT_CHECK_SEND_ENABLED=0` — sending paused
  - `USE_LIVE_CONFLICT_CHECK_ROUTING=1` — live routing remains active for dry-run previews
  - `TEST_CONFLICT_CHECK_TO` removed

VERIFICATION / AUDITABILITY
- Verified unauthenticated requests return `401`
- Verified authenticated dry-run requests return `mode: dry_run_no_email_sent` while sending is paused
- Added route logging for non-client-body routing metadata only, including routing mode, send-enabled status, selected destination, resolved recipients, CC count, and missing-field status
- Local project folder:
  - `/home/aiadmin/.openclaw/workspace/legalserver-conflict-check-email/`

CAPABILITY ADDED
- Rocky can now host a LegalServer-triggered conflict-check email webhook on Vercel, generate formatted MS365 email content, route conflict-check requests to the correct transfer mailbox/external agency recipients, and safely pause or dry-run sends while preserving live routing previews


====================================================
2026-05-14 (Session 30 — Conflict Check Email live enablement and parser hardening)
====================================================

CONFLICT CHECK EMAIL LIVE SENDING ENABLED
- Matt moved the LegalServer API call to a second form/action so the saved `conflict_check_email` value is available before the webhook runs
- Re-enabled production sending for the `legalserver-conflict-check-email` Vercel project:
  - `CONFLICT_CHECK_SEND_ENABLED=1`
  - `USE_LIVE_CONFLICT_CHECK_ROUTING=1`
- Production endpoint remains:
  - `https://legalserver-conflict-check-email.vercel.app/api/conflict-check-email`
- Confirmed live recipient routing remains:
  - always TO `slstransfers@slsct.org`
  - CLS also TO `clsalltransfersfromsls@ctlegal.org`
  - NHLAA also TO `NHLAAReferrals@nhlegal.org`
  - CC includes primary assignment email and case email

LEGALSERVER CLIENT-NAME HTML CLEANUP ADDED
- LegalServer sent `client_full_name` as an HTML anchor in at least one real test payload
- Patched the email builder to strip HTML tags/entities from client full name while preserving the visible name only
- Confirmed the email preview no longer includes anchor tags or `href` content in the client-name field

VERIFICATION / FINAL STATE
- Authenticated verification probes confirmed live routing previews with `sendEnabled:true`
- Smoke tests passed locally after the client-name HTML cleanup
- Redeployed the Vercel production alias after the parser fix
- Matt completed a real LegalServer second-form test and confirmed the workflow was working: "we are golden"

CURRENT CAPABILITY STATUS
- The LegalServer Conflict Check Email workflow is now production-ready and live:
  - LegalServer posts JSON to Rocky's Vercel webhook with `x-rocky-secret`
  - webhook validates required fields and destination routing
  - email is sent from `slstransfers@slsct.org` through Microsoft Graph
  - external routing supports CLS and NHLAA destinations with safe failure for unknown values
  - LegalServer adverse-party formatted text and client-name HTML are normalized before email generation

SECURITY / FOLLOW-UP NOTE
- Rotate the webhook shared secret when convenient because the `x-rocky-secret` appeared in a Discord screenshot during setup/testing

====================================================
2026-05-18 (Session 31 — RTC weekly report run)
====================================================

RTC WEEKLY REPORT AUTOMATION RUN COMPLETED
- The scheduled RTC weekly reporting automation ran for the previous Monday-Sunday period: May 11, 2026 through May 17, 2026
- Generated the weekly RTC Hotline report set and confirmed all four report PDFs were attached to the draft email:
  - `RTC Hotline - Call Flow Report - 0511 - 0517.pdf`
  - `RTC Hotline - Eligible Zip Codes - 0511 - 0517.pdf`
  - `RTC Hotline - Non-Eligible Zip Codes by Town - 0511 - 0517.pdf`
  - `SLS RTC Referrals & Demographics Report - 0511 - 0517.pdf`
- Created a draft email in Matt's mailbox with subject:
  - `RTC Hotline Data Reports - May 11, 2026 - May 17, 2026`
- Confirmed the draft was created only, not sent, and that exactly four attachments were present
- Verified Zoom Contact Center cap handling during the run; no days required cap-splitting
- Weekly run summary:
  - 38,123 Zoom Contact Center records fetched
  - 460 external inbound calls
  - 225 eligible calls
  - 34 non-eligible calls
- Confirmed the temporary `/tmp/rtc` workspace was removed by the pipeline cleanup step

CAPABILITY STATUS
- No new platform connection or configuration was added today; this was a successful routine run of the existing RTC weekly reporting capability

====================================================
2026-05-19 (Session 32 — AssetPanda inventory form OCR and attachment workflow)
====================================================

ASSETPANDA ATTACHMENT API DISCOVERY
- Confirmed AssetPanda v3 document attachment endpoints work with Rocky's existing AssetPanda bearer token for Assets entity `208780`:
  - `GET /v3/attachments?entity_id=208780&entity_object_id=<object_id>&type=Document`
  - `POST /v3/group/objects/<object_id>/attachments` with multipart `file=<pdf>` and `type=Document`
- Verified successful pilot upload of `gisou.pdf` to AssetPanda asset `377` / object `69d679692742932aca2e2ada` with returned document id `9910130`.

SHAREPOINT INVENTORY FORMS PROCESSING
- Used Microsoft Graph SharePoint access against Matt's `AssetPanda/Inventory Forms` folder and the `new forms` subfolder containing signed staff laptop inventory PDFs.
- Built local OCR/download/render/match processing support under:
  - `/home/aiadmin/.openclaw/workspace/tmp/inventory_forms_new/`
  - `/home/aiadmin/.openclaw/workspace/tmp/assetpanda_process_new_forms.py`
- Added audit outputs for verification and follow-up:
  - `process_new_forms_result_verified.json`
  - `unmatched_new_forms.csv`
  - `laptops_missing_pdf_forms.csv`

CAPABILITY ADDED
- Rocky can now process signed laptop inventory PDFs from SharePoint, OCR/render forms as needed, match forms to AssetPanda laptop asset records, upload the signed PDFs as AssetPanda Document attachments, and verify the resulting document links through the AssetPanda API.

WORK COMPLETED
- Original `Inventory Forms` folder: OCR/matched, uploaded, and verified 5 PDFs.
- `new forms` subfolder: found 69 PDFs; exactly matched, uploaded, and verified 52; skipped 17 unmatched forms with no ambiguous uploads.
- Produced follow-up lists for unmatched forms and laptop records still missing any PDF document attachment.


====================================================
2026-05-20 (Session 33 — Zoom meeting-history retention investigation)
====================================================

ZOOM MEETING HISTORY CHECK
- Investigated whether `rpatnode@nolsw.org` appeared as a participant in meetings hosted by `mprovost@slsct.org`.
- Confirmed `mprovost@slsct.org` is visible through the existing Zoom API connection.
- Zoom returned 17 hosted meetings for that user in December 2025; participant checks on those returned meetings found 0 matches for `rpatnode@nolsw.org`.
- For January-November 2025, Zoom's live report API returned the retention-window error that reports can only be queried for a month within the last six months.

CAPABILITY STATUS
- No new platform connection or configuration was added today.
- The existing Zoom meeting-report access was verified, and its practical retention limit was documented for future searches: older meeting participant history may require calendar records, a saved Zoom export/archive, or another retained source.
====================================================
2026-05-26 (Session 34 — MS365 mailbox/archive search investigation)
====================================================

MS365 MAILBOX SEARCH INVESTIGATION
- Matt asked Rocky to search his mailbox/online archive for evidence of when SLS-CT staff went work-from-home during COVID.
- Used the existing Microsoft 365 / Microsoft Graph application access for `mdugan@slsct.org` as Sensitive read access.
- Built temporary local mailbox-search helpers under the workspace `tmp/` area to query mail folders and search terms without committing mailbox contents.
- Confirmed the existing app registration also has an Exchange Online app token role available: `Exchange.ManageAsApp`.

CAPABILITY / LIMITATION DOCUMENTED
- Rocky can perform targeted Microsoft Graph mailbox searches against Matt's accessible mailbox and summarize results without exposing full private email bodies.
- Microsoft Graph folder/message access did not expose useful Online Archive contents for this investigation; visible archive-related folders returned no listed items for the tested paths.
- Microsoft Graph Search API for messages was not usable with the current application-permission setup for mail entities, so deeper Online Archive searching may require an Exchange Online / Purview eDiscovery path or delegated/admin-approved tooling.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing MS365 mailbox-search capability was exercised, and the practical limitation around Online Archive access was documented for future searches.


====================================================
2026-05-27 (Session 35 — Conflict-check sender update and Zoom annual answered-call counts)
====================================================

LEGALSERVER CONFLICT CHECK EMAIL SENDER UPDATED
- Updated the production Vercel configuration for `legalserver-conflict-check-email` so generated emergency conflict-check emails send from the shared mailbox `slstransfers@slsct.org` instead of Matt's mailbox.
- Confirmed Microsoft Graph app access to `slstransfers@slsct.org` with a non-sending mailbox probe before making the change.
- Replaced the production `MS365_SEND_MAILBOX` environment variable and redeployed the production alias:
  - `https://legalserver-conflict-check-email.vercel.app`
- Verification completed with the project's check script and Vercel production deployment status.
- No test email was sent during this configuration change.

ZOOM CONTACT CENTER ANNUAL ANSWERED-CALL COUNTS
- Used the existing Zoom Contact Center API access to calculate answered inbound-call counts for Screener-English, Screener-Spanish, and Operator queues for calendar years 2024 and 2025.
- Final 2024 answered queue-occurrence counts:
  - Screener-English: 12,546
  - Screener-Spanish: 3,470
  - Screener combined: 16,016
  - Operator: 13,377
- Final 2025 answered queue-occurrence counts:
  - Screener-English: 16,762
  - Screener-Spanish: 2,874
  - Screener combined: 19,636
  - Operator: 8,683
- Added reusable local helper/checkpoint scripts for annual Zoom answered-call count scans under the workspace `tmp/` area.

CAPABILITY STATUS
- New production configuration added: the LegalServer conflict-check email app now uses the shared transfer mailbox as its sender through the `MS365_SEND_MAILBOX` setting.
- Existing Zoom Contact Center reporting capability was extended with reusable annual answered-call count tooling.


====================================================
2026-05-28 (Session 36 — SharePoint PST search staging)
====================================================

PST SEARCH WORKFLOW STAGED
- Matt asked whether Rocky can search Outlook mailboxes where Matt has delegate access; clarified that Rocky can only search mailboxes exposed through the configured Microsoft 365 / Microsoft Graph access, not automatically every mailbox Matt can personally open.
- Matt asked whether uploading a `.pst` file to SharePoint would allow Rocky to search it. Confirmed that an approved SharePoint location can be used as a read-only staging path for targeted PST extraction/search work.
- Matt created the SharePoint folder `MattRocky / Documents / PST Search` for the PST upload.
- Rocky checked the folder through Microsoft Graph and confirmed the `PST Search` folder exists; at check time the folder was still empty because the PST upload/sync had not completed.
- Target search was defined for a single email with subject approximately/exactly `2/9/21 schedule`, dated 2021-02-09 around 9:37 AM, with export desired as `.eml` or `.msg` if found.

CAPABILITY STATUS
- No new external platform connection or production configuration was added today.
- Existing SharePoint / Microsoft Graph access was confirmed usable for a controlled PST staging workflow.
- PST extraction/search tooling was not installed yet; next step is to add or use suitable local PST tooling once the file is available, then extract only the requested target email.


====================================================
2026-05-30 (Session 37 — SharePoint image OCR to highlighted Word document)
====================================================

SHAREPOINT OCR DOCUMENT CREATION
- Matt created a new SharePoint folder named `Kayla` in the Matt & Rocky SharePoint site and asked Rocky to OCR the images into a Word document.
- Used existing Microsoft Graph / SharePoint access to list and download 21 image files from the `Kayla` folder.
- Used the local PaddleOCR/Python environment to OCR the images, including rotation handling for sideways screenshots.
- Built a local image-processing workflow to detect visible colored circle/marker regions in the source images and apply matching text shading in the generated Word document.
- Generated `Kayla OCR Text.docx` with:
  - 21 processed images
  - 346 OCR text lines
  - 237 color-shaded lines
  - low-confidence OCR line tracking for review
- Uploaded the finished Word document back to the same SharePoint folder through Microsoft Graph.
- Verified the uploaded file exists in SharePoint with the expected name and size.

CAPABILITY ADDED
- Rocky can now process a SharePoint folder of image files, perform OCR locally, infer simple colored annotation groupings, generate a formatted `.docx` with highlighted/shaded text blocks, and upload the finished document back to SharePoint.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing SharePoint / Microsoft Graph access was extended into a repeatable OCR-to-Word document workflow for image batches.


====================================================
2026-06-01 (Session 38 — Staff ISP Reports automation and survey document conversion)
====================================================

STAFF ISP REPORTS / ZOOM SIGN-IN-OUT REPORTING
- Matt created a SharePoint folder named `Staff ISP Reports` in the MattRocky site and uploaded an example workbook, `December 2025 - processed_ip_data.xlsx`, as the target format.
- Verified existing Microsoft Graph / SharePoint access can read and write the new folder.
- Built local reporting script:
  - `/home/aiadmin/.openclaw/workspace/staff_isp_report.py`
- The script pulls Zoom account sign-in/sign-out activity from `/v2/report/activities`, enriches unique IP addresses locally with a persistent cache, writes Excel workbooks matching Matt's example format, and can upload the generated workbook to SharePoint.
- Required Zoom Server-to-Server OAuth scope added by Matt and verified:
  - `report:read:user_activities:admin`
- Output columns now match the example workbook:
  - User Email, Time, Type, IP Address, Client Type, Version, Provider, City, State, Company
- Generated and uploaded May 2026 report to MattRocky SharePoint `Staff ISP Reports`:
  - `May 2026 - processed_ip_data.xlsx`
  - 1,987 rows, 170 unique IPs, 0 blank providers
  - Sign in: 1,663; Sign out: 324
- Backfilled and uploaded January-April 2026 reports to the same SharePoint folder:
  - January: 1,552 rows / 401 unique IPs
  - February: 1,606 rows / 409 unique IPs
  - March: 2,148 rows / 464 unique IPs
  - April: 2,208 rows / 220 unique IPs
- Verified no recurring OpenClaw or local cron job exists yet for Staff ISP Reports.
- Recommended future schedule: run monthly on the 1st, generating/uploading the previous calendar month.

CAPABILITY ADDED
- Rocky can now generate monthly Staff ISP sign-in/sign-out reports from Zoom activity logs, enrich IP metadata locally with caching, produce Excel workbooks in SLS-CT's requested format, and upload the reports to SharePoint.

YULAA SURVEY DOCUMENT CONVERSION
- Matt uploaded an Excel workbook containing a YULAA survey layout and asked Rocky to convert it to Word.
- Created `YULAA Survey.docx` from the workbook while preserving question order and listing available choice options.
- Verified the DOCX package and confirmed LibreOffice could convert/open it successfully.
- Uploaded `YULAA Survey.docx` to the root of the MattRocky SharePoint Documents library at Matt's request and verified it through Microsoft Graph.

CAPABILITY STATUS
- New Zoom reporting scope verified for account activity reports.
- Existing SharePoint / Microsoft Graph access was used for controlled report/document upload workflows.
- No recurring Staff ISP automation has been scheduled yet.

====================================================
2026-06-02 (Session 39 — Milford residential property-purchase exports)
====================================================

PUBLIC PROPERTY PURCHASE EXPORTS
- Matt asked whether Rocky could gather recent residential property-purchase data for Milford, CT.
- Avoided private MLS scraping and access-control bypasses; used public/official-ish property records instead.
- Investigated Milford land records/IQS, Milford GIS/ArcGIS, CT open data, and Vision Government Solutions.
- Found Milford GIS/ArcGIS sale data was stale through 2021, while Vision Government Solutions provided usable residential and condominium sale results.
- Built local extraction scripts under the workspace `tmp/` area for 2025 and 2026 Milford residential purchase exports.
- Generated Excel exports containing buyer/owner and purchased-property details:
  - `Milford CT 2025 Residential Property Purchases.xlsx` — 497 rows; 375 Residential and 122 Res Condo; sale dates 2025-01-03 through 2025-12-30.
  - `Milford CT 2026 Residential Property Purchases.xlsx` — 93 rows; 61 Residential and 32 Res Condo; sale dates available at run time 2026-01-02 through 2026-04-06.
- Uploaded both workbooks to the MattRocky SharePoint site. Matt moved them to the `MLS Scrape` folder; use that SharePoint folder for future property-purchase exports.

CAPABILITY ADDED
- Rocky can now produce controlled residential property-purchase Excel exports from public municipal/property-record sources, avoiding private MLS systems and documenting source limitations.
- Data handling note: buyer names and property addresses are public-record information but still personal information; future exports should stay purpose-limited and avoid unnecessary columns.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing SharePoint / Microsoft Graph access was used to deliver the generated workbooks.

====================================================
2026-06-03 (Session 40 — YULAA Judicial extraction workflow, policy updates, and LegalServer IFAR note batch)
====================================================

YULAA JUDICIAL EXTRACTION PROTOTYPE
- Started the YULAA Project workflow using a SharePoint project folder and a LegalServer API report as the source of eligible matters.
- Confirmed authenticated LegalServer LIVE API reads can retrieve source report rows, matter UUIDs, notes, docket URLs, legal problem codes, and relevant custom fields needed for YULAA processing.
- Built local prototype artifacts under the workspace `tmp/` area, including a temporary SQLite processing database and Python scripts for case review documents and exception-log generation.
- Confirmed CT Judicial docket pages and attached PDF documents can be inspected for disposition and stipulation data.
- Confirmed many Judicial complaint/stipulation PDFs are scanned/image-only, and reused the local PaddleOCR environment for reliable OCR extraction.
- Added logic to distinguish normal Judicial pages from pages that return HTTP 200 but state the case is not found or not electronically available.

YULAA EXTRACTION RULES ESTABLISHED
- Defendant first advised by SLS: use the earliest IFAR note date from LegalServer notes API; do not use the report `date_posted` field or earliest general note date.
- Housing subsidy: mark Yes if any IFAR note contains `#housingsubsidy#` or if the Legal Problem Code starts with `61` or `64`.
- Stay of Execution through date: select the latest valid Judicial stipulation document, OCR the PDF, and extract the stay-through date from the stipulation text.
- LegalServer custom-field write rule: only set YULAA processed fields after Judicial data is successfully pulled. This was dry-run tested only; no YULAA LegalServer custom-field write-back was performed.

SHAREPOINT YULAA OUTPUTS
- Created and iterated SharePoint YULAA Project output documents, including a merged exception log and a one-case question-data review document.
- Replaced the initial separate exception logs with a single `LOG - YULAA Exceptions.docx` document containing sections for missing docket URL, Judicial case not found, Judicial without disposition, and missing IFAR notes.
- Cleared sample case-specific rows from the merged exception log after extraction was verified, preserving formatting and section structure.
- Verified generated YULAA documents avoid `Created by Rocky` attribution text.

YULAA AUTOMATION CONFIGURATION
- Created OpenClaw cron job `YULAA weekly Judicial extraction` to run Mondays at 8:00 AM America/New_York.
- Guardrails for the weekly job: LegalServer reads only, CT Judicial reads/OCR, local temp artifacts, and SharePoint YULAA Project output/log updates are allowed; no LegalServer write-back without explicit approval.
- Configured the weekly workflow notes so, after updating the SharePoint exception log, the updated copy should be emailed to Matt and Ally Stratos only for this specific YULAA delivery workflow.
- Documented the user-facing project aliases Matt/Ally can use: `YULAA Project` and `YULAA weekly Judicial extraction app`.

SECURITY / PRIVACY DECISIONS UPDATED
- Reviewed the SharePoint `Security-Privacy-Decisions.txt` file and found it was behind current operating decisions.
- With Matt's approval, updated the SharePoint document to `Last Updated: 2026-06-03` and expanded it through Decisions #1-#16.
- Added durable policy notes covering Ally access boundaries, SharePoint handling, mailbox/wiki guardrails, credential-storage standards, SMS/Twilio boundaries, LegalServer write/dry-run posture, public-record export minimization, Rocky's inability to change its own permissions, and the YULAA Judicial-success-before-write rule.

LEGALSERVER IFAR HOUSING-SUBSIDY NOTE BATCH
- Matt provided a spreadsheet of 42 unique LegalServer matter/case IDs and explicitly approved adding IFAR notes with the `#housingsubsidy#` tag.
- Preflight resolved all 42 cases and confirmed none already had an IFAR note containing the tag.
- First attempted note creation was rejected by LegalServer because the required subject field was missing; no notes were created in that attempt.
- Retried with both subject and body set to `#housingsubsidy#`; 42 of 42 IFAR notes were created and verified.
- Local audit output was saved in the workspace `tmp/` area for traceability.

CAPABILITY ADDED
- Rocky can now prototype and run a controlled YULAA Judicial extraction pipeline combining LegalServer report/API reads, CT Judicial docket/document reads, OCR of scanned Judicial PDFs, SharePoint Word output generation, exception-log maintenance, and scheduled weekly orchestration with explicit LegalServer write guardrails.
- Rocky can perform approved, preflighted LegalServer IFAR note batch writes with verification and local audit output.


====================================================
2026-06-05 (Session 41 — Twilio outgoing SMS report and YULAA continuity checkpoint)
====================================================

TWILIO OUTGOING SMS REPORTING
- Matt asked whether Rocky can run Twilio reports and then requested all outgoing SMS messages grouped by sending phone number.
- Confirmed existing Twilio API access is usable for read-only reporting while preserving the permanent SMS guardrails: no Twilio settings changes, no outbound texts except to Matt's approved number, and no message-body/recipient-number collection unless explicitly justified.
- Built a local metadata-only reporting script under the workspace `tmp/` area:
  - `/home/aiadmin/.openclaw/workspace/tmp/twilio_outgoing_by_from_fast.py`
- Initial all-message pagination attempt hit a five-minute safety timeout; switched to a safer report method that lists current Twilio IncomingPhoneNumbers and queries Messages filtered by each `From` number.
- Generated final CSV:
  - `/home/aiadmin/.openclaw/workspace/tmp/twilio_reports/outgoing_sms_by_sending_number_20260605_132526.csv`
- Report scope/results:
  - 17 current Twilio sending numbers checked
  - 323,607 outgoing SMS messages
  - 392,925 message segments
  - Estimated Twilio message charges: about $1,788.23, based on Twilio price metadata
- Main operational red flag identified: sending number `+14752553552` accounted for 317,954 outgoing messages and had very high failed/undelivered counts, worth deeper follow-up if Matt asks.

YULAA CONTINUITY CHECKPOINT
- Verified the current YULAA housing-subsidy question logic against the actual local script, not just memory.
- Current implementation marks housing subsidy `Yes` if any IFAR note contains literal `#housingsubsidy#`, or if the Legal Problem Code starts with `61` or `64`; otherwise it returns `No`.
- Important caveat documented: the tag check is currently exact lowercase, so mixed-case variants like `#HousingSubsidy#` would not match unless patched to case-insensitive matching.
- Confirmed sample case `25-0374051` returns housing subsidy `Yes` because Legal Problem Code starts with `64`, even without the IFAR tag.
- Documented handoff guidance for Ally's isolated Rocky session: use the project anchors `YULAA Project` or `YULAA weekly Judicial extraction app`, plus the SharePoint YULAA Project documents and local `tmp/yulaa_*` artifacts.

CAPABILITY ADDED
- Rocky can now produce aggregate Twilio outgoing SMS reports by sending number using existing Twilio API access, with privacy-preserving defaults that avoid message bodies and recipient-level data.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing Twilio reporting access was extended into a reusable outgoing-SMS-by-sender reporting workflow.
- Existing YULAA logic and project handoff anchors were verified and documented for continuity across isolated sessions.

====================================================
2026-06-08 (Session 42 — YULAA production batch run and LegalServer note-type cleanup)
====================================================

YULAA PRODUCTION BATCH RUN
- Matt approved running the expanded YULAA workflow as a background/batched production pipeline against the LegalServer YULAA report.
- Established production guardrails: skip and log cases missing docket URLs, invalid/unusable CT Judicial pages, Judicial pages without disposition, and retryable processing errors; only successfully processed cases are imported/updated in monday.com and marked processed in LegalServer.
- Processed an unexpectedly large LegalServer report result set of 5,758 rows using an 8-worker local batch pattern instead of one large OCR/process run.
- Final production outcome:
  - 6 cases imported/updated in monday.com and verified.
  - 6 corresponding LegalServer YULAA write-backs verified with `yulaa_yes_no_471=true` and `yulaa_processed_date_472=2026-06-08`.
  - 3,279 cases logged as missing docket URL.
  - 2,471 cases logged as invalid/not-found Judicial page or bad URL value.
  - 2 cases remained as processing errors after retry/final processing.
- Corrected a final import issue where several monday.com disposition/date/stay fields were initially blank after a re-fetch parsing miss; patched the six active monday.com rows from batch-confirmed evidence and verified disposition data plus ZIP code mapping.
- Regenerated the corrected `LOG - YULAA Exceptions.docx`, uploaded it to the SharePoint YULAA Project folder, and emailed the corrected log to Matt and Ally Stratos.
- Removed the temporary YULAA batch monitor cron after completion so it would stop rechecking.

LEGALSERVER CUSTOM FIELD CLEARING LESSON
- Confirmed that LegalServer v2 top-level `null` updates can return success while not clearing UI-backed custom field storage.
- Confirmed the reliable clearing pattern for these LegalServer custom fields: v1 `PATCH /api/v1/matters/{case_uuid}` with `custom_fields` keyed by database field name and value `%empty%`.
- Applied the fix narrowly to live case `26-0426847`, clearing `yulaa_yes_no_471` and `yulaa_processed_date_472`, then verified both fields were null through the reliable custom-field query path.

LEGALSERVER AIDA ARUS NOTE-TYPE CLEANUP
- Matt approved a live LegalServer cleanup for report load 7813: change Aida Arus-created notes from `Case Notes` to `IFAR`.
- Built and ran a guarded update that changed only actual note records where the creator was Aida Arus / user ID 98 and current note type was `Case Notes`.
- Updated and verified 937 of 937 targeted notes to IFAR with 0 errors.
- Independent post-update rescan showed no remaining Aida-created `Case Notes` in the same matter set; the refreshed original report returned only 7 non-Case-Notes rows.

CAPABILITY ADDED
- Rocky can now run the YULAA workflow as a controlled batched production pipeline across thousands of LegalServer report rows, combining LegalServer reads/writes, CT Judicial docket/PDF extraction, OCR, monday.com updates, SharePoint exception-log delivery, and limited email distribution with verification gates.
- Rocky now has a verified LegalServer pattern for clearing UI-backed custom fields using the v1 `%empty%` sentinel when JSON `null` is ineffective.
- Rocky can perform approved, creator-scoped LegalServer note-type batch cleanups with preflight narrowing, live update verification, and post-update rescans.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing LegalServer, CT Judicial, monday.com, SharePoint, and MS365 email capabilities were combined into larger verified production workflows.

====================================================
2026-06-09 (Session 43 — YULAA docket recovery, LOG improvements, and monday.com map ZIP configuration)
====================================================

YULAA MISSING-DOCKET RECOVERY ENHANCEMENTS
- Expanded the YULAA workflow for cases missing `docket_url_174` in LegalServer.
- Added strict CT Judicial party-name/address discovery:
  - Searches public CT Judicial PartySearch only for exact client first/last name candidates.
  - Opens candidate Judicial case-detail pages and accepts a docket URL only when exactly one non-plaintiff party row matches the client's normalized name and mailing address.
  - Leaves ambiguous or incomplete matches skipped/logged instead of guessing.
- Added LegalServer note-derived docket discovery:
  - Scans LegalServer notes for CT civil docket-number patterns when `docket_url_174` is missing.
  - Validates each candidate against CT Judicial and accepts only a single exact client first/last non-plaintiff match.
  - Audit output stores note metadata and docket candidates only, not note bodies.
- Updated the durable batch worker so future YULAA runs try note-derived docket recovery first, then CT Judicial name/address recovery.
- Updated finalization so verified discovered docket URLs can be written back to LegalServer `docket_url_174`, including discovered cases that are later skipped because they have no usable disposition.

YULAA PRODUCTION RUNS AND LIVE CLEANUP
- Ran multiple approved YULAA production/rerun cycles against fresh LegalServer report manifests using the 8-worker batch pattern.
- Imported/created additional verified monday.com rows and marked corresponding LegalServer matters processed only after successful Judicial extraction.
- Performed approved, targeted LegalServer `docket_url_174` backfills for verified docket discoveries from name/address matching and note-derived docket matching.
- Added retry/backoff behavior for LegalServer 429 rate-limit responses during docket URL write-back.
- Added handling for literal `docket_url_174 = NONE`:
  - Treats `NONE` as terminal/no docket will ever exist.
  - Marks the matter YULAA processed in LegalServer.
  - Does not include those terminal NONE cases in the exception LOG.
- Patched finalization to fall back to batch-confirmed disposition data when a final CT Judicial re-fetch misses a disposition block, preventing blank disposition/date fields in monday.com.

YULAA EXCEPTION LOG IMPROVEMENTS
- Updated the SharePoint `YULAA Project/LOG - YULAA Exceptions.docx` workflow so Matter IDs in generated LOG tables are hyperlinks to LegalServer profile URLs.
- Added durable DOCX hyperlink generation to the finalizer for future LOG files.
- Added a first-page Summary table to the LOG with section counts, followed by a page break before detailed case lists.
- Updated the current SharePoint LOG in place with hyperlinks and the summary page, preserving the detailed case listings.
- Patched the finalizer so LOG section-count formatting is generated automatically, avoiding manual correction-email follow-up.

MONDAY.COM MAP ZIP CONFIGURATION
- Confirmed the YULAA monday.com board columns for client ZIP, City, and `Map: Zip Code` location.
- Patched the production Monday create/update workflow so future YULAA rows populate `Map: Zip Code` using ZIP centroid latitude/longitude plus address text from `api.zippopotam.us`.
- Built and ran a backfill for existing board rows with client ZIP values but empty map ZIP values.
- Verified the board ended with all current items that have client ZIP values also populated in the `Map: Zip Code` location column.

CAPABILITY ADDED
- Rocky can now recover missing YULAA docket URLs through two strict, auditable methods: LegalServer note-derived docket validation and CT Judicial name/address exact matching.
- Rocky can write verified recovered docket URLs back to LegalServer with readback verification and rate-limit handling.
- Rocky can treat explicit LegalServer `NONE` docket markers as terminal processed cases without re-logging them indefinitely.
- Rocky can generate YULAA exception LOG documents with summary counts and LegalServer hyperlinks automatically.
- Rocky can populate and backfill monday.com location/map columns using ZIP centroid coordinates for YULAA board mapping.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing LegalServer, CT Judicial, monday.com, SharePoint, and MS365 email capabilities were expanded into a more complete and resilient YULAA production workflow.

====================================================
2026-06-10 (Session 44 — YULAA resilience updates, RTC report source fix, and LegalServer note cleanup)
====================================================

RTC WEEKLY REPORT SOURCE FIX
- Investigated an RTC weekly referrals/demographics report that was unexpectedly returning only 2 cases.
- Confirmed the prior LegalServer saved report source was too narrow for recent weekly ranges and switched the weekly RTC referrals/demographics pipeline to the broader verified LegalServer saved report source.
- Reran the weekly RTC report automation, regenerated the corrected PDF set, uploaded the weekly PDFs, and created a fresh Outlook draft for Matt with the corrected attachments.

YULAA LOG AND HYPERLINK CORRECTIONS
- Clarified the YULAA LOG wording so the Imported / Updated section is explicitly labeled as Most Recent Run rather than cumulative history.
- Corrected LegalServer case-profile hyperlinks in the SharePoint `YULAA Project/LOG - YULAA Exceptions.docx` so generated links use the six-digit LegalServer matter-profile ID format required by the live site.
- Patched the durable YULAA LOG generation scripts so future generated LOG documents preserve the corrected Imported / Updated wording and correct LegalServer hyperlink target format.
- Verified the live SharePoint LOG retained all expected LegalServer hyperlink relationships after correction.

YULAA PRODUCTION RERUNS AND RETRY HANDLING
- Ran an approved full live YULAA rerun and delivered the updated LOG to SharePoint and by email to Matt and Ally after resolving a temporary SharePoint file lock.
- Reprocessed cases previously classified as Invalid / Not Found after Matt identified examples where CT Judicial pages were actually usable.
- Added a targeted invalid/not-found reprocess workflow that extracts affected cases from canonical batch output, reruns them in batches, merges corrected outcomes, and then finalizes normally.
- Separated CT Judicial transient ASP.NET error pages from true Invalid / Not Found results:
  - Detects HTTP-200 Judicial `Error Page` responses as retryable transient failures.
  - Retries detail-page reads with backoff.
  - Keeps persistent transient Judicial failures out of the Invalid / Not Found LOG category.
- Lowered future YULAA rerun/reprocess concurrency to reduce CT Judicial transient error responses.

YULAA LOCAL HOURLY RETRY CRON
- Added a local OS cron retry wrapper for YULAA invalid-case rechecks every hour on the hour without using model tokens.
- The wrapper uses a file lock, writes local logs, probes a known-good CT Judicial docket before running, and only starts the retry process when Judicial appears healthy.
- Confirmed the wrapper safely skips processing when CT Judicial is returning transient Error Page responses.

LEGALSERVER NOTE-TYPE CLEANUP
- Completed an approved LegalServer note-type cleanup for refreshed report 7813.
- Matched current report rows to live note UUIDs by matter and creator, then updated verified James Winkel-created `Case Notes` entries to `IFAR`.
- Verified all targeted note updates after PATCH and kept local audit files without storing or printing note bodies.

YULAA ARCHITECTURE CHECKPOINT
- Documented the current high-level YULAA architecture for continuity:
  - LegalServer report/API source.
  - Missing docket recovery.
  - CT Judicial validation, extraction, and OCR.
  - monday.com row create/update.
  - LegalServer write-back only after successful/approved criteria.
  - SharePoint LOG generation and limited email delivery.
- Confirmed the active durable scripts and weekly OpenClaw cron anchor for future YULAA maintenance.

CAPABILITY ADDED
- Rocky can now distinguish CT Judicial transient Error Page responses from true invalid/not-found docket pages and route them to retry-later handling instead of permanent exception categories.
- Rocky can run a local, token-free hourly YULAA retry wrapper with health probes and locking so Judicial outages do not trigger bad writes or unnecessary finalization.
- Rocky can regenerate YULAA LOG hyperlinks using the correct LegalServer matter-profile ID format and preserve corrected LOG section labels automatically.
- Rocky can repair and rerun targeted YULAA invalid/not-found batches from canonical batch artifacts instead of requiring a full workflow rerun every time.
- Rocky can perform approved LegalServer note-type cleanup for refreshed report exports with exact matching, serialized live writes, verification, and privacy-preserving audit output.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing LegalServer, CT Judicial, monday.com, SharePoint, MS365 email, Outlook draft, and local cron capabilities were hardened and combined into more resilient production workflows.

====================================================
2026-06-11 (Session 45 — YULAA retry guardrails, LegalServer IFAR cleanup, and WHPS registration checker)
====================================================

YULAA HOURLY RETRY AND FINALIZER HARDENING
- Fixed the YULAA finalizer's monday.com verification step after the monday.com `items(ids: [...])` GraphQL query failed on large ID batches.
- Patched verification to use smaller 25-item chunks after testing showed larger chunks either crashed or returned incomplete results.
- Temporarily disabled the local hourly YULAA retry cron during the repair/rerun window, then restored and verified the cron entry afterward:
  - `0 * * * * /home/aiadmin/.openclaw/workspace/tmp/yulaa_hourly_invalid_retry.sh`
- Reran the YULAA finalizer successfully and verified the clean result:
  - 161 imported/updated monday.com items.
  - 161 of 161 monday.com rows verified.
  - 161 LegalServer results.
  - 0 processing errors.
- Uploaded the corrected SharePoint `YULAA Project/LOG - YULAA Exceptions.docx` and sent the status email to Matt and Ally.

YULAA DELIVERY SAFETY GUARD
- Added a safety guard so hourly invalid/not-found retry runs cannot overwrite the canonical SharePoint LOG or email a misleading result when final processing errors remain.
- Patched the YULAA finalizer to honor `YULAA_SKIP_DELIVERY_ON_PROCESSING_ERRORS=1`:
  - If final processing has remaining errors, the run writes a local summary.
  - SharePoint upload and email delivery are skipped for that errored retry result.
- Patched the local hourly retry wrapper to set this guard variable automatically.
- Verified syntax and restored a clean canonical LOG/email result after the guarded change.

LEGALSERVER REPORT 7813 IFAR NOTE-TYPE CLEANUP
- Completed an approved live LegalServer note-type cleanup for refreshed report 7813.
- Pulled the authenticated report export and narrowed the update to verified Emma Lopez-created notes currently marked `Case Notes`.
- Dry-run matched target notes by matter and creator while excluding unrelated live Case Notes in the same matters.
- After Matt confirmed live execution, updated 233 notes from `Case Notes` to `IFAR` via the LegalServer notes API.
- Verified all 233 targeted notes as IFAR with 0 errors and preserved privacy by not printing or storing note bodies.

WHPS 2026-2027 REGISTRATION CHECKER
- Created a new OpenClaw cron job to watch for West Haven Public Schools general/non-kindergarten 2026-2027 new student registration opening.
- Job name: `WHPS 2026-2027 new student registration checker`.
- Schedule: daily at 8:00 AM America/New_York.
- Guardrail: do not treat the separate 26-27 Kindergarten Registration page as success.
- Success behavior: notify Matt through allowed safe channels with the correct registration link, then disable/remove the checker after notification.

CAPABILITY ADDED
- Rocky can now run YULAA hourly retry finalization with a delivery guard that prevents errored retry runs from replacing the canonical SharePoint LOG or sending misleading emails.
- Rocky can verify larger monday.com result sets more safely by chunking item lookups into smaller batches.
- Rocky can create targeted website-monitoring cron jobs for time-sensitive personal/administrative needs, with explicit success criteria and safe notification/removal behavior.
- Rocky can continue creator-scoped LegalServer note-type cleanups using exact matching, dry-run review, explicit approval, live update verification, and privacy-preserving audit output.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing LegalServer, monday.com, SharePoint, MS365 email, CT Judicial, local cron, and OpenClaw cron capabilities were hardened and extended into safer recurring operations.
====================================================
2026-06-12 (Session 46 — YULAA hourly retry optimization and duplicate-processing guard)
====================================================

YULAA HOURLY RETRY OPTIMIZATION
- Hardened the local YULAA hourly invalid/not-found retry flow so already-processed eligible cases are not rechecked or redelivered every hour.
- Patched the YULAA finalizer to live-check LegalServer custom fields before final eligible-case processing:
  - `yulaa_yes_no_471`
  - `yulaa_processed_date_472`
- Added logic to skip eligible cases that are already marked processed in LegalServer.
- Added logic to skip terminal no-docket writebacks that are already marked processed.
- Made monday.com verification handling safer when there are no Monday items to verify.
- Added `already_processed_skipped_count` to YULAA run summaries for clearer auditability.

DELIVERY GUARDRAIL
- Patched the hourly retry wrapper to set `YULAA_SKIP_DELIVERY_ON_NO_NEW=1`.
- With that guard enabled, hourly retry runs skip SharePoint LOG upload and email delivery when there are no newly processed cases.
- This prevents repeated hourly emails/LOG refreshes when the retry job only rediscovers cases already marked complete.

VERIFICATION
- Verified Python syntax with `py_compile`.
- Verified shell wrapper syntax with `bash -n`.
- Spot-checked the first 10 current eligible LegalServer cases and confirmed all were already marked processed with processed date `2026-06-12`.

CAPABILITY ADDED
- Rocky can now run YULAA hourly retry maintenance with duplicate-processing detection based on LegalServer processed flags.
- Rocky can suppress no-new-work YULAA hourly deliveries while still keeping local retry/audit behavior available.
- YULAA run summaries now expose skipped already-processed counts, making recurring retry runs easier to interpret.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing LegalServer, monday.com, SharePoint/email delivery, and local cron capabilities were hardened to reduce duplicate checks and noisy retry output.
====================================================
2026-06-15 (Session 47 — YULAA weekly delivery and report filter hardening)
====================================================

YULAA WEEKLY JUDICIAL EXTRACTION DELIVERY
- Verified the weekly YULAA run artifact for the June 15 12:00 UTC scheduled batch.
- Confirmed there were no new LegalServer imports or updates in that run:
  - report rows reviewed: 5,774
  - already-processed cases skipped: 161
  - final imported/updated count: 0
  - processing errors after retry: 0
- Regenerated the SharePoint LOG document locally with Last Run Date `2026-06-15 12:00 UTC`.
- Uploaded the refreshed `YULAA Project/LOG - YULAA Exceptions.docx` to SharePoint.
- Emailed the updated LOG attachment to the approved recipients only: Matt Dugan and Ally Stratos.
- Confirmed the delivery step performed no LegalServer writes.

YULAA REPORT FILTER HARDENING
- Found that the standalone read-only weekly runner could fail when the SharePoint `Project YULAA.txt` report URL included a blank LegalServer report filter parameter.
- Patched the local weekly runner to sanitize/drop blank report filters before requesting the LegalServer report export.
- Avoided hours of duplicate read-only crawling by stopping the restarted standalone runner once the already-complete guarded batch result was verified and delivered.

CAPABILITY ADDED
- Rocky can now tolerate blank LegalServer report-filter parameters in the standalone YULAA weekly runner by sanitizing the report URL before export requests.
- Rocky can safely deliver a no-new-cases weekly YULAA LOG update while preserving the no-write delivery guarantee.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing LegalServer, CT Judicial, SharePoint, MS365 email, and YULAA cron/reporting capabilities were maintained and hardened.

====================================================
2026-06-25 (Session 48 — YULAA housing subsidy tag support and retry cron restoration)
====================================================

YULAA HOUSING SUBSIDY DETECTION HARDENING
- Added `#HS` as an alternate IFAR note tag for YULAA housing subsidy detection.
- Updated the local YULAA production, finalization, weekly-run, documentation, and single-case read-only scripts so IFAR notes containing either `#housingsubsidy#` or `#HS` count as housing subsidy Yes.
- Preserved the existing Legal Problem Code 61/64 housing-subsidy logic alongside the new shorthand tag support.
- Verified the patched Python files with `py_compile` and a tag-regex smoke test.

YULAA RUN/CRON STATUS
- Started a full YULAA production rerun, but it was interrupted before batch output/finalizer completion; no completion summary or delivery should be assumed from that interrupted run.
- Re-enabled the hourly invalid retry cron after the interrupted run so normal retry maintenance could resume.

CAPABILITY ADDED
- Rocky can now recognize both the long-form `#housingsubsidy#` marker and the shorthand `#HS` marker when determining YULAA housing subsidy status from IFAR notes.
- YULAA hourly invalid retry maintenance was restored after the interrupted production rerun.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing LegalServer, CT Judicial, YULAA local automation, and local cron capabilities were hardened and maintained.

====================================================
2026-06-26 (Session 49 — YULAA NONE docket watchlist workflow)
====================================================

YULAA NONE DOCKET WORKFLOW HARDENING
- Changed the local YULAA workflow so literal `docket_url_174 = NONE` cases are no longer treated as immediately processed terminal cases.
- Added a local watchlist-backed review flow for NONE docket cases using `tmp/yulaa_none_docket_watchlist.json`.
- The batch worker now checks watched NONE cases against CT Judicial exact name/address matching at 30, 60, and 90 days from first seen.
- If a Judicial match is found during a watchlist check, the workflow writes the discovered docket URL back through the existing exact-match docket URL path.
- Only after the 90-day check finds no match does the workflow emit the terminal `terminal_no_docket_url_none` outcome.
- Updated the finalizer so it maintains the local watchlist and marks terminal NONE cases processed only after the 90-day no-match result.

VERIFICATION
- Verified updated Python files with `py_compile`.
- Ran a synthetic smoke test confirming newly seen NONE docket cases are not due immediately and that a 90-day watched case is due for review.

CAPABILITY ADDED
- Rocky can now defer and recheck YULAA cases whose LegalServer docket URL field is explicitly `NONE`, reducing premature terminal processing when a docket may appear later.
- YULAA NONE docket handling now has an auditable local watchlist and staged 30/60/90-day CT Judicial review logic.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing LegalServer, CT Judicial, and YULAA local automation capabilities were hardened with safer delayed-review behavior.

====================================================
2026-06-27 (Session 50 — YULAA full rerun monitoring and continuity checkpoint)
====================================================

YULAA FULL RERUN STATUS CHECKPOINT
- Checked the active local YULAA full rerun after Matt asked for status in Discord.
- Confirmed the rerun had started around 2026-06-27 00:22 UTC and was still in the batch-worker phase.
- Snapshot during the session:
  - 5,758 total report rows in the batch manifest.
  - About 3,550 rows processed (~61.7%).
  - 188 eligible cases found so far.
  - 3,361 skipped cases so far.
  - 1 worker error caused by a malformed docket URL value with no URL scheme.
- Clarified that the mid-run eligible count had not yet been processed into monday.com.
- Verified that monday.com writes occur later in `tmp/yulaa_finalize_batches.py` after workers complete, not during worker discovery.

CONTINUITY AND FOLLOW-UP
- Wrote a reset-safe session checkpoint to local memory for continuity before Matt reset the active Discord session.
- Scheduled a one-shot read-only follow-up check for 2026-06-27 06:15 UTC to inspect whether the run completed, finalized, or hit a blocker.
- The follow-up was configured to notify Matt only if there was a meaningful change.

CAPABILITY STATUS
- No new external platform connection was added today.
- No new production write capability was added today.
- Existing YULAA monitoring, memory checkpointing, and OpenClaw cron follow-up capabilities were used to preserve run continuity during a long-running production rerun.

====================================================
2026-06-28 (Session 51 — YULAA resumable finalizer hardening and MCP planning)
====================================================

YULAA RESUMABLE FINALIZER HARDENING
- Hardened the YULAA full-rerun finalization path after the June 27 507-case rerun exposed fragility in the old finalizer timeout model.
- Added durable per-case checkpoint directories for final-prepped cases, monday.com import/update results, and LegalServer write-back results.
- Added progress logging to `tmp/yulaa_batch/finalizer_state/progress.jsonl` so long finalizer runs can be inspected and resumed safely after interruption.
- Added `tmp/yulaa_run_finalizer_resumable.sh`, a guarded runner with a lockfile, timestamped stdout/stderr logs, metadata capture, and exit-code recording.
- Patched the full-rerun orchestrator so future reruns call the resumable runner instead of the old 30-minute timeout finalizer subprocess.
- Verified the patched Python and shell files with syntax/smoke checks.

YULAA 507-CASE RERUN COMPLETION
- Ran the hardened finalizer against the interrupted 507-case rerun.
- First pass successfully imported/updated 493 cases in monday.com, wrote back 493 cases to LegalServer, uploaded the SharePoint LOG, and emailed Matt and Ally.
- Retried the 14 remaining transient Judicial/final-prep connection failures through the same resumable finalizer path.
- Final durable state reached 507/507 final-prepped, 507/507 monday.com imported/updated, and 507/507 LegalServer write-backed.
- Confirmed no active finalizer process remained after completion.

MCP INTERNAL TOOLING PLANNING
- Researched whether existing open-source Zoom MCP servers cover Zoom Contact Center.
- Found generic/meeting-focused Zoom MCP projects, but no ready-made open-source Zoom Contact Center MCP suitable for SLS-CT needs.
- Inspected Zoom's official plugin/API references and confirmed broad Zoom Contact Center API coverage for queues, users, recordings, engagements, reports, dispositions, flows, roles, routing profiles, skills, teams, variables, and related objects.
- Recommended building a narrow internal read-only Zoom Contact Center MCP instead of trusting a broad third-party Zoom MCP.
- Drafted and emailed Matt recommended steps for a Zoom Contact Center MCP.
- Planned a privacy-first internal LegalServer MCP starting read-only against demo, with allowlisted tools, shaped/minimum-necessary responses, hard limits, no generic endpoint caller, and no writes in v1.
- Drafted and emailed Matt recommended steps for a LegalServer MCP.

CAPABILITY ADDED
- Rocky can now resume long YULAA finalizer runs using durable per-case checkpoints rather than restarting or relying on a single long process.
- YULAA full-rerun finalization now has lock/log/meta/exit capture and an auditable progress trail.
- Rocky can recover transient YULAA final-prep failures through targeted retry passes while preserving already-completed monday.com and LegalServer work.
- Rocky has a concrete implementation plan for internal read-only MCP servers for Zoom Contact Center and LegalServer, with privacy and scope guardrails appropriate for SLS-CT data.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing LegalServer, monday.com, SharePoint, MS365 email, CT Judicial, and local YULAA automation capabilities were hardened and extended.
- MCP work today was research and implementation planning only; no production MCP server was deployed yet.

====================================================
2026-06-29 (Session 52 — MCP shareable-core standard)
====================================================

MCP REPOSITORY AND CONFIGURATION STANDARD
- Established a durable build standard for future MCP work so internal MCP servers can live in GitHub and be shared later if appropriate.
- Defined the preferred pattern as a shareable MCP core plus an SLS private configuration pack:
  - Core repo: generic MCP server framework, tool definitions, documentation, install/run instructions, `.env.example`, and mock/sample data only.
  - Private config: tenant IDs, API URLs, credentials, SLS-specific field mappings, workflow rules, logs, and real organizational/client data.
- Applied the standard to planned LegalServer/Zoom-style MCPs so SLS-CT needs can be met without exposing private data, secrets, or organization-specific workflows.
- Set the expectation that any MCP repo should start private, keep secrets and real data out of GitHub, and defer public licensing/sharing decisions until reviewed.

CAPABILITY ADDED
- Rocky now has a reusable GitHub-safe MCP project pattern for building internal tools without mixing shareable framework code with SLS-CT-specific sensitive configuration.
- Future LegalServer, Zoom Contact Center, and similar MCP projects can be structured for safer collaboration, cleaner documentation, and easier private/public separation.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing GitHub planning, LegalServer MCP planning, and Zoom Contact Center MCP planning were hardened with a privacy-first repository/configuration standard.

====================================================
2026-06-30 (Session 53 — GHLA alert hardening and LSWork activity report)
====================================================

GHLA REJECTED-TRANSFER ALERT HARDENING
- Investigated why a GHLA rejected-transfer email around 4:29 AM ET did not generate a Zoom Team Chat alert.
- Confirmed the local alert wrapper only runs Monday-Friday during business hours, and the first morning poll saw no rows because the LegalServer report row had already disappeared.
- Updated the GHLA alert design so the LegalServer report can show recent rejection events from the past 7 days while Rocky performs durable script-side de-duplication.
- Patched `ghla_rejected_transfer_alerts/poll_ghla_rejected_transfers.py` to keep sent-alert state in `ghla_rejected_transfer_alerts/out/sent_alerts.json` by default.
- Added default-on `GHLA_ALERT_DEDUPE` behavior plus `GHLA_ALERT_STATE_PATH` override support.
- Built alert keys from matter/case URL, rejection timestamp when present, and reject-reason fields, then records successful Zoom posts immediately to prevent duplicate alerts.
- Updated `ghla_rejected_transfer_alerts/README.md` with the 7-day report requirement and the need for a stable rejection timestamp/date field when possible.
- Verified the updated script with syntax check and dry-run; no Zoom post was sent during the dry-run.

LSWORK ACTIVITY REPORT
- Built `scripts/lswork_activity_report.py`, a request-run LegalServer reporting script for LSWork activity review.
- The script accepts a LegalServer login, runs the Case List For Last Activity report, keeps open cases with blank close dates, checks app-log activity per matter Database ID, filters rows to the selected login, and keeps the latest 5 entries per case.
- Sends a privacy-minimized HTML summary email to `mdugan@slsct.org` only when run with `--send`; dry-runs do not email.
- Moved LSWork report API keys into local `.openclaw/legalserver.env` values instead of hardcoding them in the script.
- Documented the important XML parsing detail that the case-list report has two `<id>` fields: the first is staff/user id and the second is the matter Database ID.
- Dry-run and sent run for login `jcaez` completed successfully; Microsoft Graph sendMail returned 202.

CAPABILITY ADDED
- GHLA rejected-transfer Zoom Team Chat alerts can now catch overnight or recently rejected cases once, even when the LegalServer report shows a rolling multi-day window.
- GHLA alerting now has durable de-duplication state and configurable state-file behavior.
- Rocky can now generate and email a focused LSWork activity summary for a specific LegalServer login, using LegalServer report APIs and Microsoft Graph email delivery.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing LegalServer report API, Zoom Team Chat alerting, local automation, and Microsoft Graph email capabilities were hardened and extended.
- No LegalServer write actions were performed for the LSWork report work.

====================================================
2026-07-01 (Session 54 — June Funnel report and Staff ISP monthly automation)
====================================================

JUNE FUNNEL REPORT DELIVERY
- Processed the June 2026 Monthly Funnel Report after confirming June outputs had not been generated yet.
- Ran the local funnel report automation for year 2026 / month 6.
- Generated June raw data under `funnel_reports/raw_data/2026-06/` and June report outputs under `funnel_reports/output/`.
- Created an Outlook draft in `mdugan@slsct.org` with subject `Updated Monthly Funnel Report` and two chart PDF attachments:
  - `SLS Hotline - Monthly Funnel Report - June 2026.pdf`
  - `RTC - Monthly Funnel Report - June 2026.pdf`
- Final June funnel stage counts:
  - SLS: Stage 1 9,227; Stage 2 8,578; Stage 3 2,541; Stage 4 1,857.
  - RTC: Stage 1 2,207; Stage 2 2,021; Stage 3 731; Stage 4 589.

STAFF ISP / ZOOM SIGN-IN-OUT REPORTING
- Recalled and documented the existing Staff ISP Reports workflow for Zoom sign-in/sign-out activity enrichment.
- Generated and uploaded the June 2026 Staff ISP report to MattRocky SharePoint folder `Staff ISP Reports`:
  - File: `June 2026 - processed_ip_data.xlsx`
  - Rows: 1,850
  - Unique IPs: 180
  - Blank providers: 0
  - Sign-ins: 1,575
  - Sign-outs: 275
- Added monthly wrapper script `cron_monthly_staff_isp_report.sh`, which defaults to the previous calendar month and logs to `logs/staff_isp_report.log`.
- Created OpenClaw cron job `staff-isp-monthly-report` (`97e4e9ea-f7fb-433c-b6ba-71ee6f50bf4c`) scheduled for `0 7 1 * *` in `America/New_York`, matching the monthly RTC report cadence.
- The Staff ISP monthly cron will process and upload the previous full calendar month, with Discord announce delivery to Matt's channel.

CAPABILITY ADDED
- Rocky can now run the Staff ISP / Zoom Sign-In-Out report automatically every month without a manual request.
- Staff ISP reporting is now aligned with the existing monthly RTC report schedule and uses a reusable wrapper plus persistent logging.
- Rocky can generate the Staff ISP workbook for a completed month and upload it directly to the approved MattRocky SharePoint folder.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing Zoom activity reporting, SharePoint upload, Microsoft Graph draft-email, and OpenClaw cron capabilities were used and extended.

====================================================
2026-07-02 (Session 55 — Funnel report January/February correction pass)
====================================================

FUNNEL REPORT CORRECTIONS
- Reran January 2026 Monthly Funnel Report source data and corrected the January history values used in the June 2026 report workbooks.
- Corrected January counts:
  - SLS: Stage 1 8,406; Stage 2 7,854; Stage 3 2,442; Stage 4 1,727.
  - RTC: Stage 1 2,331; Stage 2 2,168; Stage 3 761; Stage 4 617.
- Created an Outlook draft to Matt with January chart PDFs, subject `Updated Monthly Funnel Report`.
- Patched the June 2026 SLS and RTC workbook January columns, regenerated full-report PDFs and chart-only PDFs, uploaded refreshed outputs to SharePoint, and created a follow-up Outlook draft with 4 corrected June report attachments.
- Reran February 2026 Monthly Funnel Report source data and compared it against the existing June workbook history.
- Corrected February counts:
  - SLS: Stage 1 8,088; Stage 2 7,591; Stage 3 2,295; Stage 4 1,615.
  - RTC: Stage 1 2,171; Stage 2 1,993; Stage 3 752; Stage 4 593.
- Because the February values differed from the June workbook, patched the February columns, regenerated June 2026 full-report and chart-only PDFs again, and uploaded the refreshed June outputs plus February raw data to SharePoint.
- No additional email draft was created for the February correction pass.

CAPABILITY STATUS
- No new external platform connection was added today.
- No new automation or durable capability was added today.
- Existing Funnel Report generation, SharePoint upload, and Microsoft Graph draft-email capabilities were used to correct historical report values and refresh June deliverables.

====================================================
2026-07-06 (Session 56 — OpenClaw memory backup and continuity check)
====================================================

OPENCLAW UPDATE PREP
- Confirmed Rocky's saved workspace memory files were present before Matt updated OpenClaw.
- Verified that `MEMORY.md`, `memory/`, and core persona/config files live under `/home/aiadmin/.openclaw/workspace` and should survive a normal package update.
- Provided and validated a safe pre-update backup command using `tar` against the workspace memory/persona files.
- Confirmed Matt successfully created an on-server backup archive:
  - `/home/aiadmin/openclaw-memory-backup-20260707-000254.tgz`
- Noted the remaining caveat that a destructive update command, disk issue, or removal of `~/.openclaw` would still require restoring from backup.

SESSION CONTINUITY
- After the OpenClaw reset/new-session flow, Rocky came back online and responded normally in Matt's Discord channel.
- Confirmed runtime identity for the current session when asked: Codex on GPT-5.

CAPABILITY STATUS
- No new external platform connection was added today.
- No new production automation was added today.
- Rocky's operational continuity process was hardened with a verified memory-backup workflow before OpenClaw updates.

====================================================
2026-07-07 (Session 57 — AssetPanda laptop documentation audit)
====================================================

ASSETPANDA DOCUMENTATION AUDIT
- Ran a read-only AssetPanda audit for Matt to identify laptop assets with no documentation attached to the record.
- Queried the AssetPanda Assets entity and checked v3 Document attachments for each strict laptop asset.
- Found 70 laptop assets and 23 laptop assets with no Document attachments.
- Saved the final strict result set locally:
  - CSV: `/home/aiadmin/.openclaw/workspace/tmp/assetpanda_laptop_missing_docs/laptops_missing_documents_strict.csv`
  - JSON: `/home/aiadmin/.openclaw/workspace/tmp/assetpanda_laptop_missing_docs/laptops_missing_documents_strict.json`
- Excluded one broad-pass false positive, asset `114` (`KYY Portable Laptop 15.6" Laptop Monitor`), because it was an accessory rather than a laptop.
- No AssetPanda writes were performed.

CAPABILITY STATUS
- No new external platform connection was added today.
- No new production automation was added today.
- Existing AssetPanda read-only inventory and attachment-query capabilities were used to produce a targeted missing-documentation report.

====================================================
2026-07-08 (Session 58 — Ally runtime fix and LSWork report enhancements)
====================================================

ALLY DISCORD / CODEX RUNTIME FIX
- Repaired Ally's Discord channel connection after finding `openclaw-ally.service` was running from the effective profile rooted at `/home/aiadmin/.openclaw-ally/.openclaw-ally`.
- Installed the missing `@openclaw/discord` plugin into Ally's effective profile and restarted the aiadmin-owned service process so systemd brought it back cleanly.
- Verified Ally's Discord integration loaded and resolved the configured guild/channel:
  - Guild: SLS Adrian
  - Channel: ally-chatwithadrian
- Fixed Ally's model/runtime configuration after responses failed with `Unknown model: openai-codex/gpt-5.4`.
- Updated Ally's effective OpenClaw config to use the `openai` provider model shape with Codex runtime:
  - Default model: `openai/gpt-5.4`
  - Secondary model: `openai/gpt-5.5`
  - Runtime: `codex`
  - OAuth profile: `openai:astratos@slsct.org`
- Restarted Ally and verified logs showed the corrected model, Discord channel resolution, and successful bot probe.

LSWORK ACTIVITY REPORT ENHANCEMENTS
- Reran the LSWork activity summary for Jonathan Caez (`jcaez`) through 2026-07-02 end-of-day Eastern Time.
- Updated `scripts/lswork_activity_report.py` so the report can be reused for other employees with an explicit `--through YYYY-MM-DD` cutoff.
- Enhanced the generated report by:
  - Removing the privacy-note paragraph.
  - Renaming the visible case column to `Case ID`.
  - Making each Case ID a clickable LegalServer matter-profile link using the matter database ID.
  - Calculating report ages relative to the selected cutoff date.
- Generated a revised linked PDF and created a new Outlook draft in `mdugan@slsct.org` with the revised PDF attached.
- Created pending Skill Workshop proposal `lswork-activity-summary-20260708-33e4e4a749` to preserve the reusable procedure for future employee requests.
- No LegalServer writes were performed for this report work.

POWER BI FILE TOOLING RESEARCH
- Researched server-side options for Rocky to inspect or work with Power BI files on the Linux server.
- Identified `pbixray` as the best first read-only candidate for PBIX inspection and documentation.
- Identified DuckDB's `pbix` extension, `pbi-tools.core`, Microsoft's Power BI modeling MCP, and Tabular Editor CLI as possible later additions depending on sandboxing and workflow needs.
- Confirmed current server fit: Node and Python are installed; .NET and DuckDB are not installed.

CAPABILITY ADDED
- Ally's OpenClaw/Discord/Codex runtime can now receive and respond in Adrian's Discord channel again.
- Rocky can now rerun LSWork activity summaries for arbitrary LegalServer logins through an explicit historical cutoff date.
- LSWork activity PDFs now support clickable LegalServer case links and cleaner employee-facing formatting.
- Rocky has a documented path for adding read-only Power BI PBIX inspection capability, starting with local `pbixray` testing.

CAPABILITY STATUS
- No new external SLS-CT data platform connection was added today.
- Existing OpenClaw, Discord, Codex runtime, LegalServer report API, Microsoft Graph draft-email, and local reporting capabilities were repaired or extended.

====================================================
2026-07-10 (Session 59 — Zoom Contact Center MCP scaffold and live read-only test)
====================================================

ZOOM CONTACT CENTER MCP SCAFFOLD
- Created `zoom-contact-center-mcp/` as a separate Node/TypeScript stdio MCP project for Zoom Contact Center access.
- Kept local credentials outside the project in `tools/zoom-contact-center-mcp.env`; `.env.example` contains only variable names.
- Implemented the first read-only MCP tools:
  - `list_contact_center_queues`
  - `search_contact_center_engagements`
  - `get_contact_center_engagement`
- Enforced a read-only safety posture with `ZOOM_MCP_ACCESS_MODE`; no generic arbitrary Zoom API caller was added.
- Responses are compact summaries rather than full raw Zoom API dumps.

LIVE READ-ONLY TESTING
- Matt added the missing Zoom Contact Center scopes and asked Rocky to run the live test.
- Adjusted access-mode parsing so `readonly`, `read_only`, and `read-only` spellings are accepted, including quoted env-file values.
- Built and typechecked the project successfully.
- Verified the MCP over stdio with live Zoom credentials:
  - Listed all 3 MCP tools.
  - Queue listing returned 5 queues with another page available.
  - Engagement search returned 1 engagement summary with another page available.
  - Engagement detail lookup returned successfully.
- No queue names, phone numbers, engagement IDs, or client/caller details were posted to Discord.

CAPABILITY ADDED
- Rocky now has a working internal Zoom Contact Center MCP scaffold that can expose read-only ZCC queue and engagement lookup tools to MCP-capable clients.
- The MCP is built to grow from Matt's real operational workflows while preserving privacy-first defaults.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing Zoom Contact Center OAuth access was packaged into a safer, reusable read-only MCP interface.

====================================================
2026-07-13 (Session 60 — YULAA analysis and Zoom Contact Center MCP expansion)
====================================================

YULAA MONDAY.COM DATA ANALYSIS
- Matt provided four YULAA Monday.com Excel exports in SharePoint folder `Documents/YULAA Project/For Analysis`.
- Downloaded and analyzed the 2023, 2024, 2025, and 2026 exports read-only.
- Built a local aggregate workbook:
  - `/home/aiadmin/.openclaw/workspace/tmp/yulaa_monday_analysis/aggregate_tables.xlsx`
- Summarized key relationships between first-disposition outcomes and factors such as pre-disposition SLS advice, housing subsidy, nuisance/lease/crime-count issues, attorney appearance, and legal-services referral status.
- Identified 7 data-quality cases with negative `Length of Stay of Execution`.
- Created and uploaded `YULAA Negative Stay Length Data Quality Cases.xlsx` to the same SharePoint analysis folder with the affected rows and a summary sheet.

ZOOM CONTACT CENTER MCP EXPANSION
- Expanded the internal read-only Zoom Contact Center MCP in:
  - `/home/aiadmin/.openclaw/workspace/zoom-contact-center-mcp`
- Registered it with OpenClaw as managed MCP server `zoom-contact-center`.
- Added wrapper `run-openclaw.sh` so the MCP sources credentials from `tools/zoom-contact-center-mcp.env` without committing secrets.
- Verified OpenClaw MCP probing reports 11 Zoom Contact Center tools.
- Validated representative live read-only queries, including phone-number search, missed-call lookup, and hold-time lookup.
- Confirmed design boundaries:
  - OpenClaw-hosted only for future bot instances.
  - Read-only.
  - No recordings or transcripts.
  - Queue aliases come from Zoom queue names.
  - Missed, abandoned, declined, and system disconnect categories stay separate.
  - Relative-date handling uses `America/New_York`.
- Remaining Zoom scopes needed for future expansion:
  - `contact_center:read:dataset_queue_performance:admin` for queue wait/performance data.
  - `report:read:user_activities:admin` for login activity.

ALLY ZOOM MCP ACCESS
- Added the `zoom-contact-center` managed MCP server to Ally's effective OpenClaw profile:
  - `/home/aiadmin/.openclaw-ally/.openclaw-ally/openclaw.json`
- Updated Ally's durable `TOOLS.md` and `MEMORY.md` instructions so Zoom Contact Center questions should try the OpenClaw-managed MCP first.
- Preserved safety guidance for Ally: minimum-necessary summaries, no recordings/transcripts, and separated missed/abandoned/declined/system-disconnect categories.
- Verified Ally's profile probe reports `zoom-contact-center: 11 tools`.
- Restarted Ally's aiadmin-owned service process after interactive auth blocked direct `systemctl restart`; systemd restarted `openclaw-ally.service` successfully.
- Verified Discord resolution for Ally's channel `ally-chatwithadrian`.

CAPABILITY ADDED
- Rocky now has an expanded internal read-only Zoom Contact Center MCP registered as an OpenClaw-managed server.
- Ally can now use the same Zoom Contact Center MCP from her Discord/OpenClaw runtime.
- Rocky and Ally have durable instructions to prefer the MCP for Zoom Contact Center data questions while minimizing sensitive data exposure.
- Rocky can now produce aggregate YULAA Monday.com analysis from exported Excel workbooks and return targeted data-quality workbooks through the approved SharePoint folder.

CAPABILITY STATUS
- Existing SharePoint, Monday.com export analysis, Zoom Contact Center OAuth, OpenClaw managed MCP, Ally OpenClaw, and Discord runtime capabilities were used or extended.
- No LegalServer writes were performed for the YULAA analysis work.
- The new Zoom MCP surface remains read-only and does not expose recordings or transcripts.

====================================================
2026-07-14 (Session 61 — Zoom MCP question logging and gap tracking)
====================================================

ZOOM CONTACT CENTER MCP QUESTION LOGGING
- Added MCP-owned append-only question/outcome logging to `zoom-contact-center-mcp/src/index.ts`.
- Every Zoom Contact Center MCP tool call now writes a privacy-minimized JSONL entry to:
  - `logs/zoom-mcp-question-log.jsonl`
- Added an optional `question` argument to all Zoom MCP tools so agents can preserve the original natural-language Zoom question for later gap analysis.
- Log entries include tool name, sanitized arguments, outcome (`answered` or `gap`), result counts/status, missing-scope markers, errors, and duration.
- Phone numbers and engagement IDs are redacted or hashed in local log output.
- Added human-facing documentation at:
  - `logs/ZOOM_MCP_QUESTION_LOG.md`
  - `zoom-contact-center-mcp/README.md`
- Updated workspace guidance in `AGENTS.md` so Zoom Contact Center questions should try the shared MCP first and rely on the MCP log for unanswered-question improvement tracking.

VERIFICATION
- Rebuilt the Zoom Contact Center MCP with `npm run build`.
- Reloaded OpenClaw MCP runtimes with `openclaw mcp reload`.
- Verified direct MCP logging through `run-openclaw.sh`.
- Confirmed a verification entry was appended at `2026-07-14T14:02:39Z`.

CAPABILITY ADDED
- Rocky now has a durable, shared source of truth for Zoom Contact Center MCP question outcomes and capability gaps across agents that use the shared MCP.
- Future Zoom MCP improvement work can be based on accumulated local gap logs instead of relying only on chat history.

CAPABILITY STATUS
- No new external platform connection was added today.
- Existing OpenClaw managed-MCP, Zoom Contact Center, workspace instruction, and local logging capabilities were extended.

====================================================
2026-07-15 (Session 62 — Grace OpenClaw setup and governed platform access)
====================================================

GRACE OPENCLAW INSTANCE
- Brought Grace's OpenClaw/Discord runtime into working order and verified Discord connectivity in the Grace/Moses channel.
- Established and verified Rocky-to-Grace SSH administration for approved maintenance tasks.
- Added a hard rule to Grace's instructions that Grace configuration/settings must not be changed without Matt's express approval for the specific change.
- Added a Grace-to-Rocky approval handoff workflow:
  - Grace queues Matt-approval requests locally.
  - Rocky checks for new requests every 5 minutes.
  - New requests are forwarded to Matt's Rocky Discord channel for approval.
- Added plugin-approval forwarding from Grace to Rocky/Matt's channel for applicable OpenClaw approval prompts.
- Updated the Security and Data Risk Log with Grace's approval process, Zoom MCP-first posture, and change-control expectations.

GRACE ZOOM CONTACT CENTER MCP
- Added the shared read-only Zoom Contact Center MCP to Grace's OpenClaw configuration.
- Gave Grace a private MCP wrapper/env path and copied the MCP project into Grace's tool area so it does not depend on traversing Rocky's home directory.
- Updated Grace's instructions to use the Zoom MCP first for Zoom Contact Center questions, summarize minimum-necessary results, and rely on the shared MCP question log for gaps.
- Verified Grace's MCP probe reports 11 Zoom Contact Center tools.
- Added Grace guidance for ambiguous Zoom metrics:
  - Ask a concise clarifying question when scope could mean inbound vs all directions, handled calls vs all engagements, queue-specific vs engagement-wide values, or different timezone/date boundaries.
  - Ask Moses about relevant queue/program scope when requests may span Main Hotline, RTC Hotline, Housing-English, RTC-Advocate, or related housing grant/program paths.

GRACE MS365, SHAREPOINT, AND GRAPH
- Matt created the `grace@slsct.org` mailbox and delegated access to Moses' mailbox/calendar.
- Matt created a dedicated Microsoft Entra app for Grace with scoped mailbox access.
- Rocky configured Grace's local MS365 env/helper with locked-down permissions.
- Verified Grace's Graph access:
  - Grace mailbox/calendar probes returned HTTP 200.
  - Moses mailbox/calendar probes returned HTTP 200.
  - Negative scope check against Matt's mailbox returned HTTP 403, confirming mailbox access is constrained.
- Added SharePoint `Sites.Selected` access for Grace to the Moses/Grace shared SharePoint site only.
- Verified Grace can resolve the site/document library, create/read/delete a temporary test file, and receives HTTP 403 against an unrelated Rocky/Matt SharePoint site.

GRACE MONDAY.COM
- Added Grace's Monday.com token to Grace's local locked-down env file and verified the token resolves as the Grace user.
- Verified Grace can read board-level metadata for the YULAA Survey board.
- Found Grace's Monday account/token could not yet read YULAA board columns, groups, or item IDs through the API, returning `403 UserUnauthorizedException`.
- Documented that Monday board/API permissions remain the active loose end and that any token exposed in chat should be rotated after setup.

GRACE LEGALSERVER AND CT JUDICIAL ACCESS
- Installed Grace's read-only LegalServer env/helper with locked-down permissions.
- Documented known report access for Grace, including LSWork case activity, LSWork app-log, and GHLA rejected-transfer report access.
- Verified read-only LegalServer smoke checks and CT Judicial case-detail lookup.
- Added Grace instructions for minimum-necessary LegalServer/Judicial summaries and no LegalServer writes.
- Added guidance for temporary local SQLite request databases when a recurring or complex analysis warrants it.

CAPABILITY ADDED
- Grace now has governed, approved-change-only OpenClaw administration with Rocky/Matt approval handoff.
- Grace can use the shared read-only Zoom Contact Center MCP as the first option for Zoom data requests.
- Grace has scoped Microsoft Graph access for her mailbox, Moses' mailbox/calendar, and one shared SharePoint site.
- Grace has initial read-only Monday.com and LegalServer/Judicial helper capability, with Monday board-detail permissions still pending.
- Grace's Zoom-answering behavior now includes clarification rules for ambiguous metrics and housing/RTC queue scope.

CAPABILITY STATUS
- New scoped external platform access was added for Grace: Zoom Contact Center MCP, MS365 Graph, SharePoint, Monday.com, LegalServer, and CT Judicial lookup.
- Grace's configuration now has explicit Matt approval/change-control governance.
- No Grace secrets are documented here; local credentials remain in locked-down env files.

====================================================
2026-07-16 (Session 63 — Runtime model confirmation and post-restore health check)
====================================================

OPENCLAW / DISCORD RUNTIME CONFIRMATION
- Matt asked whether the active Matt Discord session was using GPT 5.4 or GPT 5.5.
- Confirmed from live OpenClaw status that the main Matt Discord channel session was running:
  - Model: `gpt-5.5`
  - Runtime: OpenAI Codex
  - Session key: `agent:main:discord:channel:1486851314800398357`
- Noted one configuration nuance: the short alias `gpt` still points to `openai/gpt-5.4`, while the active Matt Discord session and default path were on GPT 5.5.

POST-RESTORE SYSTEMS CHECK
- Matt reported the Rocky host had been restored from backup around 2026-07-15 9 PM ET and asked for a verification pass.
- Performed a read-only health check after the restore.
- Verified the host was up with no failed system/user units, healthy memory, and about 15 GB free disk space.
- Verified Rocky's OpenClaw gateway was running on loopback port 18789, Discord was connected, and the Zoom Contact Center MCP probe reported 11 tools.
- Verified Grace's OpenClaw gateway was running on loopback port 18791, Discord was connected, Zoom MCP probe reported 11 tools, MS365 mailbox/calendar checks returned HTTP 200, and LegalServer smoke checks returned HTTP 200.
- Verified OS cron and OpenClaw cron jobs were present and idle/OK as expected.
- Confirmed late Grace setup artifacts from after the backup time appeared to have survived the restore.
- Noted watch items: Ally OpenAI auth appeared expired, Rocky was back on GPT 5.5 after local GPT 5.6 model rejection, Grace remained on the newer OpenClaw/GPT 5.6 path, and the OCR/PB poller had several data-specific skipped/erroring queued items with no writes occurring.
- No write actions or outbound message tests were performed during the health check.

CAPABILITY STATUS
- No new external platform connection was added today.
- No new write-capable tool or data integration was added today.
- Existing OpenClaw, Discord, Codex runtime, Zoom MCP, Grace runtime, MS365, LegalServer, cron, and local host health were checked and documented.
