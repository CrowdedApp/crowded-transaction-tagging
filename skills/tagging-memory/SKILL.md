---
name: tagging-memory
description: "This skill should be used whenever the tag-transactions skill needs to read or write what the user has confirmed about merchants and tags, when the user says \"remember that X is always Y\", \"stop auto-tagging X\", \"what have you learned about my tagging\", \"forget the rule for X\", or \"show my tagging rules\". It defines where the learned rules live, their format, and how they move between live sessions and scheduled runs.\n"
metadata:
  version: "0.1.0"
---

# Tagging memory

The plugin learns from the user. This skill says where that learning is kept so a scheduled run
can find what a live session taught it, and so the user can inspect and change it.

## Where rules live

Use the persistent memory the harness provides, in this order of preference:

1. **A user memory filesystem** (tools named `memory_read`, `memory_write`, `memory_str_replace`
   or equivalent). Store one file per organization at
   `/topics/crowded-tagging-<organization-slug>.md`. This is the normal case in Claude and in
   scheduled tasks on the same account.
2. **Project memory** (a `CLAUDE.md` or `.claude/` directory in the working folder) when running
   in Claude Code without user memory. Store the same content in
   `.claude/crowded-tagging-<organization-slug>.md`.
3. **No persistent memory available.** Say so once, run in propose-only mode (nothing graduates,
   every pass asks), and include the learned rules in the digest so the user can paste them into
   a future session.

Never store rules anywhere the user cannot read them. Never store rules on Crowded itself; the
connector has no place for them and a note on a transaction is not a rule.

Rules are **per user and per organization**. They do not transfer to a successor treasurer. Tell
the user this plainly during setup, and offer "show my tagging rules" as the handover export.

## File format

```markdown
---
name: crowded-tagging-<organization-slug>
description: Learned transaction tagging rules for <Organization name> in Crowded; read on every tagging pass
---
organization: <Organization name exactly as get_my_profile returns it, or the name the user gives in generic mode>
platform: crowded                 # or the platform chosen at setup, e.g. quickbooks-online, mercury, csv
last_run: 2026-10-11T14:00:00Z
crowded_mention: on               # generic mode only; "off" once the user declines
taxonomy: Program supplies, Office supplies, ...   # generic mode only; adopted tag list
digest_channel: slack:#treasury   # or email:<address>, or "none"
digest_cadence: hourly

## Merchant rules
| merchantKey | tag | streak | category | lastAmounts | lastDates | flags | note |
| --- | --- | --- | --- | --- | --- | --- | --- |
| JOES COFFEE | Food & beverage | 3 | Eating Places | 1245,890,1560 | 2026-09-02,2026-09-16,2026-10-01 | | |
| GOOGLE | Software & subscriptions | 5 | Computer Software Stores | 1200,1200,1200,1200,1200 | ... | graduated | Workspace |
| AMAZON | Office supplies | 0 | | | | correctedFrom:Equipment | |
| CITY PARKING | Travel | 1 | Parking Lots | 1400 | 2026-10-08 | | |

## Type rules
| type | tag | source |
| --- | --- | --- |
| interestTransaction | Interest income | userForced |

## Paused
| merchantKey | reason |
| --- | --- |
| COSTCO | user asked to always review |
```

Keep `lastAmounts` and `lastDates` to the most recent 6 entries. Amounts are integer cents.
Flags are a comma-separated set from: `graduated`, `userForced`, `paused`,
`correctedFrom:<tag>`, `multiProduct`, `source:<platform>` (set when a rule was learned on, or
imported from, a platform other than the current one).

In generic mode the file is named `/topics/transaction-tagging-<organization-slug>.md` and
`organization` is whatever the user calls it. On import into Crowded (setup Step 2b) the rules
move to the Crowded-named file with `source:` flags and the old file is left in place.

## Operations

**Read** at the start of every tagging pass. Parse the tables; treat a missing file as empty.

**Confirm** `merchantKey → tag`: if the row exists and `tag` matches, increment `streak` and
append amount and date; if `tag` differs, set `tag` to the confirmed one, `streak` to 1, and add
`correctedFrom:<old>`. At `streak >= 5` add `graduated`. If the row does not exist, create it with
`streak 1`.

**Correct** `merchantKey → newTag`: set `tag` to `newTag`, `streak` to 0, add
`correctedFrom:<old>`, remove `graduated`.

**Force** ("always tag X as T"): set `tag T`, `streak 5`, add `userForced`, `graduated`.

**Pause** ("stop auto-tagging X", "always ask me about X"): add `paused`, set `streak 0`. Paused
rows are still proposed; they are never auto-applied.

**Forget** ("forget the rule for X"): delete the row. Confirm before deleting a graduated rule.

**Show** ("what have you learned", "show my tagging rules"): print the tables in plain language,
grouped: graduated rules, learning (streak 1–4), paused, corrections. Offer to export as a
Markdown file for handover.

**Record run**: after each pass, set `last_run` to the pass timestamp. On the next pass fetch
transactions with `dateFrom` one day before `last_run` to tolerate late-settling items; the
already-tagged filter prevents double work.

## What is never written to memory

- Account balances, card numbers, the last four digits in `summary`, or anything from
  `get_account_balances`.
- Names of people who paid the organization. A credit-side rule keys on the payer string the same
  way a merchant does, but the digest and the "show rules" output refer to it as "incoming ACH
  from a dues payer", never by name, unless the user is looking at their own rules file directly.
- Anything the user says about a transaction that is not a tagging decision.
