<p align="center"><a href="https://apify.com/DevilScrapes/lever-jobs-scraper"><img src="assets/banner.svg" alt="Lever Jobs Scraper — Python example" width="100%"></a></p>

# Lever Jobs Scraper in Python: jobs.lever.co postings to a DataFrame, JSON and CSV

<p align="center">
<a href="https://apify.com/DevilScrapes/lever-jobs-scraper"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="Python" src="https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/lever-jobs-scraper-python/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/lever-jobs-scraper-python/actions/workflows/ci.yml/badge.svg"></a>
</p>

Here you will find a small Python wrapper, with tests, around the **[Lever Jobs Scraper](https://apify.com/DevilScrapes/lever-jobs-scraper)** that Devil Scrapes runs on Apify. Many companies host their careers page on Lever, and every such board lives at an address like `jobs.lever.co/plaid`. Pass the last part of that address (or the whole link) for as many employers as you like, and the Actor returns their current openings as rows: title, team, department, location, workplace type, commitment, posting date, plain-text description and the requirement sections. Apify runs the collection in the cloud; your script just saves `results.json` and `results.csv` for pandas, a spreadsheet or a scheduled report.

> **Actor:** [Lever Jobs Scraper](https://apify.com/DevilScrapes/lever-jobs-scraper) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in JavaScript:** [lever-jobs-scraper-javascript](https://github.com/devilscrapes/lever-jobs-scraper-javascript)

## 🔥 What you get

- **Path-based board names understood**: Lever puts the employer in the URL path, as in `jobs.lever.co/plaid`, and the Actor accepts that bare token, the board link, a single posting link or the `api.lever.co` feed address.
- **Whole board in one request per employer**: Lever returns every open posting at once, so there are no pages to walk and `maxItems` simply trims the list afterwards.
- **Team and department kept as two columns**: Lever tracks both, and neither is used to fill the other when one is missing.
- **Optional exact-match filters** on team, department, location and commitment are sent to Lever with the request and apply to every employer in the run.
- **Structured requirement blocks**: each posting's `lists` array keeps Lever's section headings with the bullet text cleaned of HTML, beside a plain-text description.
- **Unknown tokens are skipped, not fatal**: a token Lever does not recognise is counted in the final status message while the remaining employers finish.
- **Short-lived errors are retried** with increasing delays before an employer is marked as failed.

## 🎯 Use cases

### Measure where a company is hiring

Cross-tabulate `department` against `workplace_type` for a list of employers. One table shows whether a target company is growing engineering or sales, and how much of that growth is remote, hybrid or on-site.

### Track posting velocity over time

Because `created_at` is an exact timestamp, you can count postings opened in the last 7 or 30 days per company. Run the script weekly from cron and the counts become a time series of hiring momentum.

### Mine requirement text for skills

Explode the `lists` column, keep blocks whose heading mentions requirements or qualifications, and count keywords such as Python, Kubernetes or SQL. Labour-market researchers get a skills-demand sample from real postings rather than a survey.

### Build a sourcing sheet for recruiters

Filter to one `team` or `location`, sort by newest, and export title plus `hosted_url` to CSV. A recruiter can work through the list without opening each career site.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/lever-jobs-scraper-python.git
cd lever-jobs-scraper-python
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
export APIFY_TOKEN=your_token_here
python main.py                      # uses input.json
python main.py my-input.json --out data --max-charge 2
```

The script starts the [Lever Jobs Scraper](https://apify.com/DevilScrapes/lever-jobs-scraper), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

Leave out the command-line handling and the integration fits in a few lines. `ApifyClient.actor().call()` launches the run and returns once it has stopped; `iterate_items()` then yields one posting dict after another.

```python
from apify_client import ApifyClient

client = ApifyClient("YOUR_APIFY_TOKEN")
run = client.actor("DevilScrapes/lever-jobs-scraper").call(run_input={'companies': ['leverdemo', 'plaid'], 'maxItems': 5, 'proxyConfiguration': {'useApifyProxy': True}})
for item in client.dataset(run.default_dataset_id).iterate_items():
    print(item)
```

## 📥 Input

`main.py` sends `input.json` to the Actor unchanged. The sample reads two boards, Lever's own `leverdemo` board and `plaid`, and keeps at most 5 postings from each, so the trial run stays at 10 rows or fewer. To narrow a large employer before rows are charged, add `department`, `team`, `location` or `commitment`; Lever matches these values exactly, including capitalisation.

```json
{
  "companies": [
    "leverdemo",
    "plaid"
  ],
  "maxItems": 5,
  "proxyConfiguration": {
    "useApifyProxy": true
  }
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `companies` | array | yes | `["leverdemo", "plaid"]` | — |
| `team` | string | no | — | — |
| `department` | string | no | — | — |
| `location` | string | no | — | — |
| `commitment` | string | no | — | — |
| `maxItems` | integer | no | `200` | — |
| `proxyConfiguration` | object | no | `{"useApifyProxy": true}` | — |

## 📤 Output

Each row is one open posting. `company` holds the token you supplied, not a display name, which makes it a reliable grouping key across runs. `lists` is an array of `{text, content}` objects: `text` is a section heading and `content` is that section's cleaned bullet text. `created_at` and `scraped_at` are ISO 8601 timestamps in UTC.

| Field | Type | Always present |
|---|---|---|
| `posting_id` | string | yes |
| `company` | string | yes |
| `title` | string | yes |
| `team` | string or null | no |
| `department` | string or null | no |
| `location` | string or null | no |
| `workplace_type` | string or null | no |
| `commitment` | string or null | no |
| `hosted_url` | string | yes |
| `apply_url` | string | yes |
| `created_at` | string | yes |
| `description` | string | yes |
| `lists` | array | yes |
| `scraped_at` | string | yes |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.2 |
| Result emitted | $0.0015 |

Prices as configured on 2026-08-20. The [Actor page](https://apify.com/DevilScrapes/lever-jobs-scraper) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in Python

The pandas snippet below (`pip install pandas`) prints a company-by-department table split by workplace type and saves the postings opened in the last 30 days to `opened_last_30_days.csv`. Missing departments and workplace types are labelled rather than dropped, so totals still add up.

```python
import json

import pandas as pd

jobs = pd.DataFrame(json.load(open("results.json", encoding="utf-8")))
jobs["created_at"] = pd.to_datetime(jobs["created_at"], utc=True)
jobs["age_days"] = (pd.Timestamp.now(tz="UTC") - jobs["created_at"]).dt.days
jobs["department"] = jobs["department"].fillna("(none)")
jobs["workplace_type"] = jobs["workplace_type"].fillna("unspecified")

mix = pd.crosstab([jobs["company"], jobs["department"]], jobs["workplace_type"], margins=True)
print(mix.to_string())

fresh = jobs[jobs["age_days"] <= 30].sort_values("created_at", ascending=False)
columns = ["company", "title", "team", "location", "commitment", "hosted_url"]
fresh[columns].to_csv("opened_last_30_days.csv", index=False)
print(f"{len(fresh)} of {len(jobs)} postings were opened in the last 30 days")
```

## ❓ FAQ

### Where do I find a company's Lever token?

Open the company's careers page and follow any job link. If it lands on `jobs.lever.co/<something>`, that something is the token. Pasting the full link into `companies` works too; the Actor extracts the token for you.

### Does a big employer cost more to fetch?

Fetching is one request either way. You pay per posting written to the dataset, so `maxItems` and the four filters are what control cost on a board with hundreds of openings.

### What if a token is wrong or a board is empty?

A token Lever does not know is skipped and reported in the run's status message. A real board with no openings is a valid answer with zero rows. In both cases the other employers still produce data.

### Why is `location` a single value?

The Actor stores Lever's primary location for each posting. Some postings list extra locations on the careers page; those are not expanded into extra rows.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/lever-jobs-scraper) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [Lever Jobs Scraper on Apify Store](https://apify.com/DevilScrapes/lever-jobs-scraper)
- [JavaScript version of this example](https://github.com/devilscrapes/lever-jobs-scraper-javascript)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [Ashby Jobs Scraper](https://apify.com/DevilScrapes/ashby-jobs-scraper) — postings from employers whose careers pages run on Ashby.
- [Workable Jobs Scraper](https://apify.com/DevilScrapes/workable-jobs-scraper) — openings from Workable-hosted company boards.
- [SmartRecruiters Jobs Scraper](https://apify.com/DevilScrapes/smartrecruiters-jobs-scraper) — jobs from SmartRecruiters career sites, filterable by city or country.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/lever-jobs-scraper) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with Lever.
