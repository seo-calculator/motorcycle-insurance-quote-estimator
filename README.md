[README (3).md](https://github.com/user-attachments/files/27179032/README.3.md)
# motorcycle-insurance-quote-estimator# Motorcycle Insurance Cost Calculator
**Riders Share | Built by [RiZen Metrics](https://www.rizenmetrics.com/)**

A self-contained motorcycle insurance cost estimator designed to be iframed into a Prismic blog post. Outputs a monthly and annual rate range based on seven rider inputs. All logic is synchronous and client-side — no build tools, no dependencies, no API calls.

---

## What It Does

Takes seven inputs from the rider and returns:

- Estimated monthly and annual rate range
- Comparison to the national average ($500-$1,500/yr)
- Rate factors panel showing what's pushing the estimate up or down
- State-specific CTA linking to the Riders Share insurance quote page
- Secondary CTA linking to Riders Share state listings

Rate data is sourced from Progressive (published policy data, Apr 2024-Mar 2025) and the Insurance Information Institute. Estimates are validated against ValuePenguin (Jan 2026) and MoneyGeek (Dec 2025) benchmarks.

---

## File Structure

```
/
└── index.html    # Everything. All CSS and JS are inline.
```

That's it. One file.

---

## Inputs

| Field | Type | Options |
|---|---|---|
| State | Dropdown | All 50 states + DC |
| Bike type | Segmented | Cruiser / Sport / ADV / Touring / Standard |
| Engine size | Segmented | Under 500cc / 500-999cc / 1000cc+ |
| Rider age | Segmented | 18-24 / 25-34 / 35-49 / 50+ |
| Riding experience | Segmented | Under 2 yrs / 2-5 yrs / 5-10 yrs / 10+ yrs |
| Violations (last 3 yrs) | Toggle | Yes / No |
| Coverage level | Dropdown | Liability Only / Standard / Full Coverage |

All seven fields are required. Output does not render until the rider clicks "Estimate My Rate" with all fields filled.

---

## Calculation Logic

### Base monthly rate

| Coverage Level | Base |
|---|---|
| Liability Only | $20/mo |
| Standard | $47/mo |
| Full Coverage | $72/mo |

### Multipliers applied in order

**State (5 tiers):**

| Tier | Multiplier | States |
|---|---|---|
| 1 - Lowest | 0.72x | Iowa, North Dakota, South Dakota, Maine, Vermont, Montana, North Carolina, Wisconsin, Nebraska, Massachusetts, New Hampshire, Idaho, Wyoming |
| 2 - Below avg | 0.88x | Alaska, Indiana, Ohio, Minnesota, Oregon, Virginia, West Virginia, Colorado, Washington, Pennsylvania |
| 3 - Average | 1.0x | Arkansas, Kansas, Missouri, Connecticut, Maryland, Rhode Island, Illinois, Tennessee, Georgia, New Mexico, Nevada, Oklahoma |
| 4 - Above avg | 1.18x | Alabama, New York, Michigan, New Jersey, Delaware, Utah, Washington DC |
| 5 - Highest | 1.42x | Kentucky, Florida, Arizona, Mississippi, Texas, California, Hawaii, Louisiana, South Carolina |

**Bike type:** Standard 0.95x / Cruiser 1.0x / Touring 1.05x / ADV 1.2x / Sport 2.8x

**Engine size:** Under 500cc 0.75x / 500-999cc 1.0x / 1000cc+ 1.45x

**Age:** 35-49 1.0x / 50+ 1.05x / 25-34 1.2x / 18-24 2.1x

**Experience:** 10+ yrs 0.88x / 5-10 yrs 1.0x / 2-5 yrs 1.15x / Under 2 yrs 1.35x

**Violations:** No 1.0x / Yes 1.28x

### Range and floor/ceiling

Point estimate gets a +/-15% band, then floor/ceiling enforcement:

| Coverage | Monthly Floor | Monthly Ceiling |
|---|---|---|
| Liability Only | $12 | $60 |
| Standard | $35 | $130 |
| Full Coverage | $50 | $250 |

Annual values are calculated from adjusted monthly values.

---

## CTA URLs

### Insurance quote CTA
Pattern: `https://www.riders-share.com/motorcycle-insurance-[state-slug]`

State slug is generated dynamically from the dropdown value. Two-word states use hyphens (`new-york`, `north-carolina`). Washington DC maps to `washington-dc`.

> **DEV FLAG (embedded in JS):** Not all state insurance pages have been verified live. Known live as of build: california, texas, new-york, washington, pennsylvania. All others need 200-response verification before launch. See the comment in the JS near the CTA link logic.

### Listings CTA
Pattern: `https://www.riders-share.com/listings/[state-slug]`

Uses the same slug as the insurance CTA. Links the rider directly to motorcycle listings in their state.

---

## Hosting

This repo is the build and staging environment. The file deploys to GitHub Pages as-is.

**Before going live, confirm with Guillermo:**

**(a) GitHub Pages embed** - iframe src points to `https://[repo].github.io/motorcycle-insurance-calculator/`. Acceptable for staging. Does not pass link equity to the state insurance pages.

**(b) Native Riders Share hosting** - file deployed to `riders-share.com`. Preferred. Dev deploys `index.html` to the Riders Share domain so the iframe src is a first-party URL.

The surrounding Prismic article at `/blog/article/motorcycle-insurance-cost-estimator` handles all SEO. The calculator iframe does not need to rank independently.

---

## Schema (Lives in Prismic, Not Here)

Do not add schema to this file. The Prismic page wrapper handles it.

The Prismic page needs:
- `SoftwareApplication` schema pointing to the calculator URL
- `FAQPage` schema on the surrounding article copy
- Both in `@graph` structure, no script tags (Prismic injects those)

Confirm both are present before launch.

---

## Iframe Embed

Recommended embed in Prismic:

```html
<iframe
  src="https://[your-domain]/motorcycle-insurance-calculator/"
  width="100%"
  style="max-width: 700px; border: none; display: block;"
  title="Motorcycle Insurance Cost Calculator"
  loading="lazy">
</iframe>
```

The calculator is responsive and renders correctly at 700px wide. It adapts for mobile viewports down to ~360px.

---

## What's Not in This File

- No blog copy (lives in Prismic)
- No FAQPage schema (lives in Prismic)
- No navigation, header, or footer
- No Riders Share logo
- No cookies, localStorage, or analytics
- No external API calls
- No build tools or npm

---

## Data Sources

| Source | What It Contributes |
|---|---|
| Progressive (Apr 2024-Mar 2025) | Published liability-only policy data by state; floor calibration |
| Insurance Information Institute | National average band ($500-$1,500/yr) used in output comparison |
| ValuePenguin (Jan 2026) | State cost tier assignments; spot check validation |
| MoneyGeek (Dec 2025) | Secondary state validation |

---

## Spot Check Profiles

These profiles were used to validate the multiplier model before delivery. All outputs fall within the expected ranges.

| Profile | Expected | Output |
|---|---|---|
| Iowa, liability only, cruiser, 500-999cc, 35-49, 10+ yrs, no violations | $12-$20/mo | Pass |
| Average state, full coverage, cruiser, 500-999cc, 35-49, 10+ yrs, no violations | $50-$80/mo | Pass |
| California, full coverage, sport, 1000cc+, 18-24, under 2 yrs, violations | $180-$250/mo | Pass (ceiling) |
| Average state, standard, ADV, 500-999cc, 25-34, 2-5 yrs, no violations | $55-$90/mo | Pass |
| North Dakota, liability only, touring, under 500cc, 50+, 10+ yrs, no violations | $12-$18/mo | Pass |

---

## Credits

Calculator built by [RiZen Metrics](https://www.rizenmetrics.com/) for [Riders Share](https://www.riders-share.com/).
