# 01 — RESEARCH

For one company. Runs as STEP 3 of the loop in `00-mission.md`, for companies whose
verdict in `02-qualify.md` was **CONTACT**. Never run for a SKIPped company.

Your job is to fill the field table in section 4 and write it to
`out/<slug>/research.json`. You are filling a form, not forming an impression.

---

## 1. Budget

Hard caps for one company. Count as you go.

| Cap | Value |
|---|---|
| Page loads | 8 |
| Search engine queries | 2 |
| Page load attempts per URL | 3 |
| Wall-clock time | 10 minutes |

```
IF   any cap is reached
THEN stop research immediately
     AND write the research file with every field found so far
     AND set all unfilled fields to NOT_FOUND
     AND halt the company with reason_code BUDGET_EXCEEDED.
```

A partly-filled research file is still written. It is evidence of what was checked.

---

## 2. Source order

Visit in this order. Stop early once every field in section 4 has a value or has been
marked `NOT_FOUND`.

1. `<website>` — the homepage
2. `<website>/about` — then `/about-us`, then `/company`, first one that loads
3. `<website>/pricing` — then `/products`, then `/services`, first one that loads
4. `<website>/careers` — then `/jobs`, first one that loads
5. The company's public LinkedIn company page
6. Search query: `"<company_name>" funding OR raises OR launches` — news only
7. Search query: `"<company_name>" "Owner" OR "General Manager" OR "Parts Manager"` —
   contact only

```
IF   the homepage does not load after 3 attempts
THEN halt with reason_code SITE_UNREACHABLE.

IF   a page other than the homepage does not load after 3 attempts
THEN skip it and continue to the next source.
```

### Access limits

Do not log in. Do not create an account. Do not solve a captcha. Do not bypass a paywall
or a login wall. Do not read a page that requires authentication.

```
IF   a source requires any of the above to read
THEN treat that source as unavailable
     AND continue to the next source in the list.
```

---

## 3. Extraction rules

These apply to every field.

1. **Copy, do not summarise, and do not infer.** A field's value comes from text visible
   on the page.
2. **Record the source URL** for every field you fill. A field with no source URL is
   invalid and must be set to `NOT_FOUND`.
3. **Never guess.** If the page does not state it, the field is `NOT_FOUND`. Company
   size is not estimated from office photos. Industry is not deduced from a logo.
4. **`NOT_FOUND` is a literal string.** A field is never blank and never null.
5. **First match wins.** When a source gives two values for one field, take the one
   that appears earliest in the source order in section 2.
6. **Controlled values only.** Fields marked *enum* below take one of the listed values
   and nothing else. If the page's wording matches none of them, the field is
   `NOT_FOUND`.

---

## 4. Fields to collect

| Field | Type | Rule |
|---|---|---|
| `company_name` | string | Copied from the input row, unchanged |
| `website` | string | Copied from the input row, unchanged |
| `offer_summary` | string | One sentence, max 25 words, describing what the company sells. Taken from the homepage headline or the first paragraph of the about page |
| `industry` | enum | One of the values in section 5 |
| `employee_count_band` | enum | `1-10`, `11-50`, `51-200`, `201-500`, `501-1000`, `1001+`. From the LinkedIn company page's stated size, or a headcount stated on the about page |
| `hq_country` | string | Country name as written on the site |
| `hq_city` | string | City name as written on the site |
| `lsv_terms_found` | list | Every term from the LSV_TERMS list in `02-qualify.md` section 10 that appears in visible page text. Empty list if none appear |
| `hiring_now` | enum | `yes` if the careers page lists 1 or more open roles, `no` if it loads and lists 0, `NOT_FOUND` if no careers page loads |
| `open_roles` | list | Job titles listed on the careers page, max 10 |
| `recent_news` | object | `{headline, date, url}` for one item dated within the last 180 days. `NOT_FOUND` if none is dated within 180 days |
| `contact_name` | string | Per section 6 |
| `contact_title` | string | Per section 6 |
| `contact_profile_url` | string | Per section 6 |
| `sources` | object | Field name → the URL it came from, for every filled field |
| `research_completed_at` | timestamp | ISO 8601, UTC |
| `budget_used` | object | `{page_loads, search_queries, minutes}` |

Fields consumed by the qualify decision are named in `02-qualify.md`. When that file
requires a field not listed above, that is a defect in this file — halt with `RULE_GAP`
rather than inventing a field.

---

## 5. Industry enum

Assign the **first** value in this list whose label appears in the company's own
description of itself. Never assign by impression.

`software` · `fintech` · `healthcare` · `ecommerce` · `retail` · `manufacturing` ·
`logistics` · `construction` · `real-estate` · `education` · `hospitality` ·
`professional-services` · `media` · `energy` · `agriculture` · `nonprofit` ·
`government` · `other`

```
IF   the company's own description matches no label above
THEN industry = "other".

IF   the company gives no description of itself on any visited source
THEN industry = NOT_FOUND.
```

`other` and `NOT_FOUND` are different. `other` means it was described and did not match.
`NOT_FOUND` means it was never described.

---

## 6. Contact selection

One contact per company. Deterministic.

```
STEP 1  Collect every person named on the visited sources who has BOTH
        a full name AND a job title.

STEP 2  Keep only people whose title matches the BUYER_TITLE_PRIORITY list.

        BUYER_TITLE_PRIORITY, highest priority first:
          1. Owner            — also matches: Proprietor, Founder, Co-Owner
          2. General Manager  — also matches: GM, Managing Partner
          3. Parts Manager    — also matches: Parts Director, Parts Lead

        Match is case-insensitive on the full job title string. A title
        matches an entry when the entry, or one of its listed alternates,
        appears as a whole-token run inside the title.
        "Owner / Operator" matches entry 1. "Service Manager" matches none.

        IF   a person's title matches TWO entries
        THEN assign the higher-priority entry (lower number wins).

STEP 3  IF   0 people remain
        THEN contact_name, contact_title, contact_profile_url = NOT_FOUND.

        IF   1 person remains
        THEN that is the contact.

        IF   2 or more remain
        THEN take the one whose title is highest in the priority list.
             Tie-break: alphabetically by last name, A first.

STEP 4  Capture a public professional profile URL for the chosen person.
        IF none is public, contact_profile_url = NOT_FOUND.
        The contact is still valid without it.
```

Collect business role information only: name, job title, employer, public professional
profile URL. Do not collect or record a personal email address, a personal phone number,
a home location, or any personal detail about the individual.

Never guess an email address. Never construct one from a pattern.

---

## 7. Writing the file

Write `out/<slug>/research.json`.

`<slug>` is `company_name`, lowercased, with every run of non-alphanumeric characters
replaced by a single `-`, and leading and trailing `-` removed.

```
IF   out/<slug>/research.json already exists
THEN skip this company entirely and move to the next.
```

Write this file before making the qualify decision. Always.
