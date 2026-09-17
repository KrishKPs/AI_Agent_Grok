# GROK.md — the workflow, in one page

An overview of what the executor does. The binding rules are in `pack/`, bundled for
delivery as `GROK_BRIEF.md`. Where this page and the pack differ, **the pack wins** —
this page is a map, not a rule file.

---

## The shape of it

You are handed a list of companies: a name and a website, nothing more. For each one you
research, decide, and — only for the ones that pass — draft two messages for a human to
read.

**You never send anything.** Drafts go to disk and stop there.

---

## Once, at the start of the run

Fetch the company list from the published sheet URL in `inputs/source.md`.

```
Check the FIRST LINE contains "company_name" and "website".
Never judge the Content-Type header — Google serves CSV as application/binary.

IF the response is a sign-in page, or the headers are wrong, or status >= 400
THEN HALT THE ENTIRE RUN. Write out/INCIDENT.md.
     Do NOT log in. Do NOT fall back to a stale local file.
```

Read only `company_name` and `website`. Columns C through I are the human's review
columns, written back from an earlier run. Ignore them. A row that already holds a
verdict is processed again exactly like any other.

---

## For each company

One at a time, in row order, to completion or to a halt. Never two at once.

### 1. Validate
Name or website blank → `BAD_INPUT`, next company.

### 2. Decide CONTACT or SKIP
Seven checks in order. **Stop at the first failure.** Budget: **5 page loads, hard cap.**

| # | Check | Fails as |
|---|---|---|
| 1 | On the suppression list? | `ON_SUPPRESSION_LIST` |
| 2 | Site loads, not parked, 30+ words? | `SITE_DEAD` |
| 3 | Does the sheet name appear on the site? | `IDENTITY_MISMATCH` |
| 4 | Any clear non-US signal? | `NON_US` |
| 5 | Do they call themselves a parts maker or wholesaler? | `POSSIBLE_COMPETITOR` |
| 6 | Any golf-cart or LSV term present? | `NOT_LSV_BUSINESS` |
| 7 | A Parts or Service nav item, or inventory? | `NO_BUYING_SIGNAL` |

Write `out/<slug>/verdict.md` for **every** company, CONTACT and SKIP alike, recording
the exact term or page that decided it.

SKIP → write the row to `out/skipped.csv`, then go to step 7.

### 3. Research — CONTACT only
Collect what they sell, brands carried, hiring, recent news, and one contact ranked
**Owner → General Manager → Parts Manager**. Tie-break alphabetically by last name.

Anything not stated on the page is the literal string `NOT_FOUND`. Never estimated,
never inferred. Budget: 8 loads, 2 searches, 10 minutes.

### 4. Draft the cold email
Six fixed parts, 30–110 words, one question mark, zero links.

- Facts about **the company** come only from `research.json`.
- Claims about **us** are copied **verbatim** from `offer.md` — never reworded.
- A sentence tracing to neither is deleted.

### 5. Draft the LinkedIn message
Under 280 characters. Must open on a **different** fact than the email did.

Loads zero pages. **Never opens LinkedIn.**

### 6. File it
Draft paths into `out/review-queue.csv` with `reviewed=no`.

### 7. One row into `sheet-import.csv`
For **every** input row, in input order — including skips, `BAD_INPUT`, and `RULE_GAP`.

A paste-back is positional. One missing row shifts everything below it and writes each
verdict against the wrong company. Never sort, never group, never omit.

### 8. Next company.

---

## What exists at the end

```
out/<slug>/verdict.md        every company
out/<slug>/research.json     CONTACTs only
out/<slug>/email.md          CONTACTs only
out/<slug>/linkedin.md       CONTACTs only
out/review-queue.csv         the human's work list
out/skipped.csv              every SKIP, with its reason code
out/sheet-import.csv         paste into the sheet at C2
```

Every company lands in `review-queue.csv` **or** `skipped.csv`. Never both, never
neither.

---

## Four things that never bend

**Never send.** No Send, Connect, Add Note, Submit, Reply, Follow, InMail, or contact
form. No control whose effect is to transmit a message to a person.

**Never log in.** Not to email, not to LinkedIn, not to Google. A login wall is a halt,
never a prompt to authenticate.

**Never invent a fact.** Every sentence traces to `research.json` or to `offer.md`.
There is no third source, and nothing comes from memory.

**Never invent a rule.** An uncovered situation is a halt: log `RULE_GAP` naming the
exact decision you could not make, then move on.

That last one matters most. You cannot ask a question — so logging the gap **is** how
you ask it. A gap that gets logged gets fixed. A gap that gets improvised around becomes
a wrong message to a real person, and nobody finds out why.

---

## Halting is a success

A halted company costs one row in a log. A guessed company costs a real person a wrong
message. When the two are in tension, halt.
