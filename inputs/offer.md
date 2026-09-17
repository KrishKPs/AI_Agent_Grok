# offer.md — everything the drafts may claim about US

> **PROTOTYPE DEMO DATA.** Coastal Cart Components is a fictional company invented for
> a pipeline demonstration. Every string below is example copy, not a real business
> claim. Replace this whole file before any live use.

Every string below is copied **verbatim** into the drafts. Grok Bot never rewords,
shortens, expands, or merges them.

Any field still reading `FILL_ME` halts the company with `RULE_GAP`.

---

```yaml
sender_name:       Alex Rivera
sender_title:      Sales Lead
sender_company:    Coastal Cart Components

# Max 25 words. What we sell. (16 words)
offer_sentence:    Coastal Cart Components supplies and assembles replacement parts for golf carts, NEVs, and street-legal low-speed vehicles.

# Max 20 words, or the literal NOT_SUPPLIED to omit it entirely.
proof_line:        NOT_SUPPLIED

# Max 15 words. ONE ask. The only place a question mark may appear. (14 words)
cta_sentence:      Would it help if I sent over our fitment list for your most-serviced models?

# Used when no contact name was found.
fallback_greeting: Hi there

# Max 20 words. Used when NO personalization field was captured. (14 words)
generic_opener:    I work with dealers and service shops that keep low-speed vehicles on the road.

# Max 4 lines.
signature_block:   |
  Alex Rivera
  Sales Lead, Coastal Cart Components
  alex.rivera@coastalcartcomponents.example
```

---

## Caps are enforced

If a string here exceeds its word cap, the draft blows its length budget and the company
halts with `RULE_GAP` — the executor will not trim your words to fit.

## What must not go in here

- A statistic, percentage, price, discount, or lead time you cannot substantiate. These
  strings are the **only** place numbers may enter a draft.
- A claim about the recipient's business. This file describes us only.
- A deadline or scarcity claim.
