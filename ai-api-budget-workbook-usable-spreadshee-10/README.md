# AI API budget workbook â€” usable spreadsheet template

This is the actual text edition of the workbook: paste the three tab-separated blocks below into three spreadsheet sheets to obtain a working calculator. No download, account connection, API key or live price feed is required. After setup, save as XLSX if desired. This delivery kit intentionally contains only the four requested files, so no binary workbook is included.

## Instructions and assumptions

1. Create blank sheets named exactly `Inputs`, `Compare`, and `Scenarios`. Paste each corresponding block into cell A1. Tabs separate columns. If everything lands in one column, use the spreadsheet's split-text/import function with Tab as the delimiter.
2. Formulas use English Excel function names and comma argument separators. For a locale that requires semicolons, replace formula argument commas with semicolons. Ensure pasted formulas evaluate rather than appearing as text.
3. All numbers are invented examples, not current or verified prices for any service. Use one currency for all prices and subscription budgets. The examples use USD. Prices are per **million tokens**, not per token or per thousand.
4. Daily input means **total billed input**, including cached input. Cached fraction is entered as a decimal from 0 to 1. Regular input = total input Ã— (1 âˆ’ cached fraction). Do not add cached input to total input again. Daily output is separate.
5. These examples assume 30 billing days, constant daily use, linear rates, and a constant cached fraction and output/input ratio as usage scales. Enter the actual billing-period length. For a weekly/monthly usage log, divide totals by observed days to estimate daily use; a short log may not represent a whole month.
6. A repeated prompt does not prove a cache hit. Use the fraction actually billed as cached. If the gateway omits its cached-input price, set cached price equal to regular price for a conservative input-charge estimate. This is a budgeting assumption, not a claim about its billing policy. Alternatively set cached fraction to zero. Ask the gateway whether cached tokens are discounted, at what rate, and whether credits, markups or separate cache charges apply.
7. Use the effective rate you pay after any known gateway conversion or markup. Monthly fixed fee is an API-side fee incurred regardless of volume. It is not the subscription alternative's price. Add other fixed charges here only if applicable. Unmodeled usage-based charges, taxes, storage/write charges, tools, minimum charges, credit rounding, FX and tiered pricing can change the answer; account for them separately before making a purchase decision.
8. Compare only alternatives appropriate to your workload. Subscription access may not include API access and may impose limits or offer different features. â€œBudget headroomâ€ is arithmetic subscription price minus API cost, not a promise of savings. It is positive when modeled API cost is lower.
9. Validate inputs before relying on results: prices, daily volumes and fees must be nonnegative; days must be positive; cached fraction must be between 0 and 1. This paste-in template does not enforce data validation. In a delivered XLSX, apply these restrictions and visually distinguish input cells from formula cells.

## Sheet 1: Inputs

Edit B2:J4; rename options in A2:A4. All three rows start with identical usage so their rates can be compared fairly. Each subscription budget may be changed independently.

```tsv
Option	Regular input / 1M	Cached input / 1M	Output / 1M	Total input / day	Cached fraction	Output / day	Days / month	Subscription / month	API fixed fee / month
Example A	2	0.5	8	100000	0.75	10000	30	20	0
Example B	1	0.25	4	100000	0.75	10000	30	20	1
Example C	3	0.75	12	100000	0.75	10000	30	20	2
```

To model unknown cached billing for A, replace C2 with `2`. The resulting monthly API estimate is 8.40 instead of 5.025, excluding any omitted charges. Neither number establishes the gateway's actual billing rate.

## Sheet 2: Compare

Paste as-is. Format B:H as currency with at least two decimals, I as a number with three decimals, J as a whole number and K as a number with two decimals. Calculations retain full precision.

```tsv
Option	Regular cost / month	Cached cost / month	Output cost / month	Variable cost / month	Total API / month	Subscription / month	Budget headroom	Break-even usage multiplier	Break-even total input / day	Days to spend subscription budget
=Inputs!A2	=Inputs!B2*Inputs!E2*(1-Inputs!F2)*Inputs!H2/1000000	=Inputs!C2*Inputs!E2*Inputs!F2*Inputs!H2/1000000	=Inputs!D2*Inputs!G2*Inputs!H2/1000000	=SUM(B2:D2)	=E2+Inputs!J2	=Inputs!I2	=G2-F2	=IF(G2<Inputs!J2,"Already above at zero use",IF(E2=0,IF(G2=Inputs!J2,"Equal at all volumes","No crossing"),(G2-Inputs!J2)/E2))	=IF(ISNUMBER(I2),Inputs!E2*I2,"See multiplier")	=IF(G2<=Inputs!J2,0,IF(E2=0,"Never at this rate",(G2-Inputs!J2)/(E2/Inputs!H2)))
=Inputs!A3	=Inputs!B3*Inputs!E3*(1-Inputs!F3)*Inputs!H3/1000000	=Inputs!C3*Inputs!E3*Inputs!F3*Inputs!H3/1000000	=Inputs!D3*Inputs!G3*Inputs!H3/1000000	=SUM(B3:D3)	=E3+Inputs!J3	=Inputs!I3	=G3-F3	=IF(G3<Inputs!J3,"Already above at zero use",IF(E3=0,IF(G3=Inputs!J3,"Equal at all volumes","No crossing"),(G3-Inputs!J3)/E3))	=IF(ISNUMBER(I3),Inputs!E3*I3,"See multiplier")	=IF(G3<=Inputs!J3,0,IF(E3=0,"Never at this rate",(G3-Inputs!J3)/(E3/Inputs!H3)))
=Inputs!A4	=Inputs!B4*Inputs!E4*(1-Inputs!F4)*Inputs!H4/1000000	=Inputs!C4*Inputs!E4*Inputs!F4*Inputs!H4/1000000	=Inputs!D4*Inputs!G4*Inputs!H4/1000000	=SUM(B4:D4)	=E4+Inputs!J4	=Inputs!I4	=G4-F4	=IF(G4<Inputs!J4,"Already above at zero use",IF(E4=0,IF(G4=Inputs!J4,"Equal at all volumes","No crossing"),(G4-Inputs!J4)/E4))	=IF(ISNUMBER(I4),Inputs!E4*I4,"See multiplier")	=IF(G4<=Inputs!J4,0,IF(E4=0,"Never at this rate",(G4-Inputs!J4)/(E4/Inputs!H4)))
```

