# Track Workday jobs with n8n and Google Sheets

An inactive starter from DataSmithLab. Collect a bounded snapshot of public jobs from Workday career sites and keep a deduplicated log in your own Google Sheet.

[Workday Jobs Scraper on Apify](https://apify.com/datasmithlab/workday-jobs-scraper) · [Walkthrough](https://datasmith-lab.github.io/datasmithlab-workday-n8n/) · [SEO setup guide](https://datasmith-lab.github.io/datasmithlab-workday-n8n/scrape-workday-jobs-to-google-sheets-apify-n8n.html) · [Download starter ZIP](https://github.com/Datasmith-lab/datasmithlab-workday-n8n/releases/latest/download/workday-jobs-to-sheets-starter.zip)

## What it does

1. Reads existing records from a dedicated Jobs tab.
2. Starts one Workday Jobs Scraper run on Apify and waits for completion.
3. Checks HEALTH, COMPANIES and dataset counts before writing.
4. Upserts on a stable input-scope + job-ID key, preserving firstSeenAt and older records.

First records in an empty scope are `baseline`. Later unseen IDs are `logged`. First-seen time is not publication time. Limited snapshots can discover older jobs later. Jobs absent from a later snapshot are retained; this workflow does not detect closed jobs.

## Setup

Requires your own Apify account, n8n with HTTP Request, Code, Wait and Google Sheets nodes, and a Google account with spreadsheet edit access. No Slack or Telegram account is needed.

1. Create a **Jobs** tab in your spreadsheet and paste the single header row from `jobs-sheet-headers.csv` into A1. Preserve existing tabs. An existing Jobs table must use exactly these headers.
2. Import `workday-jobs-to-sheets.n8n.json`. It is inactive and contains no credentials.
3. In Configuration replace `REPLACE_WITH_SPREADSHEET_ID` with your own spreadsheet ID and set the board URL.
4. Create an n8n **Header Auth** credential: header name `Authorization`, value `Bearer YOUR_APIFY_TOKEN`. Select it on every Apify HTTP Request node. Store the token in the credential manager, never in a public workflow or Sheet.
5. Connect your own Google Sheets OAuth2 credential on Read saved jobs and Save jobs.
6. Run manually. Check the Actor reports and the Jobs tab, then run again to verify stable IDs are updated rather than duplicated.
7. Only after validation activate the daily **09:15 UTC** schedule. Self-hosted n8n must remain running. Run one execution at a time; concurrent writes are not supported.

The default is one NVIDIA board URL, 10 jobs total and per company, descriptions off, no date filter and `onlyNewSinceLastRun:false`. The Actor's internal tracking is disabled because this starter tracks observed IDs in Sheets. Use 1–5 distinct **full** `myworkdayjobs.com` board URLs (the workflow rejects company names and non-Workday addresses) and limits up to 100. Changing boards or filters creates a new scope/baseline. Raising limits does not guarantee complete coverage or make first-seen jobs newly published.

The configured Actor charge cap is $0.10 per run. Current Store prices are authoritative. This cap is not an end-to-end n8n/Google/cloud cost guarantee.

## Failures and data limits

Missing reports, failed board resolution, dataset-count mismatch, missing required job fields or duplicate source/saved IDs stop the workflow before writes. Inspect the failed execution and existing Actor run before launching another. A Google Sheets upsert can partially complete; rerun sequentially after fixing the cause. This is not a transactional delivery guarantee.

Only selected Workday boards are scanned. Limits, filters and source access determine coverage. Salary is parsed from the job description only when the posting includes a range, so many jobs have none. Missing salary and publication dates stay blank. No recruiter contacts, applications or personal applicant data are collected. Do not share your Sheet publicly merely to use the workflow.

## Validation

- **Real Actor output, logic replay:** on 12 October 2026 the workflow's Prepare delivery logic was replayed against real output of the Workday Jobs Scraper (NVIDIA board, 10 jobs, run Ch3YDJv36qQL67fop). The first pass produced 10 baseline rows. A second pass with the saved rows produced 0 new IDs and 10 unique keys.
- **Input check:** the full-URL validation accepts `https://nvidia.wd5.myworkdayjobs.com/NVIDIAExternalCareerSite` and rejects `Stripe`, a non-Workday careers URL and an `http://` address.
- **Import:** the inactive workflow imported into a clean n8n 2.26.4 instance.
- **Live Google Sheets run:** on 12 October 2026 the complete workflow ran on a self-hosted n8n 2.26.4 against a real Google Sheet and the live Actor (NVIDIA board, 10 jobs). The first run saved 10 baseline rows (run `ma0cR4cstehD5w4QF`). The second run found no new IDs (`aEmpxJcBk1fjCs9Wb`). A read-back showed 10 unique keys and no duplicates.
- **Not tested:** a scheduled (timer-triggered) run of this exact Workday workflow. It reuses the structure of the ATS starter, whose schedule was verified on 11 October 2026 (Stripe, 10 unique saved keys, no duplicates). Run it manually first and inspect the Jobs tab.

Other n8n versions and cloud instances require your own setup check. This package has not been accepted into the official n8n template directory.
