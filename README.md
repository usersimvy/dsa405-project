# Wake County Food Safety Inspections: DSA 405 Project

**Author:** svyas2 · **Course:** DSA 405 (Fall 2026)

## 1. Project Overview
This project performs an end-to-end data audit, cleaning pipeline, and accounting reconciliation on commercial food safety inspection data from Wake County, North Carolina.

**Milestone 3 (P3: Acquisition & Integration)** adds a second source scraped from the county's public health inspection portal and joins it to the inspection data, with a verification suite that is shown catching deliberately broken data.

**Question:** How do Wake County restaurant inspection scores (open-data CSV) relate to the facility records on the county's public health inspection portal?

## 2. Data Origin & Provenance

| # | Source | Access | Terms |
|---|---|---|---|
| 1 | Wake County Environmental Services Department, Food Inspections CSV (`Food_Inspections.csv`). Download link: `https://data-wake.opendata.arcgis.com/api/download/v1/items/ebe3ae7f76954fad81411612d7c4fb17/csv?layers=1` | Single cached download by code, saved to `data/raw/Food_Inspections.csv` | [PASTE the license text from the dataset page here, or write "No license is stated on the dataset page."] |
| 2 | Wake County Health Dept public inspection portal run by CDP Inc. (`https://public.cdpehs.com/NCENVPBL/ESTABLISHMENT/ShowESTABLISHMENTTablePage.aspx?ESTTST_CTY=92`) | `requests` + BeautifulSoup + `pandas.read_html`, paginated ASP.NET postback | Public records; no robots.txt is published (HTTP 404) |

## 3. Collection behaviour (P3)
- One request every 2 seconds (`MIN_DELAY_SEC`); honest User-Agent: `DSA405-Student-Project/1.0 (coursework, NC State University; svyas2@ncsu.edu)`.
- Every response is cached in `data/raw/`; re-runs read from disk and make no network requests.
- Timeouts and 429/5xx responses are retried 3 times; other non-200 responses, empty bodies, selectors that match nothing, missing columns and stuck pagination raise `AcquisitionError`, and two of these are demonstrated in the notebook.
- Scope: the first `MAX_PAGES` (40) pages of the portal. The drift from the A8 plan is documented in the notebook, Section 2.
- Person-identifier columns are dropped after loading: `INSPECTOR` and the free-text `DESCRIPTION` in Source 1, and `Inspector ID` on the portal. The raw cached files in `data/raw/` still contain them.

## 4. Join and verification
Source 1 (one row per inspection) is left-joined to the scraped facilities (one row per facility) on the facility ID, with `validate="m:1"`. The expected row count is computed before the merge. The verification suite has 7 checks (row count, key uniqueness, cardinality, value range, date range, referential integrity, no nulls) and is shown catching 7 deliberately broken copies of the data.

## 5. Known limitations
- The scraped facilities are the ones inspected most recently (the newest portal pages), not a random sample.
- Source 1 ends in September 2026, so the newest portal inspections are not in it yet.
- The portal is live, so a fresh scrape returns the newest 40 pages at that moment and the in-scope row count changes between
