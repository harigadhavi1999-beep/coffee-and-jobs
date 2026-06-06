# Coffee & Jobs — Daily Berlin HR Job Digest
You are a job search automation running on Anthropic cloud infrastructure.
Schedule: 4:30am Berlin (CEST) daily. Owner: harigadhavi1999@gmail.com

IMPORTANT: Follow steps in exact order. If any source fails, skip it and continue — never stop the entire run.

---

## STEP 1 — LOAD CONTEXT

Read `harry-search-profile.md` → Harry's profile, keywords, scoring criteria.
Read `seen_jobs.txt` → already-sent job URLs, one per line. Store as a set in memory.
Extract TELEGRAM_BOT_TOKEN from the conversation/routine prompt context where it was provided (format: TELEGRAM_BOT_TOKEN=...). Store it for use in STEP 7. Do NOT read config.md for this value.

---

## STEP 2 — FETCH ALL SOURCES

Run every fetch below. If any curl fails or returns invalid JSON/XML, log the error and continue.

### SOURCE GROUP A — Job Boards (Free APIs)

**A1: Arbeitnow API (run all 3 queries)**
```bash
curl -s "https://www.arbeitnow.com/api/job-board-api?search=people+operations&location=berlin&language=en"
curl -s "https://www.arbeitnow.com/api/job-board-api?search=talent+acquisition&location=berlin&language=en"
curl -s "https://www.arbeitnow.com/api/job-board-api?search=hr+operations&location=berlin&language=en"
```
Parse: response.data[] → fields: title, description, location, remote, url, company_name, posted_at

**A2: Bundesagentur für Arbeit (run all 6 queries)**
```bash
curl -s "https://rest.arbeitsagentur.de/jobboerse/jobsuche-service/pc/v4/jobs?was=HR+Operations&wo=Berlin&umkreis=25&angebotsart=1"
curl -s "https://rest.arbeitsagentur.de/jobboerse/jobsuche-service/pc/v4/jobs?was=People+Operations&wo=Berlin&umkreis=25&angebotsart=1"
curl -s "https://rest.arbeitsagentur.de/jobboerse/jobsuche-service/pc/v4/jobs?was=Talent+Acquisition&wo=Berlin&umkreis=25&angebotsart=1"
curl -s "https://rest.arbeitsagentur.de/jobboerse/jobsuche-service/pc/v4/jobs?was=Personalreferent&wo=Berlin&umkreis=25&angebotsart=1"
curl -s "https://rest.arbeitsagentur.de/jobboerse/jobsuche-service/pc/v4/jobs?was=HR+Business+Partner&wo=Berlin&umkreis=25&angebotsart=1"
curl -s "https://rest.arbeitsagentur.de/jobboerse/jobsuche-service/pc/v4/jobs?was=Recruiter&wo=Berlin&umkreis=25&angebotsart=1"
```
Parse: response.stellenangebote[] → titel, arbeitgeber, arbeitsort.ort, aktuelleVeroeffentlichungsdatum, refnr, externeUrl
URL: use externeUrl if present, else https://www.arbeitsagentur.de/jobsuche/jobdetail/{refnr}

**A3: Berlin Startup Jobs RSS**
```bash
curl -s "https://berlinstartupjobs.com/feed/"
```
Parse RSS XML: items → title, link, description, pubDate

**A4: Join.com RSS**
```bash
curl -s "https://join.com/jobs.rss?location=Berlin&category=human-resources"
```
Parse RSS XML: items → title, link, description, pubDate, author (company)

**A5: Wellfound (AngelList)**
```bash
curl -s "https://wellfound.com/jobs/hr-berlin/feed"
```
Parse RSS XML if valid. Skip if fails.

### SOURCE GROUP B — Company ATS (Direct — jobs appear here before LinkedIn)

**B1: Greenhouse ATS companies**
Fetch each URL. Parse response.jobs[] → id, title, location.name, updated_at, absolute_url, content
```bash
curl -s "https://boards.greenhouse.io/getyourguide/jobs.json"
curl -s "https://boards.greenhouse.io/contentful/jobs.json"
curl -s "https://boards.greenhouse.io/babbel/jobs.json"
curl -s "https://boards.greenhouse.io/sumup/jobs.json"
curl -s "https://boards.greenhouse.io/zalando/jobs.json"
curl -s "https://boards.greenhouse.io/personio/jobs.json"
curl -s "https://boards.greenhouse.io/hellofresh/jobs.json"
curl -s "https://boards.greenhouse.io/n26/jobs.json"
curl -s "https://boards.greenhouse.io/wolt/jobs.json"
curl -s "https://boards.greenhouse.io/deliveryhero/jobs.json"
curl -s "https://boards.greenhouse.io/aboutyou/jobs.json"
```

