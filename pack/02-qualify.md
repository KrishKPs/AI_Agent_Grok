# 02 — QUALIFY

The CONTACT / SKIP decision for one company. Runs as STEP 3 of the loop in
`00-mission.md`.

Input is one row: `company_name` and `website`. Output is one file:
`out/<slug>/verdict.md`.

---

## 1. What we sell, and who consumes it

We sell and assemble automobile parts for **low-speed vehicles (LSVs)**: golf carts,
neighborhood electric vehicles (NEVs), and street-legal low-speed carts.

A company is worth contacting when **both** are true:

1. It is a US-based business that **sells, services, or repairs** these vehicles — so it
   consumes parts.
2. It is **not** a competitor that manufactures or wholesales the same parts.

Target: dealers, retailers, and service/repair shops.
Not target: fleet operators, and other parts manufacturers.

That is the entire targeting definition. Do not extend it. A company that is neither
target nor listed non-target is settled by the seven checks below and by nothing else.

---

## 2. Page budget

One counter per company. Starts at 0. **Hard cap 5.**

| Slot | Page | Loaded when |
|---|---|---|
| 1 | `https://<website>` — homepage | Always |
| 2 | `https://www.<domain>` — retry | Only if slot 1 fails |

```
IF   the website value begins with "file://"
THEN load it EXACTLY as written, and skip the slot 2 www retry.
     Slots 3, 4, and 5 are resolved relative to its directory.

IF   the website value begins with "http://" or "https://"
THEN load it EXACTLY as written. The slot 2 www retry STILL APPLIES.
```

The `file://` branch exists so a fixture set can exercise the checks. It changes which
URL is loaded and nothing else — every check below behaves identically. It is scoped to
`file://` alone, because a real list carries `http://` rows that still need the retry.
| 3 | `/about`, then `/about-us`, then `/company` — first that loads | Check 3 needs it |
| 4 | `/contact`, then `/contact-us` — first that loads | Check 4 needs it and the homepage footer has no address |
| 5 | One parts, service, or inventory page linked from the nav | Check 7 needs it |

**Reading the top nav costs 0 loads.** The nav is read from whichever page is already
open. The same is true of the footer.

```
IF   the counter has reached 5
THEN load nothing further
     AND settle every remaining check using its missing-data branch.
```

Never load a page that is not in the table above. Never load a search engine. Never load
a social profile. Never load a directory listing.

---

## 3. Order of checks

Run checks 1 through 7 **in order**. **Stop at the first SKIP.** Write the verdict file
and return to `00-mission.md`.

A company that passes all seven is `CONTACT` with reason code `QUALIFIED`.

| # | Check | Reason code on failure |
|---|---|---|
| 1 | Suppression list | `ON_SUPPRESSION_LIST` |
| 2 | Site reachable | `SITE_DEAD` |
| 3 | Identity match | `IDENTITY_MISMATCH` |
| 4 | US-based | `NON_US` |
| 5 | Competitor guard | `POSSIBLE_COMPETITOR` |
| 6 | LSV core match | `NOT_LSV_BUSINESS` |
| 7 | Buying signal | `NO_BUYING_SIGNAL` |

---

## 4. Text definitions

Used by every check below. Fixed meanings.

**Visible text** — text rendered on the page for a human reader. It **excludes**
`<meta>` tags, the `<title>` tag, `alt` attributes, `title` attributes, `aria-label`
values, HTML comments, `<script>` contents, and `<style>` contents.

**Nav** — the link text of items in the page's primary navigation menu, including
dropdown items. Link text only.

**Heading** — the visible text of an `<h1>` through `<h6>` element.

**Word count** — the number of whitespace-separated tokens in the visible text.

**Term match** — case-insensitive substring match on visible text, with one exception:
the terms `LSV`, `NEV`, and `GEM` match **only** as standalone uppercase tokens
(surrounded by whitespace or punctuation). This prevents a match inside an unrelated
word.

**Company-name normalization — `N(s)`** — apply in this order:

```
1. Lowercase.
2. Replace every character that is not a-z, 0-9, or space with a single space.
   (This removes "&", periods, commas, hyphens, and apostrophes.)
3. Split on whitespace into tokens.
4. Delete every token equal to: inc  llc  co  corp  company  ltd  the
5. Join the remaining tokens with a single space.
```

`N("The Cart Co., Inc.")` = `cart`.

