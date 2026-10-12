---
name: tag-transactions
description: "This skill should be used when the user asks to \"tag my transactions\", \"categorize transactions\", \"tag new transactions in Crowded\", \"run the tagging pass\", \"what needs tagging\", \"clean up transaction tags\", \"add a note to this transaction\", or when a scheduled tagging task runs. It reads settled transactions from the connected Crowded organization, applies tags the user has previously confirmed, proposes tags for the rest, writes notes, and reports what it did. Requires the Crowded connector and the Admin role in the organization.\n"
metadata:
  version: "0.1.0"
---

# Tag transactions

Apply bookkeeping tags and notes to settled transactions in the connected Crowded organization.
Automate only what the user has already confirmed; ask about everything else. Consistency matters
more than coverage: one tag per category, no near-duplicates, no silent guesses on ambiguous
merchants.

Read `references/tagging-policy.md` before the first pass in a session. It is the full decision
procedure; this file is the workflow.

## Preconditions

1. Call `get_my_profile`. Confirm `role` is `Admin`. If not, stop and tell the user tagging needs
   the Admin role in Crowded; Read Only members can view but not tag.
2. Note `chapterName` and `organization`. Use the organization name in everything the user sees.
   Never say "chapter" to the user; the connector uses that word internally, the user's word is
   "organization".
3. Load the tagging memory for this organization (see the `tagging-memory` skill). Key it by the
   organization name from the profile. If there is no memory yet, this is a first run: propose
   everything, automate nothing.

## Workflow

### 1. Fetch what is new

- Call `list_transaction_tags` with `limit: 50` and page until `hasMore` is false. Keep the full
  list of `{id, name, transactionCount}`. This is the organization's current taxonomy.
- Call `list_chapter_transactions` with `dateFrom` set to the last run date from memory (or the
  past 30 days on a first run), `limit: 50`, and page until `hasMore` is false.
- Skip rows that already have one or more `tags`. Those were handled by a person or an earlier
  pass; never overwrite them.
- Skip rows whose `type` is `bookTransaction`, `interestTransaction`, `interestShareTransaction`,
  `feeTransaction`, `feeReversalTransaction`, or `rewardTransaction` unless the user has a
  confirmed rule for that type. Internal movements and platform items are not spend categories.

### 2. Classify each untagged row

For each row, build the merchant key from `summary` (the merchant string before the first `|`),
normalized: uppercase, strip card-processor prefixes (`SQ *`, `TST*`, `PAYPAL *`, `AMZN Mktp`,
`PP*`, `GOOGLE *`), strip trailing store numbers and reference codes. Keep the `category` field
(the card network's merchant category) alongside it.

Then apply the decision procedure in `references/tagging-policy.md`. Every row ends in exactly one
of three buckets:

| Bucket | Meaning | Action |
| --- | --- | --- |
| AUTO | A confirmed rule applies with enough confidence | Tag it now, write a note if the rule has one |
| PROPOSE | Claude has a good guess but no confirmed rule, or the merchant is multi-product | Hold; show the user the row, the proposed tag, and why |
| SKIP | Not a spend category, or no sensible guess | Leave untouched; list it under "needs you" |

### 3. Resolve tag names before creating anything

For every tag name you intend to apply, AUTO or PROPOSE:

1. Look for an exact match in the taxonomy (case-insensitive).
2. If none, look for a near-duplicate: same letters ignoring case, spaces, hyphens and plurals
   (`fastfood` = `Fast food` = `Fast-Food`), or an existing tag that plainly covers the same
   category (`Computers` vs `Equipment`, `Meals` vs `Food & beverage`).
3. If a near-duplicate exists, use the existing tag. Do not create a new one. In a live session,
   tell the user: "You already have *Equipment*; using that instead of creating *Computers*."
   In an unattended run, use the existing tag and say so in the digest.
4. Only if nothing matches, create the tag with `create_transaction_tags`. Use sentence case,
   singular where natural, no trailing punctuation. Creating a tag is itself a taxonomy decision,
   so in an unattended run create at most one new tag per pass and name it in the digest.

### 4. Apply

- Batch AUTO rows per tag and call `tag_transactions` once per tag with up to 100 transaction ids.
- For rows whose rule includes a note template, call `update_transaction_note`. Notes are visible
  to every Admin and Read Only member and appear in exports, so write them for the whole
  organization, under 500 characters, and never include anything the user said in confidence.
  Never overwrite a non-null existing note.
- Do not touch PROPOSE or SKIP rows.

### 5. Report

In a live session, show a compact table: merchant, amount, date, bucket, tag, reason. Then ask the
user to confirm or correct the PROPOSE rows one group at a time, grouped by merchant key.

In an unattended run (scheduled task), send the digest the user chose during setup. Keep it to
counts and the items that need a decision. Example shape:

> Crowded tagging, Tuesday 10:00: 9 new transactions. Auto-tagged 6. 3 need you:
> AMAZON $212.40 (Supplies? Equipment?), GOOGLE $48.00 (Software? Advertising?), new merchant
> CITY PARKING $14.00. Reply here or open Claude to confirm.

Do not list every auto-tagged row in the digest. Do not include counterparty names from
incoming payments (`receivedAchTransaction`, `checkDepositTransaction`) in the digest; those are
people, and the digest may land in a shared channel.

### 6. Learn

After the user confirms or corrects, update the tagging memory per the `tagging-memory` skill:
increment the confirmation streak for a confirmed merchant key, reset it to zero on a correction,
record the corrected tag as the new candidate. Record the run timestamp as the last run date.

## When the user asks for one transaction

If the user names a specific transaction ("tag the Costco one from Friday", "add a note to the
$212 Amazon charge"), find it with `list_chapter_transactions` using `search`, `dateFrom`,
`dateTo` or `amountMin`/`amountMax`, confirm you have the right row by echoing merchant, amount
and date, then tag or annotate it and update memory as a confirmation.

## What this skill never does

- Never removes a tag. The connector cannot, and the policy would not.
- Never tags a row that already carries tags.
- Never creates a tag when a near-duplicate exists.
- Never auto-tags a multi-product merchant without a graduated rule.
- Never writes account balances, card numbers or people's names into notes or digests.
- Never claims a transaction is "done" with receipts. Receipts are attached in the Crowded app;
  the digest may remind the user that untagged or unreceipted items exist, using `hasReceipts`
  as a count only.
