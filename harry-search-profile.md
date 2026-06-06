# Harry's Job Search Profile — Source of Truth
Version: 1.0 | Created: 2026-06-04 | Owner: Hari Gadhavi
Purpose: Single source of truth for Coffee & Jobs automation keyword filtering and Claude scoring.
Deploy to VPS: /root/n8n-data/harry-search-profile.md

---

## 1. CANDIDATE BASICS

```
Name              : Hari Gadhavi
Location          : Berlin, Germany
Work authorisation: Valid (Fiktionswirkung §81 Abs. 4 AufenthG — full work rights)
Languages         : English C1 | German A2 (learning) | Hindi C2 | Gujarati C2
Hard exclude      : Deutsche Post / DHL (current employer — never apply)
```

---

## 2. TARGET ROLE TRACKS (priority order)

```
TRACK 1 — HR Operations
  People Operations Specialist / Coordinator / Associate
  HR Operations Specialist / Coordinator / Administrator
  HR Generalist (junior) | HR Associate | HR Administrator
  Onboarding Coordinator | Employee Experience Coordinator
  HRIS Coordinator | Workday Specialist | HR Systems Associate
  HR Business Partner (associate/junior)

TRACK 2 — Talent Acquisition
  Talent Acquisition Coordinator / Specialist
  Junior Recruiter | Recruitment Coordinator
  Sourcing Specialist | Talent Operations Specialist
  Recruitment Operations Associate

TRACK 3 — HR-Tech / AI-Adjacent
  HR Technology Analyst (junior) | People Analytics Associate
  Recruiting Automation Specialist | HR Tools Administrator
  HR Process Improvement Associate | HRIS Implementation Analyst

ENTRY LEVEL (always include — run in parallel with above)
  HR Intern | HR Werkstudent | HR Praktikum (paid)
  HR Trainee | HR Graduate Programme | TA Graduate Scheme
```

---

## 3. KEYWORD GROUPS FOR REVERSE ATS FILTER

### GROUP A — Job Title Signals
Must match ANY 1. If zero match → reject before Claude.

```
people operations | people ops | hr operations | hr ops |
talent acquisition | recruiting | recruiter | sourcing specialist |
hr generalist | hr associate | hr coordinator | hr administrator |
hr business partner | hrbp | onboarding coordinator |
employee experience | workforce operations | hr specialist |
people & culture | people and culture | recruitment coordinator |
talent coordinator | hr analyst | hris coordinator |
workday specialist | hr technology | hr tech | hr trainee |
personalreferent | hr werkstudent | hr praktikum | hr intern |
personalkoordinator | personalbetreuer | recruiter*in |
talent manager | hr sachbearbeiter
```

### GROUP B — Hard Skills Signals
Must match ANY 2. Fewer than 2 → reject before Claude.

```
workday | hris | ats | applicant tracking system |
onboarding | offboarding | employee lifecycle | hire to retire |
talent acquisition | full-cycle | full cycle recruiting |
boolean search | linkedin recruiter | candidate sourcing |
payroll | timesheet | payroll coordination |
compliance | hr compliance | labor law | employment law |
contractor management | vendor management | vms |
employee relations | hr documentation | audit |
job description | structured interview | offer management |
performance management | hr reporting | hr analytics |
bamboohr | successfactors | sap hr | personio |
greenhouse | lever | smartrecruiters | jobdiva | conrep |
stakeholder management | hiring manager | talent pipeline
```

### GROUP C — AI / Automation Bonus
Match ANY 1 → adds to Claude score. Not mandatory.

```
automation | ai tools | artificial intelligence | hr tech |
people analytics | hris implementation | process improvement |
workflow automation | process optimization | recruiting automation |
hr data | reporting dashboard | hr systems | digital hr |
hr transformation | future of work | hr innovation
```

### GROUP D — Seniority Signals (INCLUDE)
These confirm entry-to-mid level. Presence → positive signal.

```
junior | entry level | entry-level | associate | coordinator |
specialist | generalist | analyst | intern | internship |
werkstudent | praktikum | trainee | graduate | early career |
0-2 years | 1-3 years | 2-4 years | up to 3 years | no experience required |
fresh graduate | recent graduate | career starter
```

### GROUP E — NEGATIVE KEYWORDS (INSTANT REJECT)
Match ANY 1 → reject immediately. Zero Claude cost.

```
LANGUAGE BARRIERS:
c1 german required | c2 german required | deutsch c1 pflicht |
fließende deutschkenntnisse | verhandlungssicheres deutsch |
muttersprachliche deutschkenntnisse | german fluency required |
german c1 | german c2 | deutschkenntnisse c1 | deutschkenntnisse c2

SENIORITY TOO HIGH:
7+ years | 8+ years | 9+ years | 10+ years | 10 years experience |
senior director | vp of people | vp of hr | vice president hr |
head of people | head of hr | chief people officer | cpo |
global head | director of hr | director of people | c-level hr

EMPLOYER EXCLUSIONS:
deutsche post | dhl | dhl group | dp dhl

WRONG FUNCTION:
sales director | business development manager |
warehouse manager | production manager | manufacturing lead |
vertrieb | lagerleiter | produktionsleiter

AGENCY EXCLUSION (opaque client):
on behalf of an undisclosed client | client confidential |
our client is looking | agency: unknown end client
```

