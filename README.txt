
====================================================
2026-03-31 (Session — RTC Reports Pipeline Complete + SLS Analysis)
====================================================

RTC HOTLINE REPORTS — FULLY AUTOMATED
- Weekly cron: every Monday 7 AM ET -> covers previous Mon-Sun
- Monthly cron: first Monday of month 7 AM ET -> covers previous calendar month
- Both pipelines: cap-handled ETL, 3 PDF charts, SharePoint upload, draft email in MDugan@slsct.org
- First automated runs: Monday April 6, 2026
- Historical backfill: 12 weeks (12/29/25-3/22/26) completed
- Charts: Call Flow (diverging bars), Eligible Zip Codes (pie), Non-Eligible by Town (bar)
- Cap handling: if any day = 10k records, auto re-pulls in 4-hour UTC windows

BUG FOUND — EXCEL MACRO (RTCReport-Macro.txt)
- Location: CONVERT TO EST section, line FilterTime.Formula = "=I1-TIME(5,0,0)"
- Bug: macro subtracts TIME(5,0,0) from portal timestamps already in ET -> double-shift
- Effect: filter applied to 1:50 PM-9:45 PM instead of intended 8:50 AM-4:45 PM
- All historical reports understated stages 2-5 by ~2x
- Fix: change '=I1-TIME(5,0,0)' to '=I1' in the formula
- Rocky's pipeline uses the corrected filter

SHAREPOINT — ORGANIZED
- Folders: RTC Hotline Reports/Weekly, RTC Hotline Reports/Monthly,
  RTC Historical Reports, SLS Main Hotline Reports, RTC Reference Files
- Report Menu document created: Rocky - Report Menu.txt (root of MattRocky site)
- Security-Privacy-Decisions.txt updated with Decisions #1-5
- All temp data (/tmp/rtc/) deleted after every pipeline run

SLS MAIN HOTLINE — LEGAL ISSUE SELECTION REPORT
- SLS-1 report built: first-tier legal issue pie chart (Housing, Family, Consumer, etc.)
- March 2026 chart uploaded: SLS Main Hotline Reports/
- 16-month chart (Dec 2024-Mar 2026) in progress as background job

TOWNINDEX GAPS FOUND
- 67 zip codes in TownIndex have no routing_type in col I (Hartford, Bridgeport, Stamford, etc.)
- Macro mis-classifies these as Non-CT -> inflated 577 non-CT vs Rocky's correct 50
- Fix: pull zip codes from RTC flow JSON and update TownIndex (pending Matt flow review)
- West Haven (06516) and Hamden (06514) added as patches

HARD BOUNDARIES UPDATED
- LegalServer LIVE: confirmed NOT yet connected (DEMO only)
- Email: draft-only permanently, never auto-send to distribution lists
- Temp data cleanup: /tmp/rtc/ deleted after every pipeline run

PENDING / TODO
- Matt to review RTC Hotline flow JSON before TownIndex update
- TownIndex: add 06516 (West Haven) and 06514 (Hamden) to source file in SharePoint
- LegalServer LIVE connection: pending demo validation
- Staff onboarding checklist: to build
- Staff offboarding checklist: to build
- Re-run historical RTC reports with corrected time filter after TownIndex fix