**Domain normalization — `D(s)`** — lowercase; remove `http://`, `https://`; remove a
leading `www.`; remove everything from the first `/` onward; remove any `:port`.

`D("https://www.CartWorld.com/parts")` = `cartworld.com`.

---

## 5. CHECK 1 — Suppression list

**Input:** `inputs/suppression.csv`, columns `company_name` and `domain`.
**Page loads:** 0.

```
STEP 1  IF   inputs/suppression.csv does not exist
             OR it has 0 data rows
        THEN no matches — continue to CHECK 2.

STEP 2  domain_hit = TRUE if D(row website) equals D(any domain cell).
        name_hit   = TRUE if N(row company_name) equals N(any company_name cell).
        Both comparisons are EXACT EQUALITY after normalization, never substring.

STEP 3  IF   domain_hit is TRUE
        THEN SKIP, reason_code ON_SUPPRESSION_LIST,
             evidence = "domain match: <normalized domain>".

        ELSE IF name_hit is TRUE
        THEN SKIP, reason_code ON_SUPPRESSION_LIST,
             evidence = "name match: <normalized name>".

        ELSE continue to CHECK 2.
```

**Tie-breaker.** When `domain_hit` and `name_hit` are both TRUE, record the domain as
evidence. Domain match beats name match.

**Missing data.** A missing file, an empty file, or a header-only file means no matches.
It is never a reason to SKIP. An individual row with a blank `domain` cell contributes
no `domain_hit`; a row with a blank `company_name` cell contributes no `name_hit`.

---

## 6. CHECK 2 — Site reachable

**Input:** the homepage.
**Page loads:** slot 1, and slot 2 only on failure.

```
STEP 1  Load the homepage per the section 2 slot table.   [slot 1]

        IF   it fails (DNS failure, timeout, connection error,
             or HTTP status 400 or higher)
        THEN IF the website value begins with "file://"
             THEN SKIP, reason_code SITE_DEAD,
                  evidence = "load failed: <error or status>".
             ELSE load https://www.<D(website)> ONCE.   [slot 2]

                  IF that also fails
                  THEN SKIP, reason_code SITE_DEAD,
                       evidence = "root and www both failed: <error or status>".

STEP 2  DEAD_MARKERS, matched case-insensitively against visible text:
             "domain is for sale"
             "buy this domain"
             "site can't be reached"
             "404"
             "under construction"

        IF   1 or more DEAD_MARKERS appear
        THEN SKIP, reason_code SITE_DEAD,
             evidence = the first marker in the list above that appeared.

STEP 3  IF   word count of visible text is under 30
        THEN SKIP, reason_code SITE_DEAD,
             evidence = "visible body text = <count> words".

STEP 4  Continue to CHECK 3.
```

**Tie-breaker.** A DEAD_MARKER beats a passing word count. A page with 4,000 words and
the text `buy this domain` is `SITE_DEAD`. STEP 2 runs before STEP 3 for this reason.

**Missing data.** A page that loads but yields no readable visible text has a word count
of 0, which is under 30, so STEP 3 skips it. There is no third outcome.

---

## 7. CHECK 3 — Identity match

Confirms the site belongs to the company named in the sheet.

**Input:** `N(company_name)` from the row; visible text of the homepage and `/about`.
**Page loads:** slot 3, and only if the homepage does not settle it.

```
STEP 1  target = N(company_name from the input row).

        IF   target is an empty string after normalization
        THEN SKIP, reason_code IDENTITY_MISMATCH,
             evidence = "sheet name normalized to empty".

STEP 2  Build the token list of N(homepage visible text).

        IF   the tokens of `target` appear as a CONTINUOUS RUN of whole tokens
             in that list
        THEN continue to CHECK 4, evidence = "name found on homepage".

STEP 3  Load /about, then /about-us, then /company — first that loads.  [slot 3]

        IF   none loads
             OR the budget is exhausted
        THEN SKIP, reason_code IDENTITY_MISMATCH,
             evidence = "name absent from homepage; no about page loaded".

STEP 4  IF   the tokens of `target` appear as a CONTINUOUS RUN of whole tokens
             in N(about page visible text)
        THEN continue to CHECK 4, evidence = "name found on /about".
        ELSE SKIP, reason_code IDENTITY_MISMATCH,
             evidence = "name absent from homepage and /about".
```

**Whole-token run.** `target` must match complete tokens, in order, with nothing between
them. `cart world` matches `... premier cart world dealer ...`. It does not match
`golfcartworld` (one token) and it does not match `cart superstore world` (interrupted).