---

## 4. LOCATION PREFERENCES

```
TIER 1 (primary):
  Berlin — all districts, all work arrangements

TIER 2 (apply with relocation note on CV):
  Munich | Frankfurt am Main | Hamburg |
  Stuttgart | Düsseldorf | Cologne | Leipzig

TIER 3 (fully remote Germany-based):
  Any city if contract is Germany-based and fully remote

HARD EXCLUDE:
  Outside Germany (unless remote + Germany contract)
  Roles requiring physical presence outside Germany
```

---

## 5. COMPANY PREFERENCES

```
TIER 1 — Best fit:
  HR-tech companies: Personio, Leapsome, Workmotion, Factorial HR,
                     HiBob, Kenjo, Zavvy, Recruitee, Greenhouse EMEA
  Series A-C Berlin startups (<500 employees, English-working culture)
  AI-first companies with People Ops function

TIER 2 — Good fit:
  International tech companies (Berlin offices): Zalando, HelloFresh,
  N26, Delivery Hero, About You, GetYourGuide, Contentful, SumUp, Wolt

TIER 3 — Open:
  Large corporates with mature HR: BMW, Siemens, SAP, Bosch,
  Mercedes-Benz HR, Allianz, Deutsche Bank, Commerzbank

SIGNALS THAT INCREASE SCORE:
  "English-speaking environment" | "international team" |
  "no German required" | "English is our working language" |
  Series A / B / C | "scale-up" | "fast-growing" | "HR-tech"
```

---

## 6. CLAUDE SCORING CRITERIA
Used by Haiku node after keyword filter passes.

```
BASE SCORE (100 points):
  40 pts — Responsibilities match Harry's actual experience
             (HR ops lifecycle, recruiting, Workday, automation)
  25 pts — Required skills match Harry's hard skills
             (Workday, ATS tools, onboarding, Boolean, payroll)
  15 pts — Location and work arrangement fit
             (Berlin first, hybrid/remote preferred)
  10 pts — Company type fit
             (international, English-primary, tech/startup)
  10 pts — Seniority fit
             (entry to mid, 0-4 years)

BONUS (additive, max total 100):
  +15 — Workday listed as required skill
  +12 — Role explicitly mentions AI, automation, HR-tech
  +10 — Other tools Harry used: VMS, JOB Diva, Conrep
  +8  — HR-tech or AI-first company
  +5  — Boolean search, LinkedIn Recruiter, sourcing required

DISPLAY THRESHOLDS:
  70-100 → Strong match. Include in digest. Apply immediately.
  50-69  → Good match. Include in digest. Worth reviewing.
  40-49  → Partial match. Include with gap note.
  <40    → Discard. Do not surface.
```

---

## 7. HARRY'S ACTUAL EXPERIENCE (for Claude context)

```
MAGNIT GLOBAL — People Operations Specialist (Jan 2023 – Jan 2024)
  HR operations for 500+ contractors across EMEA, US, UK
  Tools: Workday (admin), VMS Wand, Excel, ServiceNow ticketing
  Key wins: 20% onboarding reduction, 50+ hrs/month saved,
            24-hr SLA on 50+ weekly queries, audit-ready docs,
            full payroll coordination and timesheet validation,
            weekly orientation sessions for 6-7 contractors/cohort

APIDEL TECHNOLOGIES — Talent Acquisition Recruiter (Dec 2021 – Jan 2023)
  Full-cycle recruiting for 80+ technical and specialist roles
  Tools: JOB Diva, Conrep, LinkedIn Recruiter, Boolean search
  Markets: US and UK | Clients: Intel, Kimberly-Clark, Hubbell, MilliporeSigma
  Key wins: 25% qualified applicant increase, 150+ interviews,
            85% retention rate, Fortune 500 client delivery

EDUCATION:
  MBA, Human Resources Management — BSBI Berlin, Completed April 2026
  BA, Political Science — MS University Vadodara, 2021

AI/AUTOMATION SKILLS (genuine differentiator):
  Claude API, Claude Code, n8n, Zapier, Make, Apify,
  ChatGPT, Gemini, Prompt Engineering, Vibe Coding
  Built: Automated job pipeline, resume tailoring system, personal AI OS
```

---

## 8. SOFT FLAGS (include in digest but highlight)

```
Flag these — do not reject, but add warning note:
  → German B1/B2 preferred (not required) — "Language gap: may be asked"
  → Salary below €28,000 gross — "Salary: below target"
  → Company <10 employees LinkedIn — "Very small team: verify"
  → On-site only, no remote/hybrid — "No flexibility mentioned"
  → "Student status required" for internships — HARD BLOCK
  → "Student preferred" for internships — navigable, include with note
```

---

## DEPLOYMENT NOTE
This file lives locally at: C:\Users\harsh\n8n-automations\harry-search-profile.md
VPS path (for n8n access): /root/n8n-data/harry-search-profile.md
Sync command: scp this file to VPS before Phase 1 build.
n8n reads this via Read File node or inline as workflow context.
