# Outbound Research Agent: Grok Rule Pack

A rule pack that turns an autonomous browser agent (xAI **Grok Bot**) into a careful outbound lead researcher. You give it a spreadsheet of companies (name and website). For each company it researches the site, decides with fixed rules whether the company qualifies, and writes a cold-email draft and a LinkedIn-message draft for the ones that do.

**It never sends anything.** Every draft goes to a file for a human to review.

## The core idea

The executor can't ask follow-up questions, so every rule must be followable without judgement:

- no vague words ("relevant", "good fit"): only yes/no checks or comparisons to stated numbers or lists
- every fact names the exact page it comes from, with a defined fallback when it isn't found
- the executor may not rewrite its own instructions, and may never enter a compose-and-send flow
- anything not covered makes it stop that company and log `RULE_GAP` instead of guessing

## Layout

```
pack/            the rules, read in numeric order
  00-mission.md    the per-company loop and where the input list comes from
  01-research.md   what to read on each site
  02-qualify.md    qualify / disqualify rules
  03-email.md      cold-email draft rules
  04-linkedin.md   LinkedIn draft rules
  05-output.md     exact file names and schemas under out/
  06-guardrails.md overrides everything else
GROK_BRIEF.md    the whole pack merged into one brief for the executor
inputs/          companies.csv, suppression.csv, offer.md, source.md
fixtures/        18 fake company websites for testing the pack offline
out/             where the executor writes drafts and logs
```

## Usage

1. Fill in `inputs/offer.md` and put your list in `inputs/companies.csv` (or set a published CSV URL in `inputs/source.md`).
2. Give `GROK_BRIEF.md` to Grok Bot as its full instruction set. **Don't connect any tool that can send email or messages.**
3. Review the drafts in `out/` before anything goes out.