Break-even multiplier = (subscription âˆ’ API fixed fee) / baseline monthly variable cost. It scales **both** input and output while preserving their ratio and the cached fraction. Break-even daily input is meaningful only with that same mix; multiply daily output by the same multiplier too. If baseline daily input is zero but output is positive, the multiplier still applies to output; the zero input threshold is not a standalone usage limit.

â€œDays to spend subscription budgetâ€ assumes the API fixed fee is charged at the start and baseline daily usage stays constant. It describes spending within one billing period. If the result exceeds the period's days, read it as â€œbudget not reached this periodâ€; do not extrapolate across new monthly fees. Zero means the fee alone meets or exceeds the budget.

## Sheet 3: Scenarios

The multiplier scales both input and output, leaving fixed fees unchanged. Edit B2:B4 to any nonnegative multipliers. Headroom below zero means API spend exceeds that subscription budget.

```tsv
Scenario	Usage multiplier	Example A API / month	Example B API / month	Example C API / month	A headroom	B headroom	C headroom
Low	0.25	=Compare!E2*$B2+Inputs!J2	=Compare!E3*$B2+Inputs!J3	=Compare!E4*$B2+Inputs!J4	=Inputs!I2-C2	=Inputs!I3-D2	=Inputs!I4-E2
Typical	1	=Compare!E2*$B3+Inputs!J2	=Compare!E3*$B3+Inputs!J3	=Compare!E4*$B3+Inputs!J4	=Inputs!I2-C3	=Inputs!I3-D3	=Inputs!I4-E3
High	4	=Compare!E2*$B4+Inputs!J2	=Compare!E3*$B4+Inputs!J3	=Compare!E4*$B4+Inputs!J4	=Inputs!I2-C4	=Inputs!I3-D4	=Inputs!I4-E4
```

## Worked results and acceptance checks

For A: monthly input is 3,000,000 tokens, comprising 750,000 regular and 2,250,000 cached; monthly output is 300,000. Cost = 0.75 Ã— 2 + 2.25 Ã— 0.5 + 0.30 Ã— 8 = **5.025**. This checks that cached input is not counted twice.

| Output | Example A | Example B | Example C |
|---|---:|---:|---:|
| Baseline variable cost | 5.0250 | 2.5125 | 7.5375 |
| Baseline API including fee | 5.0250 | 3.5125 | 9.5375 |
| Headroom against 20 | 14.9750 | 16.4875 | 10.4625 |
| Break-even usage multiplier | 3.980100 | 7.562189 | 2.388060 |
| Break-even total input/day | 398009.95 | 756218.91 | 238805.97 |
| Low API cost | 1.25625 | 1.628125 | 3.884375 |
| High API cost | 20.1000 | 11.0500 | 32.1500 |

For C at high usage, daily variable cost is 6.975 Ã— 4 / 30 = 0.93; after its 2 fixed fee, the remaining 18 budget lasts **17.91 days**. This illustrates how usage can exhaust a subscription-sized budget well before month end, without asserting anything about a named provider.

Check these independent edits on A, restoring the original inputs between checks:

| Edit | Expected result |
|---|---|
| Cached fraction = 0 | Monthly API = 8.40 |
| Cached fraction = 1 | Monthly API = 3.90 |
| Cached price = regular price = 2 | Monthly API = 8.40 at any cached fraction |
| Both daily volumes = 0, fee = 0 | Monthly API = 0; no crossing; days = â€œNever at this rateâ€ |
| Fee = 20, positive baseline volume | Multiplier = 0; days = 0 |
| Fee = 21 | â€œAlready above at zero useâ€; days = 0 |
| Both daily volumes = 0, fee = 20 | â€œEqual at all volumesâ€; days = 0 |

These are arithmetic acceptance checks, not a claim of testing in every spreadsheet application. Before sending an XLSX, open it in the intended application, confirm formulas calculate and reproduce these results. Add a fourth sheet named `Instructions` containing the assumptions and usage guidance above.

---

**Need it customised, or a full version for your work?** [Message us on Telegram](https://t.me/mostafa_ideas_2050_bot?start=r_page_10), no sign-up.

**محتاج نسخة مخصصة أو كاملة لشغلك؟** [ابعتلنا على تليجرام](https://t.me/mostafa_ideas_2050_bot?start=r_page_10) من غير تسجيل.
