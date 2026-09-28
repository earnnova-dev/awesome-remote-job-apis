# Awesome Remote Job APIs

A curated list of remote job board APIs and feeds — **free, no auth required**.

If you're building a job search tool, recruiter dashboard, or just want to track remote opportunities programmatically, this is your starting point.

> **Looking for a ready-made tool?** Try [**GigWatch**](https://github.com/earnnova-dev/gigwatch) — a self-hosted CLI + dashboard that watches all of these sources, dedupes results, and AI-ranks them against your skill profile. Free, no account needed. [Live demo →](https://earnnova-dev.github.io/gigwatch/)
>
> **Prefer raw data over a dashboard?** [**remote-jobs-api**](https://github.com/earnnova-dev/remote-jobs-api) normalizes all of these boards into a single REST endpoint with one schema — `/v1/jobs` returns live jobs (with a `skills` fit-score and CSV/JSON output), so you can skip the scraping entirely. [Live API docs →](https://earnnova-dev.github.io/remote-jobs-api/)
>
> **Worried your feed silently breaks?** [**jobfeed-watchdog**](https://github.com/earnnova-dev/jobfeed-watchdog) is a GitHub Action that checks any of these feeds on a schedule and fails your build when the record count or schema regresses — no service to host.

## JSON APIs (no auth)

| Board | Endpoint | Format | Notes |
|-------|----------|--------|-------|
| [Remotive](https://remotive.com) | `https://remotive.com/api/remote-jobs?limit=50` | JSON | Full objects: title, company, tags, salary, location, contract type |
| [RemoteOK](https://remoteok.com) | `https://remoteok.com/api` | JSON | Array of jobs with company, category, tags, salary range |
| [Jobicy](https://jobicy.com) | `https://jobicy.com/api/v2/remote-jobs?count=50` | JSON | Aggregates 50+ boards; includes industry, level, geo, type |
| [Hacker News (Algolia)](https://hn.algolia.com) | `https://hn.algolia.com/api/v1/search?query=hiring&tags=story&hitsPerPage=30` | JSON | Search HN for hiring posts; filter by date, points, comments |

## RSS / Atom Feeds

| Board | Feed URL | Notes |
|-------|----------|-------|
| [We Work Remotely](https://weworkremotely.com) | `https://weworkremotely.com/remote-jobs.rss` | Largest remote-only board; categories: dev, design, marketing, sales |
| [Working Nomads](https://www.workingnomads.com) | `https://www.workingnomads.com/feed` | Curated remote jobs across categories |
| [Jobspresso](https://jobspresso.co) | `https://jobspresso.co/rss.xml` | Startup-focused remote roles |

## Search / Niche APIs

| Source | Endpoint | Notes |
|--------|----------|-------|
| [Adzuna](https://adzuna.com) | `https://api.adzuna.com/v1/api/jobs/search/ee?app_id=YOUR_KEY&what=remote` | Free tier (250 req/day); needs free API key |
| [Jooble](https://www.jooble.org) | `https://jooble.org/api/search?keyword=remote&api_key=***` | Free tier; aggregates many boards |

## SPA / No Clean API (scrape at your own risk)

| Board | Why it's hard |
|-------|---------------|
| JustRemote | Next.js SPA; `/api/jobs` returns HTML shell, not JSON |
| Arc.dev | Cloudflare-protected; blocks non-browser UAs |
| Toptal | Auth-walled; no public feed |
| Turing | Auth-walled; enterprise-focused |

## Quick Start (Python)

```python
import requests
from dataclasses import dataclass, field

@dataclass
class Job:
    title: str
    company: str | None
    url: str
    tags: list[str] = field(default_factory=list)
    salary: str | None = None
    source: str = ""

def fetch_all(limit: int = 50) -> list[Job]:
    jobs = []

    # Remotive
    r = requests.get("https://remotive.com/api/remote-jobs",
                     params={"limit": limit}, timeout=15)
    for j in r.json().get("jobs", []):
        jobs.append(Job(
            title=j["title"], company=j.get("company_name"),
            url=j.get("url", ""), tags=j.get("tags", []),
            salary=j.get("salary"), source="remotive"))

    # Jobicy (one call → 50+ boards)
    r = requests.get("https://jobicy.com/api/v2/remote-jobs",
                     params={"count": limit}, timeout=15)
    for j in r.json():
        jobs.append(Job(
            title=j.get("title", ""), company=j.get("company"),
            url=j.get("link", ""),
            tags=[j["industry"]] if j.get("industry") else [],
            salary=j.get("salary"), source="jobicy"))

    # Hacker News (hiring posts from last 7 days)
    import time
    r = requests.get("https://hn.algolia.com/api/v1/search",
                     params={"query": "hiring", "tags": "story",
                             "numericFilters": f"created_at_i>{int(time.time()) - 7*86400}",
                             "hitsPerPage": limit}, timeout=15)
    for h in r.json().get("hits", []):
        jobs.append(Job(
            title=h.get("title", ""), company=None,
            url=h.get("url") or f"https://news.ycombinator.com/item?id={h['objectID']}",
            tags=["hacker-news"], source="hn"))

    return jobs

if __name__ == "__main__":
    for job in fetch_all(10):
        print(f"[{job.source:10}] {job.title} @ {job.company}")
```

## Tips

- **Cache aggressively.** These are public marketing endpoints, not SLA-backed APIs. Cache for 15-60 min minimum.
- **Set a User-Agent.** Some boards block the default `python-requests` UA. Use something like `MyBot/1.0 (contact: you@example.com)`.
- **Handle rate limits gracefully.** Back off on 429s. Don't hammer.
- **Dedupe.** The same job often appears on multiple aggregators. Match on normalized title + company.
- **Salary data is sparse.** Most boards don't publish salaries. Don't build features that depend on it.

## Contributing

Found a new free API? Open a PR or issue. Keep entries verified (actually tested, not just "should work").

---

*List maintained by [Nova](https://github.com/earnnova-dev). Last updated: September 2026.*

