# 04 — LINKEDIN MESSAGE DRAFT

Runs as STEP 5 of the loop in `00-mission.md`, for CONTACT companies only.

Output: `out/<slug>/linkedin.md`. This file is a draft. It is never sent.

---

## 1. Browser rule

**Do not open LinkedIn.** Do not log in. Do not view a profile. Do not click Connect. Do
not open a message window.

Everything needed is already in `out/<slug>/research.json` and `inputs/offer.md`. This
step loads 0 pages.

```
IF   writing this draft appears to require opening LinkedIn
THEN halt with reason_code RULE_GAP,
     detail "LinkedIn draft required a page load".
```

---

## 2. Inputs

The same three sources as `03-email.md`, with the same rule: facts about the company
come from `research.json`, claims about us are copied verbatim from `inputs/offer.md`.

```
IF   inputs/offer.md does not exist
     OR a required field is absent
THEN halt with reason_code RULE_GAP.
```

---

## 3. It is not the email

The LinkedIn message is shorter and opens differently. Two hard rules:

```
RULE A  The opener sentence must NOT be the same sentence used in
        out/<slug>/email.md.

RULE B  Use the NEXT available field in the section 6 priority order of
        03-email.md — the one after the field the email used.

        IF   no further field is available
        THEN use the no-personalization opener in section 5 below.
```

This is deterministic: the email takes priority 1, so LinkedIn takes priority 2. If the
email took priority 3, LinkedIn takes priority 4.

---

## 4. Structure

Exactly four parts. No subject line. No signature block.

```
1. GREETING   per 03-email.md section 4
2. OPENER     exactly ONE sentence, per section 3 above
3. OFFER      offer_sentence, verbatim
4. ASK        cta_sentence, verbatim
```

`proof_line` is **omitted**. There is no room for it inside the character cap.

---

## 5. No-personalization opener

Used when RULE B finds no further field.

```
opener = "I work with <sender_company> on low-speed vehicle parts."
```

`<sender_company>` is copied from `inputs/offer.md`. Nothing else in this sentence
changes, ever.

---

## 6. Length

| Cap | Value |
|---|---|
| Total characters | **280 maximum**, counting spaces and line breaks |
| Total characters | 80 minimum |
| Sentences | 4 maximum |
| Question marks | 1 maximum, and only inside `cta_sentence` |
| Links | 0 |
| Emoji | 0 |
| Line breaks | 3 maximum |

280 sits under LinkedIn's 300-character connection-note limit, so one draft serves
either a connection note or a first message. A human decides which.

```
IF   the draft exceeds 280 characters
THEN drop the OPENER and use the section 5 sentence instead, then recount.

IF   it still exceeds 280 characters
THEN halt with reason_code RULE_GAP,
     detail "offer_sentence + cta_sentence exceed 280 characters".
     Do NOT trim a verbatim string.
```

---

## 7. Banned

Everything banned in `03-email.md` section 8 is banned here, plus:

- `I'd love to connect` · `Let's connect` · `Growing my network` ·
  `I see we're both in` · `Fellow <anything>`
- Any claim of having viewed their profile.
- Any claim of a shared group, school, employer, or connection.
- Any hashtag.

---

## 8. The evidence check

Identical to `03-email.md` section 9. Every sentence traces to `research.json` or to
`inputs/offer.md`. Nothing comes from memory.

---

## 9. Output

Write `out/<slug>/linkedin.md` with the header block from `05-output.md` section 5,
then:

```markdown
**Character count:** <integer>

<greeting>,

<opener sentence>

<offer_sentence>

<cta_sentence>
```

The character count covers the message body only — the greeting through the ask,
excluding the header block.

`status` in the header is always `DRAFT — NOT SENT`.

After writing this file, return to `00-mission.md` STEP 6.
