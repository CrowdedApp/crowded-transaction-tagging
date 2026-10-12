---
name: setup-auto-tagging
description: "This skill should be used when the user installs the plugin, or asks to \"set up Crowded tagging\", \"set up auto-tagging\", \"set up transaction tagging\", \"start tagging my transactions automatically\", \"schedule the tagging pass\", \"change where my tagging digest goes\", \"turn off the tagging schedule\", \"import my tagging rules\", or asks how the plugin works. It first asks whether the organization uses Crowded; on Crowded it checks the connection and role, runs a first interactive pass, lets the user choose a digest channel, and creates the hourly scheduled task; otherwise it records the user's platform and hands off to generic-tagging.\n"
metadata:
  version: "0.1.0"
---

# Set up auto-tagging

Walk the user from "just installed" to "running on a schedule" in one conversation. Ask only
what cannot be inferred. Keep the user's language: "organization", never "chapter"; "tag",
never "code" or "GL account".

## Step 0: Which platform?

Ask one question before anything else: "Does your organization keep its money on Crowded?"

- **Yes** → continue to Step 1. Record `platform: crowded` in memory.
- **No** → show the platform list from `references/platforms.md`, grouped (accounting,
  banking and spend, donor platforms, spreadsheets, other), and let the user pick one or type
  their own. Record `platform: <name>` in memory. Then hand off to the `generic-tagging` skill:
  explain in two sentences that the plugin will propose and learn tags from their exports and
  hand back a tagged file, that it cannot write into that platform through Claude, and ask for
  their first export. Skip the rest of this skill; there is no connection check, schedule or
  digest in generic mode.
- **Not sure / both** → ask whether the account they want to tag is one they sign into at
  crowdedfinance.com. Yes means Crowded; otherwise generic.

If a Crowded connector is already present and responds to `get_my_profile`, skip the question
and continue to Step 1.

## Step 1: Check the connection

Call `get_my_profile` from the Crowded connector.

- If the connector is not available, tell the user to connect Crowded in their Claude connectors
  (the plugin lists it) and sign in with the Crowded login they use for the organization they
  want to tag. Stop here until it is connected.
- If `role` is not `Admin`, explain that tagging needs the Admin role in Crowded. Read Only
  members can see transactions but the connector will refuse to tag. Stop here.
- Read `organization` and `chapterName`. Tell the user which organization this connection is
  bound to, in one sentence, and that a Crowded connection covers one organization at a time: to
  tag a different one they reconnect in Crowded and run setup again.

## Step 2: Check memory

Follow the `tagging-memory` skill to find out which persistence is available. Tell the user in
one sentence where their learned rules will live and that the rules belong to them, not to the
organization: a successor treasurer starts fresh unless they are handed the "show my tagging
rules" export.

If no persistent memory is available, say the plugin will still work but will ask every time and
never graduate a rule, and continue.

## Step 2b: Import rules from another platform

If memory holds a rules file from a non-Crowded platform (`platform` other than `crowded`, or a
file the user pastes from "show my tagging rules" on another account), offer to import it:

1. Read the taxonomy and merchant rules from that file.
2. Compare the taxonomy to `list_transaction_tags` for this organization. Use existing tags where
   a near-duplicate exists; list the remainder and ask before creating them with
   `create_transaction_tags` (up to 20 per call).
3. Copy merchant rules into this organization's memory with their streaks intact but flagged
   `source: <old platform>`. On the first pass, treat a flagged rule as `streak 4` at most, so one
   Crowded-side confirmation graduates it; the merchant string may be spelled differently by a
   different card processor and the first match should be checked by a person.
4. Tell the user how many tags were matched, how many created, how many rules carried over.

## Step 3: First interactive pass

Run the `tag-transactions` skill in live mode over the last 30 days. This does three things at
once: it shows the user what the plugin does, it seeds the memory with their first
confirmations, and it surfaces near-duplicate tags in their existing taxonomy. If
`list_transaction_tags` returns obvious duplicates (`Food` and `Food & Beverage`, `Supplies` and
`supplies`), point them out and ask which one to keep going forward; the connector cannot merge
tags, so record the preferred one in memory and route future proposals to it.

Keep this pass short. Group proposals by merchant and ask about the five most frequent merchants
first; offer to continue or to stop and schedule.

## Step 4: Choose the digest channel

Ask one question, with the options the harness actually has connected. Typical options:

- A Slack channel or DM (if a Slack connector is present)
- An email address (if an email connector is present)
- No digest: the scheduled task tags what it can and the user opens Claude when they want to
  review

Say what the digest contains: counts, the items that need a decision with merchant and amount,
and a reminder of how many items still have no receipt attached. Say what it does not contain:
names of people who paid the organization, balances, card numbers. Ask whether the chosen
channel is one the organization is comfortable receiving spend summaries in; if it is a personal
inbox or a workspace the organization does not control, suggest counts-only mode (no merchant
names in the digest, just "3 need you, open Claude").

Record the choice in memory as `digest_channel` and whether counts-only mode is on.

## Step 5: Create the scheduled task

Use the scheduled-task capability the harness provides (in the Claude apps these are "scheduled
tasks"). Cadence: hourly, on the hour. Each run is a fresh session, so the prompt must stand
alone. Create it with this prompt, filling the bracketed values:

```
You are running the Crowded Transaction Tagging plugin on a schedule for [Organization name].

1. Load the tagging-memory skill and read the rules file for this organization.
2. Load the tag-transactions skill and run one unattended pass: fetch settled transactions since
   last_run minus one day, skip anything already tagged, apply the tagging policy, AUTO only
   what the policy permits, never create more than one new tag.
3. Update memory: last_run, any streak changes from rows you observed already tagged by a person.
4. Send the digest to [digest_channel] in [full | counts-only] mode. If there is nothing new and
   nothing pending, send nothing.
5. If the Crowded connector is unavailable or returns a permission error, send one line to the
   digest channel saying tagging paused and why, then stop. Do not retry within the run.

Never tag rows that already have tags. Never remove tags. Never include payer names, balances or
card numbers in the digest.
```

Tell the user the task exists, when it first fires, and that they can pause it by asking
"turn off the tagging schedule". If the harness has no scheduled-task capability, say so and
offer the manual alternative: run "tag my transactions" whenever they open Claude.

## Step 6: Confirm

Summarize in four lines: organization bound, where rules live, digest channel and mode, schedule
status. Then stop.

## Changing settings later

- "Change where my digest goes" → Step 4 only, update memory, update the scheduled task prompt.
- "Turn off the tagging schedule" → disable the scheduled task; memory is kept.
- "Turn it back on" → re-enable or recreate from Step 5.
- "Switch organizations" → explain reconnecting Crowded to the other organization, then rerun
  Step 1. The old organization's rules stay in their own file.