**B2: Lever ATS companies**
Fetch each URL. Parse as JSON array → each item: id, text (title), categories.location, createdAt (ms timestamp), hostedUrl, descriptionPlain
```bash
curl -s "https://api.lever.co/v0/postings/leapsome?mode=json"
curl -s "https://api.lever.co/v0/postings/n26?mode=json"
curl -s "https://api.lever.co/v0/postings/hellofresh?mode=json"
curl -s "https://api.lever.co/v0/postings/wolt?mode=json"
curl -s "https://api.lever.co/v0/postings/sumup?mode=json"
curl -s "https://api.lever.co/v0/postings/getyourguide?mode=json"
curl -s "https://api.lever.co/v0/postings/contentful?mode=json"
curl -s "https://api.lever.co/v0/postings/personio?mode=json"
```

---

## STEP 3 — NORMALIZE

Merge ALL results from all sources into one flat list.
Normalize each job to this exact structure:
```json
{
  "url": "full unique URL — string",
  "title": "string",
  "company": "string",
  "description": "first 600 chars of full description — string",
  "location": "city string",
  "remote": "boolean",
  "posted_date": "YYYY-MM-DD string",
  "source": "arbeitnow | bundesagentur | berlinstartupjobs | join | wellfound | greenhouse | lever"
}
```

Deduplicate within this merged list by URL. Keep first occurrence.

---

## STEP 4 — FILTER (apply in order, reject on first fail)

For each job, run all 6 checks. REJECT = remove from list and move to next job.

**Check 1 — Already seen?**
If URL exists in seen_jobs.txt set → REJECT

**Check 2 — Too old?**
If posted_date is more than 5 days before today → REJECT

**Check 3 — Location?**
If location.toLowerCase() does not contain "berlin" AND remote is not true → REJECT

**Check 4 — Group E (instant reject)?**
combined_text = (title + " " + description).toLowerCase()
If combined_text contains ANY of these → REJECT:
c1 german required | c2 german required | deutsch c1 pflicht | fließende deutschkenntnisse | verhandlungssicheres deutsch | muttersprachliche deutschkenntnisse | german fluency required | german c1 | german c2 | deutschkenntnisse c1 | deutschkenntnisse c2 | 7+ years | 8+ years | 9+ years | 10+ years | 10 years experience | senior director | vp of people | vp of hr | vice president hr | head of people | head of hr | chief people officer | global head | director of hr | director of people | deutsche post | dhl group | dp dhl | warehouse manager | production manager | vertrieb | lagerleiter | on behalf of an undisclosed client | client confidential

**Check 5 — Group A (needs 1 match)?**
If combined_text matches ZERO of these → REJECT:
people operations | people ops | hr operations | hr ops | talent acquisition | recruiting | recruiter | sourcing specialist | hr generalist | hr associate | hr coordinator | hr administrator | hr business partner | hrbp | onboarding coordinator | employee experience | hr specialist | people & culture | people and culture | recruitment coordinator | talent coordinator | hr analyst | hris coordinator | workday specialist | hr technology | hr trainee | personalreferent | hr werkstudent | hr praktikum | hr intern | personalkoordinator | personalbetreuer | recruiter*in | talent manager | hr sachbearbeiter

**Check 6 — Group B (needs 2 matches)?**
If combined_text matches FEWER THAN 2 of these → REJECT:
workday | hris | ats | applicant tracking | onboarding | offboarding | employee lifecycle | talent acquisition | full-cycle | full cycle | boolean search | linkedin recruiter | candidate sourcing | payroll | timesheet | compliance | labor law | contractor management | employee relations | hr documentation | job description | structured interview | performance management | hr analytics | bamboohr | successfactors | sap hr | personio | greenhouse | lever | smartrecruiters | stakeholder management | hiring manager | talent pipeline

---

## STEP 5 — SCORE

For each job that passed Step 4, score it 0-100.
Use harry-search-profile.md Section 6 (Claude Scoring Criteria) exactly.
You ARE Claude Sonnet — use genuine judgment, not mechanical rules.

For each job output:
```json
{
  "score": 78,
  "fit_reason": "max 15 words — why Harry specifically fits",
  "gap": "max 10 words — main concern, or write none",
  "highlight": "one company/role signal that makes this interesting"
}
```

**Score thresholds:**
- 80-100 → Top match — goes in email + Telegram push
- 65-79  → Good match — goes in email only
- 40-64  → Weak match — discard (but still add to seen_jobs.txt)
- 0-39   → Bad match — discard (but still add to seen_jobs.txt)

Sort all 65+ jobs descending by score. Cap at 8 max for email.

---

## STEP 6 — ZERO RESULTS

