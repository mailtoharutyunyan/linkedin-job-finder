# LinkedIn Job Finder

**[→ Open the tool](https://mailtoharutyunyan.github.io/linkedin-job-finder/)**

LinkedIn's job search UI won't let you search worldwide, and typing a country name into the location
box silently resolves to the wrong place. This is a single static page that builds the search URL
directly — with 48 verified country geoIds and the keyword patterns that actually surface remote
contractor roles.

No backend, no analytics, no cookies, no account. Nothing is scraped: the page only builds links,
which open in your own logged-in session.

---

## Why this exists

I was looking for remote backend work from a country most job boards treat as a rounding error, and
found that the filters lie to you in three specific, measurable ways. Each one costs you weeks if you
don't know about it.

### Trap 1 — a country name in the location box resolves to the wrong country

Pass `location=Armenia` as text and LinkedIn matches **Armenia, Quindío — a city in Colombia.**

Measured: **1,432 results, every one of them Colombia or Latin America.** The reasonable conclusion
is that no remote work exists for you, and you stop looking. Pass the correct numeric country id
instead and the same search returns **83 genuinely Armenia-remote roles.**

The same collision exists for **Georgia** — the country versus the US state. Almost certainly for
others too.

Fix: always pass `geoId=<number>`, never `location=<text>`. The tool ships 48 verified ids and a
console snippet for looking up any location LinkedIn knows about.

### Trap 2 — "Remote" means remote *within that country*

A worldwide + remote + contract search returns listings reading "Mexico (Remote)", "India (Remote)",
"United States (Remote)". Every one still expects local work authorization.

**Remote is not the same as globally hireable**, and no filter expresses the difference. You have to
read the posting. A line saying *"must be authorized to work in X"* or *"W2 only, no C2C"* is a hard
no however remote the role looks.

### Trap 3 — the Contract job-type filter deletes almost everything real

This one is counter-intuitive. Filtering job type = `Contract` against one country returned
**1 result.** The same search unfiltered returned **83.**

Companies hiring international contractors still tag the posting **Full-time** — the contractor
arrangement lives in the contract, not in LinkedIn's job-type field. Filtering on Contract throws
away the roles you're looking for.

Fix: leave job type on Any, and describe the arrangement in your keywords instead.

---

## The useful trick: search the payment mechanism

If you need work that crosses borders without a work permit, don't search job titles — search how
they intend to pay you. Companies put it right in the title:

```
"through Deel" OR "Employer of Record" OR "contractor agreement"
```

That surfaces postings literally titled *"Senior Software Engineer JAVA (Contractor through Deel)"*.

One caveat learned the hard way: these are usually region-scoped. US companies mostly use
contractor-via-platform arrangements for Latin America nearshore; European companies use them for
Eastern Europe and the Caucasus. So pair the keywords with a region, and read the location line.

Other patterns that work: `B2B`, `independent contractor`, `self-employed`, `invoice`,
`day rate`, `EMEA`, `work from anywhere`, `globally distributed`.

---

## What's in the tool

| | |
|---|---|
| **Search builder** | Keywords, location, recency, job type, workplace type, experience, Easy Apply, under-10-applicants, sort by newest. Live URL preview, copy, open. |
| **Contractor presets** | Six one-click searches for cross-border / no-permit arrangements. |
| **48 verified geoIds** | Filterable, copyable table. Plus a console snippet to look up any location. |
| **ATS direct search** | Builds Google queries across Lever, Greenhouse, Ashby, Workable, Teamtailor, Recruitee, SmartRecruiters — often finds roles before aggregators index them. |
| **Board launcher** | Ten free boards, keywords carried across where supported. |

---

## URL parameter reference

| Param | Meaning | Values |
|---|---|---|
| `keywords` | Search terms | supports `OR` and `"exact phrase"` |
| `geoId` | Location id | numeric — `92000000` is Worldwide |
| `f_TPR` | Posted within | `r3600` 1h · `r86400` 24h · `r604800` week · `r2592000` month |
| `f_WT` | Workplace type | `1` on-site · `2` remote · `3` hybrid (comma-separate to combine) |
| `f_JT` | Job type | `F` full-time · `P` part-time · `C` contract · `T` temporary · `I` internship |
| `f_E` | Experience level | `1` intern · `2` entry · `3` associate · `4` mid-senior · `5` director · `6` executive |
| `f_AL` | Easy Apply only | `true` |
| `f_EA` | Under 10 applicants | `true` |
| `sortBy` | Sort order | `DD` newest · `R` relevance |

Verified working August 2026. LinkedIn shipped an AI-driven job search around that time; despite
reports that it would stop honouring most filters, `geoId`, `f_WT`, `f_JT` and `f_E` were all still
applied correctly in testing. If that changes, open an issue.

---

## Looking up a geoId yourself

For any city, region or country not in the table, run this in your browser console while logged in
to LinkedIn (it has to be same-origin — CORS blocks it from anywhere else, which is why the ids are
baked in rather than fetched live):

```js
await (await fetch('https://www.linkedin.com/jobs-guest/api/typeaheadHits'
  + '?origin=jserp&typeaheadType=GEO&geoTypes=COUNTRY_OR_REGION,POPULATED_PLACE'
  + '&query=' + encodeURIComponent('YOUR LOCATION'))).json()
```

Match the `displayName` exactly before trusting the `id` — that's precisely how the Armenia/Colombia
collision bites.

---

## Running locally

It's one file with no dependencies or build step.

```bash
git clone https://github.com/mailtoharutyunyan/linkedin-job-finder.git
open linkedin-job-finder/index.html
```

---

## A note on scraping

This tool deliberately doesn't scrape LinkedIn. Automated scraping violates their Terms of Service
and can get your account restricted — a bad trade when you're job hunting. Everything here is link
construction; you stay logged into your own session and click through like any other user.

## Contributing

geoIds drift and LinkedIn changes its parameters. If you find a stale id, a new collision like the
Armenia one, or a filter that stopped working, please open an issue or PR.

## License

MIT — see [LICENSE](LICENSE).
