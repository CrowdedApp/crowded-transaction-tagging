# Tagging policy

The decision procedure for every untagged, settled transaction. Apply the rules in order; the first
rule that fires decides the bucket. The buckets are AUTO (tag now), PROPOSE (ask), SKIP (leave).

## Inputs per row

- `merchantKey`: normalized merchant string (see SKILL.md step 2)
- `category`: the card network merchant category string on the row, e.g. "Eating Places",
  "Computer Software Stores", "Advertising Services", "Taxicabs and Limousines"
- `type`: the transaction type, e.g. `purchaseTransaction`, `originatedAchTransaction`
- `direction`: `Debit` (money out) or `Credit` (money in)
- `signedAmount`: integer cents, negative for money out
- `createdAt`: ISO timestamp
- Memory for `merchantKey` in this organization, if any: `{tag, streak, lastAmounts[], lastDates[], noteTemplate?}`

## Rule 0: not a spend category

If `type` is one of `bookTransaction`, `interestTransaction`, `interestShareTransaction`,
`feeTransaction`, `feeReversalTransaction`, `rewardTransaction`, `releaseTransaction`,
`paymentCanceledTransaction`, `cardReversalTransaction`, `chargebackTransaction`,
`disputeTransaction`, or any `returned*` / `dishonored*` type:

- If memory has a confirmed rule for that exact `type` in this organization (e.g. the user said
  "always tag interest as Interest income"), bucket AUTO.
- Otherwise bucket SKIP. List under "needs you" only if `direction` is `Debit`.

## Rule 1: graduated merchant rule

If memory has `merchantKey` with `streak >= 5` and no correction since:

- Bucket AUTO with the remembered tag. Apply the note template if one exists.

A streak counts consecutive confirmations of the same tag for the same `merchantKey` in the same
organization. Any correction resets the streak to 0 and replaces the candidate tag with the
corrected one. Confirmations of a *proposed* tag count; so does the user tagging the merchant
themselves in the Crowded app when a later pass observes it already tagged with the candidate.

## Rule 2: recurring charge

If memory has `merchantKey` with `streak >= 2`, and the current row matches the pattern:

- same `signedAmount` as at least two prior confirmed rows for this merchant (exact match in
  cents), **or**
- `signedAmount` within 2% of the prior amount **and** `createdAt` falls within 3 days of the
  same day-of-month or same weekday cadence as the prior two rows

then bucket AUTO with the remembered tag. Recurring subscriptions, rent, payroll providers and
dues to a parent organization are the same category every time; the amount and cadence are the
evidence. This rule does not require five confirmations because the pattern itself is the
confidence.

## Rule 3: multi-product merchant

A merchant is multi-product when any of these hold:

- `merchantKey` is in the built-in list below, **or**
- the same `merchantKey` has appeared in this organization with two or more different `category`
  values, **or**
- the user has confirmed two or more different tags for this `merchantKey` in the past

Built-in multi-product merchants (Claude maintains this list from general knowledge; extend it
when evidence appears): AMAZON, AMZN, GOOGLE, APPLE, MICROSOFT, WALMART, TARGET, COSTCO,
SAMS CLUB, PAYPAL, VENMO, SQUARE / SQ *, STRIPE, EBAY, ETSY, BEST BUY, HOME DEPOT, LOWES,
STAPLES, OFFICE DEPOT, CVS, WALGREENS, UBER, LYFT, DOORDASH, GRUBHUB, INSTACART, SHOPIFY, META,
FACEBOOK, LINKEDIN, X / TWITTER, TIKTOK, GODADDY, NAMECHEAP, ZOOM, CANVA, ADOBE.

For a multi-product merchant that has not graduated under Rule 1 or matched Rule 2:

- Bucket PROPOSE. Use `category` to narrow the guess ("Computer Software Stores" at GOOGLE
  suggests Software; "Advertising Services" suggests Advertising) and say which `category` the
  network reported. Offer the two or three most plausible existing tags.

Never AUTO a multi-product merchant on name alone, however many times it has appeared, until the
streak reaches 5 for a single tag.

## Rule 4: single-purpose merchant with a confirmed rule

If memory has `merchantKey` with `streak >= 1` and the merchant is not multi-product:

