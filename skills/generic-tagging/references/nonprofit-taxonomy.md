# Best-practice tagging taxonomy for small nonprofits

A starting set of tags, built to roll up cleanly into the IRS Form 990 / 990-EZ expense and
revenue lines and the three functional classes auditors and funders expect (program,
management & general, fundraising). Offer it in generic mode when the user has no taxonomy of
their own; adopt the user's own names when they have them.

Principles the plugin enforces, whichever names are used:

1. One tag per category. Merge spelling variants; do not keep both "Supplies" and "supplies".
2. Tag the purpose, not the merchant. "Amazon" is not a category; "Office supplies" is.
3. Keep the list short enough to remember. Twenty to thirty tags covers most organizations under
   $1M. More than fifty is a sign the tags are doing the job of a general ledger.
4. Revenue and expense tags do not mix. A credit gets a revenue tag; a debit gets an expense tag.
5. Internal movements (transfers between the organization's own accounts, card payments from
   checking) get a transfer tag or no tag, never a spend tag; otherwise expenses double-count.
6. Reimbursements to volunteers take the tag of what was bought, with a note naming the
   reimbursed purpose, not a "Reimbursement" tag; the latter hides where the money went.
7. Where a functional class matters (program vs admin vs fundraising), use a second tag for the
   class rather than multiplying the first list by three.

## Expense tags

| Tag | Use for | 990-EZ line (approx.) |
| --- | --- | --- |
| Program supplies | Materials consumed delivering the mission: craft supplies, uniforms, kits, food for participants | Part I line 16 (other) / Part III |
| Program events | Venue, activity and participant costs for mission events (not fundraisers) | Part III |
| Office supplies | Paper, toner, small desk items | line 16 |
| Software & subscriptions | SaaS, domains, hosting, email tools, Zoom, Canva | line 16 |
| Equipment | Durable items: laptops, projectors, furniture, tools (consider a capitalization threshold) | line 16 / Schedule |
| Printing & copying | Print shops, flyers, banners | line 15 |
| Postage & shipping | USPS, UPS, FedEx, courier | line 15 |
| Advertising & promotion | Paid ads, sponsored posts, promotional items | line 16 |
| Travel | Flights, lodging, rideshare, parking, mileage, tolls | line 16 |
| Meals & hospitality | Meals during meetings or travel, volunteer thank-yous, refreshments | line 16 |
| Training & conferences | Registration fees, courses, certifications | line 16 |
| Dues & affiliations | Payments to a parent organization, memberships in associations | line 16 |
| Insurance | Liability, D&O, event insurance | line 16 |
| Professional fees | Accountant, bookkeeper, lawyer, consultant, tax preparer | line 13 |
| Bank & payment fees | Processor fees, wire fees, card fees | line 16 |
| Rent & facilities | Rent, storage units, room hire | line 14 |
| Utilities & telecom | Phone, internet, electricity | line 14 |
| Maintenance & repairs | Facility and equipment upkeep | line 14 |
| Licenses, permits & taxes | State filings, permits, sales tax, government fees | line 16 |
| Gifts & recognition | Awards, flowers, recognition gifts | line 16 |
| Grants & donations made | Scholarships, grants to other organizations, donations | lines 10–11 |
| Payroll & contractors | Wages, stipends, contractor payments | lines 12–13 |
| Fundraising expenses | Direct costs of fundraising events and campaigns | Part I line 6b |
| Transfers out | Movement to the organization's own other accounts | not an expense |

## Revenue tags

| Tag | Use for | 990-EZ line (approx.) |
| --- | --- | --- |
| Donations | Individual gifts, online giving platform payouts | line 1 |
| Grants received | Foundation, corporate or government grants | line 1 |
| Membership dues | Dues from members | line 3 |
| Program service revenue | Fees for the mission activity: tuition, tickets to program events, registrations | line 2 |
| Fundraising event income | Gross receipts from fundraisers | line 6a |
| Sponsorships | Corporate sponsorships | line 1 or 6a, by terms |
| Merchandise sales | T-shirts, products | line 7a |
| Interest income | Interest on deposits | line 4 |
| Reimbursements received | Money back from vendors or members | offset, not revenue |
| Transfers in | Movement from the organization's own other accounts | not revenue |

## Functional class tags (optional second tag)

| Tag | Meaning |
| --- | --- |
| Program | Spending that delivers the mission |
| Admin | Management and general: governance, accounting, insurance, filings |
| Fundraising | Cost of raising money |

Rule of thumb for small organizations: tag the functional class only on items above a threshold
the treasurer picks (for example $250), and let small items default to Program. Funders look at
the ratio; auditors look at the method being consistent.

## Mapping from card-network merchant categories

The same map used in the Crowded path (`tag-transactions/references/tagging-policy.md`) applies
when a CSV carries an MCC or category column. When it does not, infer from the merchant string
and say so in the reason.

## Disclaimer to include once per session in generic mode

This taxonomy is a bookkeeping convenience and a common starting point. It is not tax or
accounting advice. Line references are approximate and change; confirm with the organization's
preparer.
