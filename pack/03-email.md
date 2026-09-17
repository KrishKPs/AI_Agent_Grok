# 03 — COLD EMAIL DRAFT

Runs as STEP 4 of the loop in `00-mission.md`, for CONTACT companies only.

Output: `out/<slug>/email.md`. This file is a draft. It is never sent.

---

## 1. Inputs

Three sources, and no others.

| Source | Supplies |
|---|---|
| `out/<slug>/research.json` | Every fact about the company |
| `out/<slug>/verdict.md` | The `evidence` string that qualified them |
| `inputs/offer.md` | Every claim about **us** |

```
IF   inputs/offer.md does not exist
     OR any required field in section 2 is absent from it
     OR any field still holds the literal placeholder FILL_ME
THEN halt the company with reason_code RULE_GAP,
     detail "inputs/offer.md missing or unfilled field <name>".
     Do NOT write an email. Do NOT invent the missing text.
     FILL_ME is never copied into a draft.
```

---

## 2. `inputs/offer.md` — required fields

A human writes this file once. It is the only place a claim about our company,
our parts, our pricing, or our lead times may come from.

```yaml
sender_name:       <string>
sender_title:      <string>
sender_company:    <string>
offer_sentence:    <string, max 25 words — what we sell>
proof_line:        <string, max 20 words — or the literal NOT_SUPPLIED>
cta_sentence:      <string, max 15 words — one ask>
fallback_greeting: <string, used when contact_name is NOT_FOUND>
generic_opener:    <string, max 20 words — used when every personalization
                    field is NOT_FOUND>
signature_block:   <string, max 4 lines>
```

**These strings are copied verbatim into the draft.** Never reword them. Never shorten
them. Never expand them. Never merge two of them into one sentence. If a string reads
awkwardly in place, that is a defect in `inputs/offer.md` for a human to fix, and not
something to smooth over.

---

## 3. Structure

Exactly six parts, in this order. No part may be added, removed, or reordered.

```
1. SUBJECT      one line, per section 5
2. GREETING     per section 4
3. OPENER       exactly ONE sentence, per section 6
4. OFFER        offer_sentence, verbatim
5. PROOF        proof_line, verbatim — OMITTED ENTIRELY if NOT_SUPPLIED
6. ASK          cta_sentence, verbatim
                then signature_block, verbatim
```

---

## 4. Greeting

```
IF   contact_name is NOT_FOUND
THEN greeting = fallback_greeting
ELSE greeting = "Hi " + the contact's FIRST name only.
```

The first name is the first whitespace-separated token of `contact_name`. Never use a
title, never guess a nickname, never shorten a name.

---

## 5. Subject line

Max **55 characters**. Chosen by which field the opener used, per section 6.

| Opener source | Subject template |
|---|---|
| `lsv_terms_found` | `<matched term> parts — <sender_company>` |
| `open_roles` | `Parts support for <company_name>` |
| `recent_news` | `Parts support for <company_name>` |
| `offer_summary` | `Parts support for <company_name>` |
| `generic_opener` | `Parts support for <company_name>` |

```
IF   the filled template exceeds 55 characters
THEN use "Parts support for <company_name>".

IF   that also exceeds 55 characters
THEN use "Parts support".
```

No emoji. No `Re:`. No `Fwd:`. No ALL CAPS word. No exclamation mark.

---

## 6. Opener — exactly one sentence

Use the **first** source below that is not `NOT_FOUND` and not empty. Fixed order, so
the same research file always produces the same opener.

| Priority | Field | Sentence |
|---|---|---|
| 1 | `lsv_terms_found` | `Saw that <company_name> works with <first term in the list>.` |
| 2 | `open_roles` | `Saw <company_name> is hiring for <first role in the list>.` |
| 3 | `recent_news` | `Saw the news about <headline>.` |
| 4 | `offer_summary` | `Saw what <company_name> does — <offer_summary>.` |
| 5 | none of the above | `generic_opener`, verbatim |

**The opener states one captured fact and stops.** It does not praise the company, does
not describe the company back to itself, and does not guess why they do what they do.

```
IF   the opener would exceed 20 words
THEN drop to the next priority.

IF   priority 5 is reached
THEN use generic_opener, verbatim.
```

---

## 7. Length

| Cap | Value |
|---|---|
| Subject | 55 characters |
| Body words | 30 minimum, 110 maximum |
| Body sentences | 6 maximum |
| Question marks | 1 maximum, and only inside `cta_sentence` |
| Links | 0 |
| Attachments | 0 |

Body word count excludes the subject line and the signature block.

The 30-word floor is a structural check, not a style target. A four-part body with
`proof_line: NOT_SUPPLIED` lands near 40 words, and that is a finished draft.

```
IF   the body is under 30 words
THEN it is missing a required part — re-check section 3.

IF   the body exceeds 110 words
THEN the offer_sentence, proof_line, or cta_sentence in inputs/offer.md is
     over its own cap. Halt with RULE_GAP, detail "offer.md field over cap".
     Do NOT shorten a verbatim string to fit.
```

---

## 8. Banned in the draft

These do not appear in the output, in any casing or wording:

`I hope this email finds you well` · `Quick question` · `Just following up` ·
`Circle back` · `Touch base` · `Reach out` · `Synergy` · `Leverage` ·
`Game-changer` · `Revolutionary` · `Best-in-class` · `World-class` ·
`Industry-leading` · `Cutting-edge` · `I came across your website` ·
`I noticed you're doing great things` · `As you know` · `I'll keep this brief`

Also banned:

- **Praise of the company.** No adjective describing them — no `impressive`,
  `great`, `amazing`, `fantastic`, `leading`.
- **Invented urgency.** No deadline, no `limited time`, no `spots left`.
- **Invented familiarity.** No claimed mutual connection, no claimed prior contact, no
  `following up on my last email` when no email was sent. Nothing was sent.
- **Numbers we did not supply.** No percentage, price, discount, lead time, or statistic
  unless it appears verbatim in `inputs/offer.md`.
- **Claims about their business** beyond the captured field the opener cites.

---

## 9. The evidence check

Before writing, run this on every sentence:

```
IF   the sentence states a fact about THE COMPANY
     AND that fact is not a captured field in out/<slug>/research.json
THEN delete the sentence.

IF   the sentence states a fact about US
     AND that text is not copied verbatim from inputs/offer.md
THEN delete the sentence.
```

Every sentence in the draft traces to one file or the other. There is no third source.
Nothing comes from memory.

---

## 10. Output

Write `out/<slug>/email.md` with the header block from `05-output.md` section 5, then:

```markdown
**Subject:** <subject line>

<greeting>,

<opener sentence>

<offer_sentence>

<proof_line — omit this block entirely if NOT_SUPPLIED>

<cta_sentence>

<signature_block>
```

`status` in the header is always `DRAFT — NOT SENT`.

Do not open an email client. Do not copy this text into a compose window. Do not click
any send control. Writing the file is the whole task.
