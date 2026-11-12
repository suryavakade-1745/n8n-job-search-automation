# Job Search Automation — n8n

Self-hosted n8n workflows that scrape job listings from **Naukri** and **Indeed**, remove duplicates, score every role against my profile with **Claude AI**, and log the results to **Google Sheets** — automatically, every 8 hours.

---

## Why I built it

Checking job boards by hand every day is slow and repetitive. These workflows do the searching, cleaning and first-pass ranking automatically, so I only review the roles that fit.

## How it works

Both workflows follow the same pipeline:

| # | Node | n8n node type | What it does |
|---|---|---|---|
| 1 | Every 8 Hours | Schedule Trigger | Starts a run three times a day |
| 2 | Config | Edit Fields (Set) | Holds the settings: Apify token, keywords, locations, max jobs and the candidate background used for scoring |
| 3 | Build Searches | Code | Turns the keyword list (× locations for Indeed) into one search request each |
| 4 | Scrape Naukri / Indeed (Apify) | HTTP Request | Runs an Apify scraper actor and returns the listings |
| 5 | Normalize | Code | Maps raw scraper output to one clean format and strips HTML from descriptions |
| 6 | Remove Duplicates | Code | Drops repeat listings within the run, keyed on apply URL (or title + company) |
| 7 | Rate Job (Claude) | HTTP Request | Sends each job + the candidate background to the Anthropic Messages API and gets back a score (0–100), a one-line verdict and up to 5 key skills |
| 8 | Merge Score | Code | Parses the AI response and joins it onto the job record |
| 9 | Save to Jobs Sheet | Google Sheets | Appends the scored jobs to the `Jobs` sheet |

### Output columns (Google Sheet)

`platform` · `title` · `company` · `location` · `applyUrl` · `salary` · `contactEmail` · `score` · `verdict` · `keywords` · `description` · `scoredAt`

## Repository contents

```
n8n-job-search-automation/
├── README.md
├── job-search-naukri.json    # Naukri workflow (n8n export, credentials removed)
└── job-search-indeed.json    # Indeed workflow (n8n export, credentials removed)
```

## Setup

1. Run n8n (self-hosted or n8n Cloud).
2. **Workflows → Import from File** → choose `job-search-naukri.json` or `job-search-indeed.json`.
3. In the **Config** node, fill in:
   - `apifyToken` – your Apify API token
   - `keywords` – comma-separated job titles to search
   - `locations` – comma-separated cities (Indeed only)
   - `maxJobs` / `maxItems` – listings per search
   - `experience` – a short description of your background, used by the AI to score fit
4. Create and attach credentials:
   - **Anthropic API** → *Rate Job (Claude)*
   - **Google Sheets** (service account) → *Save to Jobs Sheet*
5. In **Save to Jobs Sheet**, replace `YOUR_GOOGLE_SHEET_ID` with your sheet's ID and make sure it has a tab named `Jobs`.
6. Run once with **Execute workflow** to test, then activate it.

> No API keys, tokens, credential IDs or sheet IDs are included in this repository.

## Challenges solved

- **Apify memory limits** – capped each scraper run at 1 GB (`memory=1024`) and sent searches one at a time (batch size 1) so runs finish within the plan limits.
- **Anthropic API rate limiting** – sent jobs to Claude one at a time (batch size 1) so large batches are scored without hitting rate limits.
- **Messy AI output** – the Merge Score step strips stray markdown and falls back to a safe default if the response can't be parsed, so one bad reply never breaks the run.

## Roadmap

- [ ] Cross-run de-duplication: append-or-update rows keyed on `applyUrl`, so the same job isn't added twice across runs.
- [ ] Rule-based pre-filter (location, experience) before AI scoring to cut API cost.
- [ ] Switch scoring to a smaller, cheaper Claude model.

## Tools

n8n · Apify · Anthropic Claude API · Google Sheets · JavaScript (Code nodes)

---

**Surya V Vakade** — SAP FICO · Finance & Analytics
[LinkedIn](https://www.linkedin.com/in/surya-vv) · [GitHub](https://github.com/suryavakade-1745)