- If `category` on this row matches the `category` recorded with the rule, bucket AUTO.
- If `category` differs, bucket PROPOSE and point out the change ("JOES COFFEE usually reports
  Eating Places; this one is Grocery Stores").

A single confirmation is enough for a single-purpose merchant because there is only one thing
the merchant sells.

## Rule 5: single-purpose merchant, no rule yet

If the merchant is not multi-product and memory has nothing:

- Map `category` to the organization's taxonomy using the table in the next section. If the
  mapped tag exists in the taxonomy, bucket PROPOSE with that tag and reason "merchant category:
  <category>". If no tag maps, bucket PROPOSE with a suggested new tag name, clearly marked as
  new.

Do not AUTO on `category` alone. The network category is a strong hint and a weak commitment;
"Eating Places" can be a board dinner, a volunteer thank-you or a fundraiser meal, and some
organizations tag those differently.

## Rule 6: amount outliers

Before finalizing any AUTO under Rules 1, 2 or 4: if `|signedAmount|` is more than 5x the
median of the remembered `lastAmounts` for that merchant, **and** the merchant is multi-product or
the amount exceeds $500.00 (50000 cents), downgrade to PROPOSE with reason "amount outlier".
A $900 laptop at a merchant that usually sells $30 of printer paper is not Supplies.

## Rule 7: incoming money

For `direction: Credit` rows (`receivedAchTransaction`, `checkDepositTransaction`,
`cashDepositTransaction`, incoming `cardTransaction`):

- Apply Rules 1, 2 and 4 by `merchantKey` (here the payer or source string).
- Otherwise bucket PROPOSE with revenue-side tags only (dues, donations, program revenue,
  grants, reimbursements, transfers in). Never propose an expense tag for a credit.
- Never put the payer's name into a digest or a note. Refer to it in the digest by type and
  amount only ("1 incoming ACH $250.00 needs a tag").

## Category to tag starter map

Use this only to *propose*. Prefer an existing tag in the organization's taxonomy over the name
in this table; the table gives the concept, the taxonomy gives the spelling.

| Network category contains | Proposed concept |
| --- | --- |
| Eating Places, Fast Food, Restaurants, Caterers, Bakeries | Food & beverage (or Meals, Catering if the taxonomy has them) |
| Grocery Stores, Supermarkets | Food & beverage or Supplies, ask |
| Computer Software Stores, Digital Goods, Information Services | Software & subscriptions |
| Advertising Services, Direct Marketing | Advertising & promotion |
| Office Supplies, Stationery | Office supplies |
| Hardware Stores, Home Supply, Building Materials | Maintenance & repairs or Equipment, ask |
| Electronics Stores, Computers & Peripherals | Equipment |
| Taxicabs, Limousines, Passenger Railways, Airlines, Lodging, Hotels, Car Rental, Parking | Travel |
| Service Stations, Fuel | Travel or Vehicle, ask |
| Postal Services, Courier, Freight | Postage & shipping |
| Telecommunication, Cable, Utilities, Electric, Gas, Water | Utilities |
| Insurance | Insurance |
| Professional Services, Legal, Accounting, Consulting | Professional fees |
| Charitable Organizations, Religious Organizations, Membership Organizations | Dues & affiliations (outgoing) or Donations made |
| Schools, Educational Services, Conferences | Training & conferences |
| Printing, Quick Copy, Commercial Photography | Printing |
| Gift, Card, Novelty, Florists | Gifts & recognition |
| Sporting Goods, Recreation, Amusement | Program expenses |
| Money Orders, Wire Transfer, Financial Institutions | Transfers (SKIP unless the user has a rule) |
| Government Services, Court Costs, Fines, Tax Payments | Fees, licenses & taxes |
| Medical, Dental, Pharmacies | Program expenses or Supplies, ask |

## Streak accounting

| Event | Effect on `merchantKey` memory |
| --- | --- |
| User confirms proposed tag T | if candidate == T: streak += 1; else candidate = T, streak = 1 |
| User corrects to tag U | candidate = U, streak = 0; record `correctedFrom: T` |
| Later pass sees merchant already tagged T by a person in the app | treat as confirmation of T |
| User says "always tag X as T" | candidate = T, streak = 5, `userForced: true` |
| User says "stop auto-tagging X" | streak = 0, `paused: true` (Rule 1, 2, 4 do not fire) |
| Five consecutive confirmations | graduated; Rule 1 applies |

Streaks are per organization. A treasurer connected to a different organization starts from that
organization's memory, not this one's.

## Things that are never automated

- Removing or replacing an existing tag.
- Tagging rows that already carry tags.
- Creating more than one new tag in an unattended pass.
- Any row where the rule's recorded `category` and the row's `category` disagree, except under
  Rule 2 (recurring amount) where the amount is the evidence.
- Any row the user has explicitly asked to be reminded about.