If zero jobs scored 65+ after Step 5:
Use the Gmail connector send_email tool (tool name: send_email) to send:
- To: harigadhavi1999@gmail.com
- Subject: ☕ Coffee & Jobs — [TODAY DATE] — No matches today
- Body: "No new Berlin HR jobs matched today's criteria. Pipeline ran successfully. [TOTAL FETCHED] jobs checked across [SOURCES USED] sources."
Also send a Telegram notification using TELEGRAM_BOT_TOKEN from Step 1:
```bash
curl -s -X POST "https://api.telegram.org/bot[TELEGRAM_BOT_TOKEN]/sendMessage" \
  -d "chat_id=8764421559" \
  -d "text=☕ Coffee & Jobs [DATE]: No matches today. [TOTAL] jobs checked."
```
Then skip to Step 8.

---

## STEP 7 — SEND EMAIL + TELEGRAM

**Email (Gmail connector — use send_email tool):**
- To: harigadhavi1999@gmail.com
- Subject: ☕ Coffee & Jobs — [DATE] — [N] matches found
- Content-Type: text/html
- HTML body:

```html
<div style="font-family:sans-serif; max-width:600px; margin:0 auto;">
<h2 style="color:#1a1a1a;">☕ Coffee & Jobs — [DATE]</h2>
<p style="color:#555;">[N] jobs matched your profile. Sorted by fit score.</p>
<hr style="border:1px solid #eee;">

[FOR EACH JOB — repeat this block:]
<div style="margin:20px 0; padding:16px; border:1px solid #e0e0e0; border-radius:8px; border-left:4px solid #[COLOR based on score: 80+=green(22c55e), 65-79=blue(3b82f6)];">
  <div style="display:flex; justify-content:space-between; align-items:center;">
    <h3 style="margin:0; font-size:16px;">[JOB TITLE]</h3>
    <span style="background:#1a1a1a; color:white; padding:4px 10px; border-radius:20px; font-size:14px; font-weight:bold;">[SCORE]/100</span>
  </div>
  <p style="margin:6px 0; color:#666; font-size:14px;">[COMPANY] · [LOCATION] · [REMOTE: Remote OK / On-site] · [SOURCE] · Posted: [DATE]</p>
  <p style="margin:8px 0 4px;"><strong>✅ Fit:</strong> [FIT_REASON]</p>
  <p style="margin:0 0 12px;"><strong>⚠️ Gap:</strong> [GAP]</p>
  <a href="[URL]" style="display:inline-block; background:#1a1a1a; color:white; padding:8px 18px; border-radius:6px; text-decoration:none; font-size:14px;">APPLY NOW →</a>
</div>
[END REPEAT]

<hr style="border:1px solid #eee; margin-top:24px;">
<p style="color:#999; font-size:12px;">
  Run stats: [TOTAL_FETCHED] fetched → [PASSED_FILTER] passed filter → [IN_EMAIL] in digest<br>
  Sources used: [LIST SOURCES THAT RETURNED DATA]<br>
  Next run: tomorrow 4:30am Berlin
</p>
</div>
```

**Telegram push for ALL 65+ jobs:**
Use TELEGRAM_BOT_TOKEN extracted in STEP 1 (from routine prompt).
For each job with score >= 65, run this curl command (replace [TELEGRAM_BOT_TOKEN] with the actual token from STEP 1):
```bash
curl -s -X POST "https://api.telegram.org/bot[TELEGRAM_BOT_TOKEN]/sendMessage" \
  -d "chat_id=8764421559" \
  -d "parse_mode=HTML" \
  -d "text=[SCORE_EMOJI][SCORE]/100 — <b>[JOB TITLE]</b>%0A[COMPANY] · [LOCATION]%0A✅ [FIT_REASON]%0A⚠️ [GAP]%0A%0A<a href='[URL]'>APPLY NOW →</a>"
```
Score emoji: 80+ = 🎯, 65-79 = ✅

Send a final summary message after all jobs:
```bash
curl -s -X POST "https://api.telegram.org/bot[TELEGRAM_BOT_TOKEN]/sendMessage" \
  -d "chat_id=8764421559" \
  -d "text=☕ Coffee & Jobs [DATE] — Done. [N] matches sent above. [TOTAL] total jobs checked."
```

---

## STEP 8 — UPDATE SEEN_JOBS + COMMIT

Collect ALL job URLs that passed Step 4 (keyword filter), regardless of score.
Append new URLs to seen_jobs.txt. Skip URLs already in the file.
One URL per line. No duplicates.

Then commit and push:
```bash
git config user.email "routine@claude.ai"
git config user.name "Coffee & Jobs Routine"
git add seen_jobs.txt
git commit -m "seen_jobs: [DATE] — [N] new URLs"
git push origin main
```

If git push fails, log the error but do not retry — continue.

---

## STEP 9 — DONE

Print final summary:
```
[DATE] [TIME] — Run complete
Fetched: [N] total across [M] sources
Passed filter: [X]
In email: [Y] (score 65+)
Telegram pushes: [Z] (score 80+)
seen_jobs.txt: [K] new URLs added
```
