# Platform picker

Present this list when the user says they do not use Crowded. Group it; let the user pick one or
type their own. Record the choice in memory as `platform`. The right-hand column tells the
generic-tagging skill how transactions will arrive and whether a connector may exist.

## Accounting and bookkeeping

| Platform | Typical export | Connector note |
| --- | --- | --- |
| QuickBooks Online | Bank feed → "Export to Excel" from Banking; or Transaction Detail by Account report as CSV | A QuickBooks connector may exist but exposes reports, not bank-feed categorization; treat as read-only |
| QuickBooks Desktop | Bank register → Excel | No connector; CSV only |
| Xero | Accounting → Bank accounts → Export statement CSV | CSV only |
| Sage Intacct | GL detail export | CSV only |
| NetSuite | Transaction search → CSV | CSV only |
| Aplos | Transactions → Export | CSV only |
| MoneyMinder | Bank register export | CSV only |
| Blackbaud Financial Edge NXT | Journal entry / bank register export | CSV only |
| Wave | Transactions → Export | CSV only |
| FreshBooks | Expenses → Export | CSV only |
| Zoho Books | Banking → Export | CSV only |

## Banking and spend platforms

| Platform | Typical export | Connector note |
| --- | --- | --- |
| Mercury | Transactions → Export CSV | If a Mercury connector is present, read transactions from it; no write-back |
| Relay | Transactions → Export | CSV only |
| Brex | Transactions → Export | If a connector is present, read-only |
| Ramp | Transactions → Export | If a connector is present, read-only |
| Bill.com (Spend & Expense / Divvy) | Transactions → Export | CSV only |
| Novo, Bluevine, Found, Lili | Transactions → Export CSV | CSV only |
| Chase, Bank of America, Wells Fargo, US Bank, PNC, Truist, Capital One, any other bank | Online banking → Download activity (CSV, QFX, OFX) | CSV only; QFX/OFX are read as CSV after the user converts or the plugin parses the simple cases |
| Credit union | Online banking → Download | CSV only |

## Donor and program platforms with a ledger

| Platform | Typical export | Connector note |
| --- | --- | --- |
| Givebutter, Zeffy, Donorbox, Bloomerang, Neon One, Little Green Light | Payouts / transactions export | CSV only; these are revenue-side, tag with revenue concepts |
| Stripe, PayPal, Square | Payouts / balance transactions export | CSV only; revenue-side with fees to split out |

## Spreadsheets

| Platform | Typical export | Connector note |
| --- | --- | --- |
| Google Sheets | Share the sheet or export CSV | If a Sheets connector is present, read the sheet directly and write a tag column back |
| Excel / Numbers | Save as CSV or upload the workbook | Upload |

## Other

Let the user type the name. Treat it as CSV only. Add the name to memory so the next setup can
offer it.
