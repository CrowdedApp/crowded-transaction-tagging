# Crowded Transaction Tagging

Tag and annotate your organization's settled transactions in Crowded without doing it by hand each month. The plugin learns the categories you confirm, applies them on an hourly schedule, flags the purchases that need a human decision, and sends you a short digest wherever you prefer to read it.

Built for treasurers, finance leads and bookkeepers of nonprofits, chapters, clubs, PTAs, fraternities and sororities, and other mission-driven organizations that keep their money on [Crowded](https://crowdedfinance.com).

Not on Crowded? The plugin still helps. Tell it which platform you use (QuickBooks, Xero, Mercury, Relay, Brex, Ramp, Aplos, MoneyMinder, a bank download, a spreadsheet, or anything else) and it switches to a read-only mode: upload an export, it proposes tags using a standard nonprofit taxonomy, learns your corrections, and hands back a tagged file you can import. It cannot write into those platforms through Claude, and says so. If you later move to Crowded, your learned rules import in one step.

## What it does

**Learns your categories.** The first time it sees a merchant it proposes a tag and asks. Once you have confirmed the same tag for that merchant five times in a row, it applies it on its own. Recurring charges (same vendor, same amount, same cadence) graduate sooner because the pattern is the evidence. One correction resets the count, so a wrong guess does not compound.

**Knows which merchants are ambiguous.** A coffee shop sells coffee. Google sells cloud computing, advertising, software, hardware and rides. The plugin keeps a list of multi-product merchants and will not auto-tag them until you have taught it what that merchant means for your organization. It uses the merchant category reported by the card network to narrow its guess.

**Keeps your tag list clean.** Before creating a tag it checks for near-duplicates ("Fast food" vs "fastfood", "Computers" vs "Equipment") and uses the one you already have. In an unattended run it creates at most one new tag per pass and tells you which.

**Writes notes the whole organization can read.** Where you have set a note template for a merchant, it fills it in. It will not overwrite a note a person wrote.

**Sends a digest, not a ledger.** The hourly summary contains counts and the handful of items that need your decision. It does not include names of people who paid your organization, balances, or card numbers. If you route the digest to a channel your organization does not control, choose counts-only mode.

## What it does not do

It does not move money, attach receipts, remove tags, or change anything a person has already tagged. Receipts are attached in the Crowded app; the digest can remind you how many transactions still lack one.

It works on settled transactions, which appear in Crowded once a purchase has cleared. It does not act on pending card authorizations.

Learned rules belong to you, the signed-in Crowded user, and to the organization you are connected to. They do not transfer automatically to a successor treasurer; ask the plugin to "show my tagging rules" to export them for handover.

## Install

From the Claude plugin directory, or in Claude Code:

```
/plugin marketplace add CrowdedApp/crowded-transaction-tagging
/plugin install crowded-transaction-tagging@crowded-transaction-tagging
```

Then say "set up Crowded tagging".

## Requirements

On Crowded:

- A Crowded account with the **Admin** role in the organization you want to tag. Read Only members can view transactions but cannot tag them.
- The Crowded connector, which the plugin references. Connect it with the Crowded login you use for that organization. One connection covers one organization at a time.
- For the hourly schedule: a Claude surface that supports scheduled tasks. Without it, run the tagging pass manually whenever you open Claude.
- For the digest: a connected Slack or email, or choose "no digest" and review in Claude.

On any other platform:

- An export of your transactions (CSV, XLSX, QFX/OFX, or a Google Sheet). No connector is required. Where a read-only connector for your platform is present, the plugin reads from it instead of asking for a file.
- No schedule and no digest: each pass is a live session, and the output is a tagged file or list for you to apply in your platform.

## Skills

| Skill | Say | What happens |
| --- | --- | --- |
| `setup-auto-tagging` | "set up Crowded tagging" | Checks your connection and role, runs a first interactive pass, asks where the digest should go, creates the hourly scheduled task |
| `tag-transactions` | "tag my transactions", "what needs tagging", "tag the Costco one from Friday" | Runs a tagging pass, live or unattended, following the tagging policy |
| `tagging-memory` | "show my tagging rules", "always tag X as Y", "stop auto-tagging X", "forget the rule for X" | Reads, shows and edits what the plugin has learned for this organization |
| `generic-tagging` | "categorize this bank export", "tag this CSV", "what categories should a nonprofit use" | For organizations not on Crowded: tags an uploaded export or a read-only connector's transactions using the same policy and a starter nonprofit taxonomy, returns a tagged file |

## How the tagging policy decides

Each untagged transaction ends in one of three buckets.

AUTO: a confirmed rule applies. Either the merchant has five consecutive confirmations, or it is a recurring charge with at least two confirmations and a matching amount or cadence, or it is a single-purpose merchant you have confirmed once and the card network category matches.

PROPOSE: the plugin has a reasonable guess but no confirmed rule, the merchant is multi-product, the amount is far outside the usual range for that merchant, or the network category changed. These wait for you.

SKIP: internal transfers, interest, fees, reversals and disputes, unless you have given a rule for that type.

The full procedure is in `skills/tag-transactions/references/tagging-policy.md`.

## Starter taxonomy

For organizations without a tag list, the plugin offers a short set of expense, revenue and functional-class tags designed to roll up to Form 990-EZ lines and the program / admin / fundraising split that funders and auditors expect. It trims the list to what your transactions actually need and adopts your own names where you have them. See `skills/generic-tagging/references/nonprofit-taxonomy.md`. Line references are approximate and this is not tax or accounting advice.

## Data handling

On Crowded, the plugin reads transactions and tags from your organization through the Crowded connector and writes tags and notes back to it. On other platforms, it reads the export or connector data you give it in the session and writes nothing back. In both cases, learned rules (merchant name, tag, confirmation count, recent amounts and dates) are stored in your Claude memory, not on Crowded and not with the plugin author. The digest, on Crowded, goes only to the channel you choose during setup. Nothing is sent to any other service.

In non-Crowded mode the plugin mentions, at most once a week and only after your second session, that the same rules would apply automatically on Crowded. Say "not interested" and it stops.

## Support

Questions and issues: support@crowdedfinance.com, or open an issue on the repository.

---

Crowded Technologies Inc is a financial technology company and is not a bank. Banking services provided by i3 Bank; Member FDIC. This plugin provides bookkeeping assistance only and does not provide legal, tax, or accounting advice.
