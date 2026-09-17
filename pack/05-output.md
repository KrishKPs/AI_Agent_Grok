# 05 — OUTPUT

Exact file names, locations, and schemas. Runs as STEP 6 of the loop in
`00-mission.md`, and governs every write you make at any step.

You write only inside `out/`. Never `pack/`, never `CLAUDE.md`, never `inputs/`.

---

## 1. The slug

Used in every path below.

```
slug = company_name
       lowercased
       every run of non-alphanumeric characters replaced by a single "-"
       leading and trailing "-" removed
```

`Acme Robotics, Inc.` becomes `acme-robotics-inc`.

The same slug is used for a company everywhere. Never two slugs for one company.

---

## 2. What gets written, and when

| Step | File | Written for |
|---|---|---|
| 2 | `out/<slug>/verdict.md` | **Every** company past `BAD_INPUT` — CONTACT and SKIP alike. Schema in `02-qualify.md` section 12 |
| 2 | `out/skipped.csv` | Every SKIPped, errored, or halted company |
| 3 | `out/<slug>/research.json` | CONTACT companies only |
| 4 | `out/<slug>/email.md` | CONTACT companies only |
| 5 | `out/<slug>/linkedin.md` | CONTACT companies only |
| 6 | `out/review-queue.csv` | CONTACT companies only |
| 6 | `out/sheet-import.csv` | **Every** input row, in input order — see section 4a |

A company appears in `review-queue.csv` **or** `skipped.csv`. Never both, never neither.
Every company gets a `verdict.md` regardless of which one it lands in, and every input
row gets exactly one `sheet-import.csv` row.

---

## 3. `out/review-queue.csv`

The human's work list. The only file a reviewer needs to open.

Header, written once when the file is created:

```csv
company_name,website,contact_name,contact_title,reason,email_draft_path,linkedin_draft_path,reviewed
```

| Column | Value |
|---|---|
| `company_name` | From the research file |
| `website` | From the research file |
| `contact_name` | From the research file, or `NOT_FOUND` |
| `contact_title` | From the research file, or `NOT_FOUND` |
| `reason` | The qualify reason from `02-qualify.md`, naming which tests passed |
| `email_draft_path` | `out/<slug>/email.md` |
| `linkedin_draft_path` | `out/<slug>/linkedin.md` |
| `reviewed` | The literal string `no` |

```
`reviewed` is ALWAYS written as "no".
You never write "yes". You never change a row whose value is "yes".
Only a human changes that column.
```

Append one row. Never rewrite an existing row. Never reorder the file.

---

## 4. `out/skipped.csv`

Header, written once when the file is created:

```csv
company_name,website,reason_code,detail
```

`reason_code` is one of the codes in section 5 of `00-mission.md` and nothing else.

`detail` is one sentence of fact. For `RULE_GAP` it must name the exact decision you
could not make — that sentence is what gets the pack fixed. For `INSUFFICIENT_DATA` it
must name which field was `NOT_FOUND`.

---

## 4a. `out/sheet-import.csv`

One file the human pastes back into the Google Sheet. It mirrors the sheet's columns
exactly, so a paste lands every value in the right place.

Header, written once when the file is created:

```csv
company_name,website,verdict,reason_code,evidence,location,email_draft_path,linkedin_draft_path,reviewed
```

**ROW ALIGNMENT IS THE WHOLE POINT OF THIS FILE.**

```
Write EXACTLY ONE row per input row, in INPUT ROW ORDER.

This includes rows halted as BAD_INPUT, rows halted as RULE_GAP, and every
SKIP. A company that produced no verdict still produces a row.

Never sort. Never group. Never omit. Never merge two rows.
A missing or reordered row shifts every row beneath it and writes the wrong
verdict against the wrong company.
```

| Column | CONTACT row | SKIP row |
|---|---|---|
| `company_name` | From the input row, unchanged | Same |
| `website` | From the input row, unchanged | Same |
| `verdict` | `CONTACT` | `SKIP` |
| `reason_code` | `QUALIFIED` | The reason code |
| `evidence` | The `evidence` string from `verdict.md` | Same |
| `location` | `US` or `unknown` | `US` or `unknown` |
| `email_draft_path` | `out/<slug>/email.md` | Blank |
| `linkedin_draft_path` | `out/<slug>/linkedin.md` | Blank |
| `reviewed` | The literal string `no` | Blank |

A row halted before a verdict was reached carries `verdict = SKIP`, its halt code in
`reason_code`, the halt detail in `evidence`, and `unknown` in `location`.

`reviewed` is written as `no` and never as `yes`. Only a human changes it, in the sheet.

---

## 5. Draft files

One folder per qualified company: `out/<slug>/`.

Both files carry this header block, so a reviewer reading a draft alone can see what it
was built from and that it has not been sent:

```markdown
---
company: <company_name>
website: <website>
contact: <contact_name> — <contact_title>
research: out/<slug>/research.json
status: DRAFT — NOT SENT
generated: <ISO 8601 UTC>
---
```

`status` is always the literal string `DRAFT — NOT SENT`. There is no other value.

Body content is governed by `03-email.md` and `04-linkedin.md`.

---

## 6. Write discipline

1. **Append, never overwrite.** For the two CSVs, add a row. Never rewrite the file.
2. **One row per company per file.** Before appending, check whether the slug is already
   present. If it is, skip the company and move on.
3. **Write the header only if the file does not exist.**
4. **Finish a company's writes before starting the next company.**
5. **Never delete anything in `out/`.**

---

## 7. Encoding

UTF-8. Unix line endings.

In CSV values: wrap any value containing a comma, a quote, or a newline in double
quotes, and double any internal quote. Never let a draft body into a CSV cell — the CSV
carries the path to the draft, not the draft.