**Partial match is NO.** A match on some tokens of `target` but not all of them, in one
continuous run, fails the check.

**Tie-breaker.** Homepage is checked before `/about`. The first page that matches is the
one recorded as evidence.

**Missing data.** No `/about` page and no homepage match is `IDENTITY_MISMATCH`, not a
continue. This is the one place where absent information causes a SKIP, because an
unverified identity makes every later check meaningless.

---

## 8. CHECK 4 — US-based

Skips only on a **clear** non-US signal. Absent information never skips.

**Input:** visible text of the homepage footer and, when needed, `/contact`.
**Page loads:** slot 4, and only when the homepage footer carries no address.

```
STEP 1  Read the homepage footer.  [0 loads]

        IF   no postal address appears there
             AND the budget allows
        THEN load /contact, then /contact-us — first that loads.  [slot 4]

STEP 2  Collect, from every page loaded so far:
        us_address     = an address block containing "United States", "USA",
                         or "U.S.A.", OR a 2-letter USPS state abbreviation
                         immediately followed by a 5-digit ZIP (optionally +4).
        foreign_address= an address block whose final line names a country that
                         is not the United States.
        phone_codes    = the country code of every phone number shown.
        currencies     = every currency symbol or code attached to a price.

STEP 3  IF   us_address is present
        THEN location = US, continue to CHECK 5,
             evidence = the matched state+ZIP or country line.

        ELSE IF foreign_address is present
        THEN SKIP, reason_code NON_US,
             evidence = "address country: <country>".

        ELSE IF 1 or more phone numbers were found
             AND every one carries a country code other than +1
        THEN SKIP, reason_code NON_US,
             evidence = "phone country code <code>".

        ELSE IF 1 or more prices were found
             AND no price is in USD
             AND 1 or more prices are in another currency
        THEN SKIP, reason_code NON_US,
             evidence = "prices in <currency>, no USD".

        ELSE location = unknown, continue to CHECK 5.
```

**Tie-breaker.** A US address anywhere beats every foreign signal. STEP 3 tests
`us_address` first for this reason: a US dealer listing one overseas phone number stays
in. Among foreign signals the order is address, then phone, then currency, and the first
one to fire is the evidence recorded.

**Missing data.** No address, no phone, and no price means `location = unknown` and the
company **continues**. Never SKIP for unknown location.

---

## 9. CHECK 5 — Competitor guard

Removes companies that make or wholesale the parts we sell.

**Input:** visible text of every page loaded so far.
**Page loads:** 0.

```
STEP 1  COMPETITOR_PHRASES, matched case-insensitively:
             "parts manufacturer"
             "parts wholesaler"
             "wholesale distributor of parts"
             "OEM parts supplier"
             "aftermarket parts distributor"

STEP 2  self_claim = TRUE when a COMPETITOR_PHRASE appears in a sentence that
        ALSO contains one of: "we", "our", "us", or N(company_name).
        A phrase inside a customer list, a brand list, or a link to another
        business does not set self_claim.

        IF   self_claim is FALSE
        THEN continue to CHECK 6.

STEP 3  WHOLE_VEHICLE_TERMS, matched against visible text, nav, or headings:
             "carts for sale"    "shop carts"      "inventory"
             "new carts"         "used carts"      "vehicles for sale"
             "cart sales"        "service"         "repair"
             "maintenance"

        IF   1 or more WHOLE_VEHICLE_TERMS appear
        THEN continue to CHECK 6.
             The company sells or services whole vehicles, so it consumes
             parts, so it is a customer.

        ELSE SKIP, reason_code POSSIBLE_COMPETITOR,
             evidence = "<matched phrase>; no whole-vehicle term found".
```

**Tie-breaker.** Doing both makes it a customer. When a competitor phrase and a
whole-vehicle term both appear, the company **continues**. STEP 3 exists only to give
that outcome a countable test.

**Missing data.** No competitor phrase found means continue. The absence of a phrase is
never itself a SKIP.

---

## 10. CHECK 6 — LSV core match

Confirms the company works with these vehicles at all.

**Input:** visible text and nav of the homepage.
**Page loads:** 0.

