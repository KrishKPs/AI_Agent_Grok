# source.md — where the company list comes from

**Optional.** When this file is absent, or `published_csv_url` still reads `FILL_ME`,
the run reads `inputs/companies.csv` from disk instead.

When it holds a URL, that URL is fetched once at the start of the run and its rows
become the company list.

---

```yaml
# The Google Sheets "Publish to web" CSV link. It must end in output=csv.
# Format: https://docs.google.com/spreadsheets/d/e/2PACX-.../pub?gid=<id>&single=true&output=csv
published_csv_url:  https://docs.google.com/spreadsheets/d/1k9XoeILHH3hXyC7mJTBEOSJyf2H1RNiW-FnpA3Rsu8U/edit?usp=sharing
```

---

## Requirements

1. The URL must be reachable **without logging in.** Grok Bot never authenticates. A URL
   that returns a sign-in page halts the entire run.
2. The fetched CSV must have the columns `company_name` and `website`. Extra columns are
   ignored. A row missing either value is skipped as `BAD_INPUT`.
3. The URL must end in `output=csv`. An `/edit` or `/view` link serves HTML, not CSV, and
   halts the run.

## What publishing exposes

`Publish to web` makes the published tab readable by **anyone with the link**, with no
Google account required. It is a public URL.

To keep that surface small:

- Publish **one dedicated tab** holding only `company_name` and `website` — never the
  whole workbook, and never a tab carrying notes, contacts, pricing, or personal data.
- Publishing is reversible: `File → Share → Publish to web → Stop publishing`.
- Unpublishing does not un-cache. Treat anything published as having been seen.
