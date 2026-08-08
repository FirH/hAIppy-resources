# Customer Membership Tiers — Dataset Context

Used in Section 8 of `correlation_example_powerpoint_data.ipynb` ("Ordinal categories — a low Pearson
with no outliers at all").

## The scenario

A subscription business — think a SaaS product or an e-commerce membership programme — with **60
customers spread across six membership plans**. Each row is one customer, described by what they pay,
how much support they consume, how happy they are, and how long they have been around.

The customer base is a pyramid, which is what a real membership programme looks like:

| Tier | Customers | What it is |
|---|---|---|
| Free | 20 | No payment. The entry level, and the largest group. |
| Basic | 14 | The cheapest paid plan. |
| Silver | 10 | Ordinary paying customers. |
| Gold | 7 | The start of the premium range. |
| Platinum | 5 | High-value accounts. |
| Diamond | 4 | Enterprise-level. Smallest group, largest revenue. |

## The columns

| Column | What it says about the customer |
|---|---|
| `tier` | Which membership plan they are on, `Free` through `Diamond`. |
| `tier_code` | The same plan written as a number, 1 for Free up to 6 for Diamond, so it can be fed to a model. It records only the **order** of the plans — it does not claim a Diamond customer is "six times" a Free one. |
| `monthly_spend` | What the customer actually pays per month, in dollars. A Free customer spends around \$3 on incidental purchases; a Diamond customer around \$900. This is the revenue the business earns from them. |
| `support_tickets` | How many times per month they contact support. Free customers file about 12; Diamond customers about 1. Premium plans include onboarding and a dedicated account manager, so the heaviest users of support are the ones paying the least. |
| `satisfaction` | Their rating of the service, 1 (unhappy) to 5 (delighted). It rises with tier — **except Gold and Platinum, who both report 4**. Paying for the more expensive plan did not make them any happier. |
| `account_age_months` | How long they have been a customer, 1 to 47 months. It has nothing to do with tier here: four years of loyalty does not imply an upgrade, and a Diamond customer may have joined last month. |

## The story in the data

**The customers who pay the most cost the least to serve, and are the happiest.** Spend rises with tier,
support load falls with tier, and satisfaction rises with tier. `account_age_months` is the control — it
should be unrelated to everything, and it is.

## Why this dataset was built this way

The tiers are priced **multiplicatively**, roughly 3x per step:

| Tier | Typical monthly spend | Step from previous |
|---|---|---|
| Free | \$3 | — |
| Basic | \$8 | +\$5 |
| Silver | \$22 | +\$14 |
| Gold | \$65 | +\$43 |
| Platinum | \$210 | +\$145 |
| Diamond | \$900 | **+\$690** |

One tier of difference means \$5 at the bottom of the table and \$690 at the top. But `tier_code` spaces
those six plans evenly — 1, 2, 3, 4, 5, 6 — because that is how we wrote the codes, not because reality
works that way.

That mismatch is the entire point of the section. Pearson assumes the gap between codes 1 and 2 is the
same size as the gap between codes 5 and 6, so it reports `tier_code` vs `monthly_spend` as a merely
moderate **0.736**, while Spearman — which only reads the ordering — reports **0.972**.

Two design choices make the demonstration airtight:

- **No outliers.** Each tier's spend varies only about 1.2x from its own minimum to its own maximum, and
  no tier's range touches the next tier's. Six tight, well-separated clusters, nothing to clean. Pearson's
  low number cannot be blamed on a bad value.
- **Only one column is multiplicative.** `support_tickets` and `satisfaction` are ordinal or count columns
  whose real steps happen to be roughly even, and Pearson handles both correctly. `monthly_spend` is the
  only column on a multiplicative scale — so it is the only row and column that lights up in the
  Spearman-minus-Pearson difference heatmap.

The takeaway for the slide: the problem is not "categorical data" in general, it is the mismatch between
**evenly spaced codes** and **unevenly spaced reality** — and the difference heatmap points straight at
the column responsible.