```
STEP 1  LSV_TERMS, in this order:
             golf cart              golf carts
             low-speed vehicle      LSV
             neighborhood electric vehicle
             NEV                    street-legal cart
             utility vehicle        Club Car
             E-Z-GO                 EZGO
             GEM                    Bintelli
             Evolution              ICON EV
             Star EV                Advanced EV
             Tomberlin              Polaris GEM
             Yamaha golf

STEP 2  IF   1 or more LSV_TERMS appear in the homepage VISIBLE TEXT or NAV
        THEN continue to CHECK 7,
             evidence = the first term in the list above that matched.
        ELSE SKIP, reason_code NOT_LSV_BUSINESS,
             evidence = "no LSV term in homepage text or nav".
```

**Visible text or nav only.** A term found only in a `<meta>` tag, the `<title>`, an
`alt` attribute, or an HTML comment does **not** count. Apply the visible-text
definition in section 4 without exception.

**`LSV`, `NEV`, and `GEM`** match only as standalone uppercase tokens, per section 4.

**Tie-breaker.** When several terms match, record the one that appears earliest in the
STEP 1 list. The list order is fixed so two runs on the same page produce the same
evidence string.

**Missing data.** CHECK 2 guarantees at least 30 words of visible text, so this check
always has text to read. Zero matches is a decision, not missing data.

---

## 11. CHECK 7 — Buying signal

Confirms the company buys parts or does service work.

**Input:** nav and headings of every page loaded so far.
**Page loads:** slot 5, and only when nothing has matched yet.

```
STEP 1  BUYING_TERMS, in this order:
             Parts              Service            Repair
             Maintenance        Accessories        Shop Parts
             Service & Repair

STEP 2  IF   1 or more BUYING_TERMS appear as a NAV item or a HEADING
        THEN CONTACT, reason_code QUALIFIED,
             evidence = "nav/heading: <matched term>".

        A dedicated Parts nav item alone qualifies.
        A dedicated Service nav item alone qualifies.

STEP 3  SALES_STATEMENTS, matched against visible text:
             "carts for sale"   "shop carts"   "inventory"

        IF   1 or more SALES_STATEMENTS appear
        THEN CONTACT, reason_code QUALIFIED,
             evidence = "sales statement: <matched term>".

STEP 4  IF   the budget allows
             AND the nav has a link whose text matches a BUYING_TERM
                 or a SALES_STATEMENT
        THEN load that page ONCE and repeat STEP 2 and STEP 3 on it.  [slot 5]

STEP 5  SKIP, reason_code NO_BUYING_SIGNAL,
        evidence = "no parts, service, or inventory item in nav or headings".
```

**Tie-breaker.** Nav and heading matches (STEP 2) are tested before visible-text sales
statements (STEP 3). When both would match, record the STEP 2 term. Within a step,
record the term appearing earliest in its list.

**Missing data.** An exhausted budget at STEP 4 goes straight to STEP 5 and SKIPs with
`NO_BUYING_SIGNAL`. It does not halt, and it does not load a sixth page.

---

## 12. Output — `out/<slug>/verdict.md`

Written for **every** company, whether `CONTACT` or `SKIP`, including a SKIP at CHECK 1
with 0 pages loaded.

`<slug>` is `company_name`, lowercased, every run of non-alphanumeric characters
replaced by a single `-`, leading and trailing `-` removed.

```markdown
---
company: <company_name, exactly as in the input row>
website: <website, exactly as in the input row>
verdict: <CONTACT | SKIP>
reason_code: <one code from the table in section 3, or QUALIFIED>
evidence: <the exact term, phrase, or page that triggered the verdict>
location: <US | unknown>
pages_loaded: <integer 0 through 5>
---
```

Field rules:

- `verdict` is `CONTACT` or `SKIP`. No other value.
- `reason_code` is `QUALIFIED` when `verdict` is `CONTACT`, and one of the seven failure
  codes when `verdict` is `SKIP`. No other value.
- `evidence` quotes the **exact** matched term and names the page it was found on. Never
  a paraphrase, never a summary.
- `location` is `US` or `unknown`. A company that reached CHECK 5 or later has one of
  these two. A company that SKIPped before CHECK 4 records `unknown`.
- `pages_loaded` is the counter from section 2 at the moment the verdict was reached.

```
IF   out/<slug>/verdict.md already exists
THEN skip this company and move to the next.
```

**After writing the file:**

```
IF   verdict is SKIP
THEN append the row to out/skipped.csv per 05-output.md
     AND return to 00-mission.md STEP 7.

IF   verdict is CONTACT
THEN continue to 00-mission.md STEP 4 (draft the cold email).
```
