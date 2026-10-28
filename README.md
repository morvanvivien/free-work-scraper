# How to Scrape Free-Work: French IT Jobs and Freelance Missions (Daily Rates, Skills, Remote Mode)

> **Run it now, no code: [free-work-jobs-scraper on the Apify Store](https://apify.com/bibian/free-work-jobs-scraper)** &nbsp;|&nbsp; $2.00 per 1,000 job postings &nbsp;|&nbsp; no browser, no proxy

Free-Work is the main French board for IT jobs and freelance missions (around 10,000 live postings). It has no public export and no official API for third parties. This guide shows how to get the postings as clean structured data: freelance daily rates, permanent salaries, skills, remote mode and location, with a free Apify Actor, from the console or from code.

Searches this page answers: free-work scraper, french freelance missions, french IT jobs scraper, freelance daily rate France.

## What you get from the Free-Work scraper

- Every IT job and freelance mission, newest first, with title, company, contract type, seniority and remote mode.
- Freelance daily rates (`minDailyRate`, `maxDailyRate`) and permanent salary ranges (`minAnnualSalary`, `maxAnnualSalary`) when the posting shows them.
- Skills as a list, plus location with coordinates, start date, duration, publication and expiry dates.
- Optional full description, candidate profile and company description as plain text.

Export the dataset as JSON, CSV or Excel, or read it through the Apify API. A real sample (five results, personal data removed, long texts truncated) is in [`sample-output.json`](sample-output.json).

## Output fields

| Field | What it is |
|---|---|
| `id`, `url` | Free-Work posting id and public link |
| `title`, `jobCategory` | Job title and Free-Work job family |
| `companyName` | Hiring company or IT consultancy as displayed |
| `contracts` | Array: `contractor` (freelance), `permanent`, `fixed-term`, `apprenticeship`, `internship` |
| `experienceLevel` | Seniority level (for example `intermediate`, `senior`) |
| `remoteMode` | `full`, `partial` or `none` |
| `locationLabel`, `city`, `postalCode`, `region`, `country`, `latitude`, `longitude` | Where the job is based |
| `minDailyRate`, `maxDailyRate` | Freelance daily rate range in EUR (null when not shown) |
| `minAnnualSalary`, `maxAnnualSalary` | Permanent salary range in EUR (null when not shown) |
| `durationValue`, `durationPeriod`, `renewable`, `startsAt` | Mission length, renewal and start date |
| `skills`, `softSkills` | Arrays of required skills |
| `publishedAt`, `updatedAt`, `expiresAt` | ISO timestamps |
| `description`, `candidateProfile`, `companyDescription` | Plain text, only when full descriptions are enabled |

## Use cases

- **Track French freelance daily rates.** Group `minDailyRate` and `maxDailyRate` by skill, city or seniority to see what a Python, data or cloud freelancer is offered in France.
- **Build a daily job alert.** Schedule a run every morning with "published within 1 day" and send only the new missions to Slack, e-mail or a Google Sheet.
- **Find the companies that are hiring.** List which firms and consultancies recruit for a given technology this week.
- **Feed a job board or a newsletter.** Use the structured fields and the canonical `url` to power a niche board for French tech freelancers.

## Input

| Field | What it does |
|---|---|
| `keywords` | Job title, skill or technology, for example `data engineer`. Empty means every posting. |
| `contracts` | Any of `contractor`, `permanent`, `fixed-term`, `apprenticeship`, `internship`. Empty means all. |
| `remoteMode` | `any`, `full`, `partial` or `none`. |
| `publishedWithinDays` | Keep only recent postings. Use `1` for a daily monitoring run. |
| `maxItems` | The run stops once this many postings are saved (default 100). |
| `includeDescriptions` | Adds description, candidate profile and company description. |

Example input:

```json
{
  "keywords": "data engineer",
  "contracts": [
    "contractor"
  ],
  "remoteMode": "any",
  "publishedWithinDays": 7,
  "maxItems": 100,
  "includeDescriptions": false
}
```

## Four ways to use it

### 1. Apify console, no code

1. Open [the Actor page](https://apify.com/bibian/free-work-jobs-scraper) and click **Try for free** (a free Apify account is enough).
2. Fill in the input form (keywords, filters and the maximum number of results) and click **Start**.
3. When the run finishes, open the **Output** tab, check the table, then **Export** as CSV, Excel or JSON.
4. For a daily alert, open **Schedules**, create a daily schedule for the Actor and, optionally, add an integration that sends new results to Slack, e-mail or Google Sheets.

### 2. Apify API with JavaScript

```bash
npm install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
node examples/run.js
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

const run = await client.actor('bibian/free-work-jobs-scraper').call({
    "keywords": "data engineer",
    "contracts": [
        "contractor"
    ],
    "remoteMode": "any",
    "publishedWithinDays": 7,
    "maxItems": 100,
    "includeDescriptions": false
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
for (const item of items) {
    console.log(`${item.title} | ${item.companyName} | ${item.locationLabel} | ${item.minDailyRate ?? '-'}-${item.maxDailyRate ?? '-'} EUR/day`);
}
```

The complete file is [`examples/run.js`](examples/run.js) (ES module, Node 18 or later).

### 3. Apify API with Python

```bash
pip install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
python examples/run.py
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

run = client.actor("bibian/free-work-jobs-scraper").call(run_input={
    "keywords": "data engineer",
    "contracts": ["contractor"],
    "remoteMode": "any",
    "publishedWithinDays": 7,
    "maxItems": 100,
    "includeDescriptions": False,
})

for item in client.dataset(run.default_dataset_id).iterate_items():
    print(f"{item['title']} | {item.get('companyName')} | {item.get('locationLabel')} | {item.get('minDailyRate')}-{item.get('maxDailyRate')} EUR/day")
```

The complete file is [`examples/run.py`](examples/run.py). Never put your token in the code: read it from the `APIFY_TOKEN` environment variable as above.

### 4. REST API on RapidAPI, one key for all your APIs

If you already use RapidAPI, the same data is available as a plain REST API with monthly plans and a free tier: [free-work-jobs on RapidAPI](https://rapidapi.com/VivienMORVAN/api/free-work-jobs), also listed on [Zyla API Hub](https://zylalabs.com/api-marketplace/company+data/free+work+jobs+api/14510). No Apify account needed, billing goes through your marketplace subscription.

```bash
curl -X POST "https://free-work-jobs.p.rapidapi.com/v2/acts/bibian~free-work-jobs-scraper/run-sync-get-dataset-items?clean=true" \
  -H "x-rapidapi-key: <YOUR_RAPIDAPI_KEY>" \
  -H "x-rapidapi-host: free-work-jobs.p.rapidapi.com" \
  -H "content-type: application/json" \
  -d '{"keywords": "data engineer", "publishedWithinDays": 7, "maxItems": 50}'
```

The response is the JSON array of postings, with the same fields as above.

## Pricing

Pay per result: **$2.00 per 1,000 job postings** (that is $0.002 per job posting), plus a tiny start fee of $0.00005 per run. You pay only for the results saved to the dataset. Apify's free plan includes monthly credits, enough to try it on a few hundred results. Prices are those shown on the [Store page](https://apify.com/bibian/free-work-jobs-scraper) when this guide was written; the Store page is the reference.

## Limits and honest notes

- Only what Free-Work publishes is collected. Many freelance missions do not show a daily rate, so `minDailyRate` and `maxDailyRate` are often `null` (this is visible in `sample-output.json`).
- The dataset contains job postings only. Recruiter contact details are never exported.
- Postings are the public ones: anything behind a login is out of scope.
- This is an unofficial tool, not affiliated with Free-Work. Check the site terms and the data protection rules that apply to your use.

## FAQ

**Does it need a browser or a proxy?**  
No. The Actor reads the data directly, so runs are fast and cheap.

**How do I only get new missions every day?**  
Set `publishedWithinDays` to `1` and schedule the Actor once a day. Results come newest first, so a date-limited run stops early.

**Can I filter freelance missions only?**  
Yes, set `contracts` to `["contractor"]`.

**Can I get the data in Excel or Google Sheets?**  
Yes. Export the dataset as CSV, Excel or JSON from the Apify console, or connect the Apify Google Sheets integration.

## Try it

**[Open free-work-jobs-scraper on the Apify Store](https://apify.com/bibian/free-work-jobs-scraper)**

The code in `examples/` is released under the MIT license (see [LICENSE](LICENSE)). The data belongs to its original publishers.
