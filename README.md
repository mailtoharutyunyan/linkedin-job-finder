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
Its own location box says so, while the page title still reads "Jobs in Armenia."

Measured: **1,385 results, every one of them Colombia or Latin America.** The reasonable conclusion
is that no remote work exists for you, and you stop looking. Pass the correct numeric country id
instead and the same search returns **83 genuinely Armenia-remote roles.**

I then tested 39 country names to see how widespread this is. **Not one of the 39 resolved to the
plain country.** Nine land on a different country, in several cases a different continent:

| You type | LinkedIn searches | |
|---|---|---|
| Armenia | Armenia, Quindío, **Colombia** | confirmed end to end |
| Georgia | Georgia, **United States** | confirmed end to end |
| Lebanon | Lebanon, Tennessee, **United States** | |
| Cuba | Cubatão, São Paulo, **Brazil** | |
| Malta | Malta, Illinois, **United States** | |
| Turkey | Turkey, North Carolina, **United States** | |
| Guinea | National Capital District, **Papua New Guinea** | |
| Liberia | Liberia, Guanacaste, **Costa Rica** | |
| Niger | Lagos State, **Nigeria** | |

The other thirty silently narrow you to one city or region instead of jumping countries — Poland
becomes Mazowieckie, Ukraine becomes Kyiv City, India becomes Maharashtra, Ireland becomes County
Dublin, Brazil becomes São Paulo. Less alarming, equally lossy: every role outside that region
vanishes and nothing tells you.

Two are verified end to end in the live job search. `Georgia` resolved to "Georgia, United States"
and returned 9,645 results with nothing in Tbilisi. The remaining seven are the resolver's first
choice rather than a click-through I checked individually — stated plainly rather than dressed up.

Fix: always pass `geoId=<number>`, never `location=<text>`. The tool ships 48 verified ids, flags
the colliding names in its own picker, and includes a console snippet for looking up anything else.

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

## What it does that LinkedIn doesn't

**Searches many countries in one go.** LinkedIn's UI accepts exactly one location. If you can legally
work in six countries, that's six separate searches you have to remember to re-run. The `f_PP`
parameter takes a comma-separated list and works fine — it just isn't exposed anywhere in the
interface. Pick as many as you like; one search covers them all.

**Uses ids, never names.** So you can't land in the wrong hemisphere.

**Hides jobs you can't take.** One field turns `clearance, W2` into
`NOT clearance NOT W2`, so the postings that would waste your time never show up.

**Remembers the search.** Everything is written to the page's own address. Bookmark it and your setup
comes back; send the link and someone else gets your setup, not a blank form.

**Skips the boards.** One button searches Lever, Greenhouse, Ashby, Workable, Teamtailor, Recruitee
and SmartRecruiters at once — jobs land there before aggregators index them.

### Deliberately kept simple

Two fields and one button. Everything else is behind a single **Refine** disclosure, closed by
default, with defaults already set to remote, past week, mid-senior, newest first.

There is **no Contract job-type filter**, on purpose — see finding 2 below. It cut results from 83 to
1 because companies tag international contractor roles as Full-time.

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
