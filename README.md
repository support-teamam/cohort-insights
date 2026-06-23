# cohort-insights

Public, reproducible "people like you" finance reference data for Team AM —
decoupled from any app release. A pipeline turns **public-domain US federal
data** into plain JSON that [Ask Monie](https://teamam.org/apps/ask-monie) and
[teamam.org](https://teamam.org) consume read-only.

> [!WARNING]
> **Reference only — not financial advice.** Every figure here is derived
> mechanically from public US federal datasets. It is **not reviewed by
> financial professionals**, is aggregated/estimated, and **can be outdated**
> (federal releases lag 6–18 months). Always verify against the cited source
> before relying on any number. Earnings are gross wages for an occupation in a
> metro — they are **not** your peers' actual savings, take-home pay, or budgets.

## What's here

`data/designer-earnings.json` — annual earnings for designer-family occupations
across major US metros: mean + the 10th/25th/median/75th/90th percentiles. Lets
an app answer "how do I compare?" by placing a user's income in the real
distribution for their role and city.

Each cohort record stores the exact **BLS series IDs** it came from, so any value
is one API call away from re-verification.

## Sources (all public domain — 17 U.S.C. §105, no license required)

| Source | Used for | Access |
|---|---|---|
| **BLS OEWS** (Occupational Employment & Wage Statistics) | earnings by occupation × metro × percentile | `api.bls.gov` (public API) |
| **FRED `PSAVERT`** (U.S. BEA personal saving rate) | a *national* saving-rate reference only | FRED CSV |

Not yet integrated (next): **BLS Consumer Expenditure Survey** (spending by
income/age) and **Fed Survey of Consumer Finances** (net worth by age) — these
are needed before any *cohort-specific savings* claim. Until then we only show a
clearly-labeled **national** saving rate, never "designers in NYC save X%."

## Methodology

1. Build BLS OEWS series IDs for each occupation (SOC) × area (CBSA) × wage
   percentile, e.g. NYC graphic-designer median = `OEUM003562000000027102413`.
2. Fetch the latest annual estimate via the BLS public API (batched, no key).
3. Assemble per-cohort JSON; drop cells BLS suppresses.
4. **Self-verify**: percentiles must be monotonic (p10 ≤ p25 ≤ median ≤ p75 ≤
   p90); the build fails loudly otherwise. Latest run: 0 failures across 28
   cohorts.

## Reproduce / verify

```bash
BUILD_DATE=$(date '+%Y-%m-%d') node scripts/build.mjs
```

Spot-check any number directly, e.g. NYC graphic designers, median:
`https://api.bls.gov/publicAPI/v1/timeseries/data/OEUM003562000000027102413`

## Refresh cadence

BLS OEWS publishes annually (≈ spring, labeled "May"). Re-run the pipeline on
each release and commit the new JSON — apps and the website pick it up with no
release of their own.
