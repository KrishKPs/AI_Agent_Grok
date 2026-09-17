# 06 — GUARDRAILS

Hard limits. These override every other file in the pack. When a guardrail conflicts
with any other rule, the guardrail wins and the other rule is a defect.

---

## 1. The final gate

Run this before writing any draft file. All six must be true. If any is false, do not
write the draft — halt the company and log it.

| # | Check | If false |
|---|---|---|
| 1 | Every factual claim in the draft traces to a field in `out/<slug>/research.json` | Delete the claim, then re-run the gate |
| 2 | The research file exists and was written before the qualify decision | `RULE_GAP` |
| 3 | The qualify decision is recorded with a reason naming the tests it passed | `RULE_GAP` |
| 4 | No field used in the draft holds the value `NOT_FOUND` | `INSUFFICIENT_DATA` |
| 5 | The draft header reads `status: DRAFT — NOT SENT` | Fix the header, then re-run |
| 6 | Nothing has been sent, and no send control has been clicked | Halt everything — see section 4 |

---

## 2. Never

- **Never send.** No Send, Connect, Add Note, Submit, Reply, Follow, Request, InMail, or
  contact-form submit. No control whose effect is to transmit a message to a person.
- **Never log in** to email, to LinkedIn, or to any system, for any purpose. This
  includes the company-list fetch in `00-mission.md` section 1: a published CSV URL that
  answers with a sign-in page halts the run. It is never a prompt to authenticate.
- **Never bypass** a login wall, a paywall, a captcha, or a robots restriction.
- **Never create** an account, a profile, or a session anywhere.
- **Never invent a fact** about a company or a person.
- **Never invent a rule.** A gap is logged, never filled.
- **Never guess or construct an email address** from a name pattern or a domain.
- **Never collect personal information.** Business role only: name, job title, employer,
  public professional profile URL. No personal email, no phone, no home location, no
  personal detail about the individual.
- **Never write outside `out/`.**
- **Never set `reviewed` to `yes`.** That column belongs to the human.
- **Never delete** a file in `out/`.
- **Never process two companies at once.**

---

## 3. Always

- Work one company at a time, in input row order, to completion or to a halt.
- Write the research file before the qualify decision.
- Record a decision and a reason for every company, including disqualified ones.
- Record a source URL for every captured field.
- Respect the budget in `01-research.md`.
- Prefer halting over guessing, every time.

---

## 4. If a send happens

If a message is transmitted for any reason, including by accident or by a page acting on
its own:

```
STOP all processing immediately.
Do NOT continue to the next company.
Write out/INCIDENT.md recording: what was sent, to whom, from which page,
  at what time, and the last action taken before it happened.
Halt the run.
```

A human decides what happens next. Do not attempt to recall, delete, or correct the
message — that is another transmitted action.

---

## 5. Precedence

When two rules conflict:

```
1. This file (06-guardrails.md)
2. 00-mission.md
3. The numbered rule file governing the current step
4. Nothing else — there is no fallback, no default, and no common sense layer.
```

If a conflict is not resolved by that order, it is a `RULE_GAP`. Halt and log it.

---

## 6. The standing test

Before any action, ask: **is this action fully specified by a rule in this pack?**

```
IF   yes    THEN take the action.
ELSE        halt the company, log RULE_GAP, move to the next.
```

There is no third branch. "Close enough to a rule" is not a rule. An action that seems
obviously correct but is written down nowhere is exactly the action this pack exists to
prevent.
