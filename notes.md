# Fixed income derivatives: an explainer

Why traders use IRS, OIS and CDS on top of bond holdings, how each is priced and unwound, and how a trade is captured for cost analysis.

## Why these exist

A bond's price moves for two separate reasons. Interest rate derivatives and credit derivatives each trade one of them, on its own, without buying or selling the bond itself.

### Interest rate risk

A bond pays a fixed coupon. If market rates rise after you buy it, new bonds are issued paying more than yours. Your bond's future cash flows now get discounted at a higher rate, which lowers their present value, so the bond's price falls, even though the borrower's ability to pay hasn't changed at all. The reverse happens if rates fall.

### Credit risk

Separately, the market can decide a borrower is more or less likely to default. That view is priced as a **credit spread**: the extra yield a bond pays over a risk-free bond of the same maturity. If the spread widens, the bond's price falls for a reason that has nothing to do with interest rates.

Pension funds, insurers and large asset managers hold rate and credit exposure at scale, and their views and mandates change more often than they want to trade the underlying bonds. Selling and rebuying bonds each time is slow and can move the market against them. A derivative lets them adjust one risk at a time, layered on top of the existing bond holdings, in a single trade:

- **Interest rate swap (IRS):** exchange a fixed interest payment for a floating one, to manage rate risk.
- **Credit default swap (CDS):** pay a premium in exchange for compensation if a borrower defaults, similar to insurance on the bond.

## Types we support

| Product | Risk it trades | Underlying |
|---|---|---|
| IRS | Interest rate | A term reference rate, e.g. 3m or 6m EURIBOR |
| OIS | Interest rate | An overnight rate, compounded daily |
| Index CDS (ICDS) | Credit | A basket of ~50-125 borrowers (e.g. CDX, iTraxx) |
| Single-name CDS (SCDS) | Credit | One borrower (corporate or sovereign) |

Currencies traded include EUR, GBP, USD, CAD, NZD and HKD for IRS and OIS.

## Interest rate swaps: IRS and OIS

Both exchange a fixed coupon for a floating coupon, calculated on a shared **notional**. The notional itself never changes hands; it only sizes the payments. The difference between the two products is how the floating side is set.

In an **IRS**, the floating rate resets every 3 or 6 months: at the start of each period, that period's floating payment is fixed to whatever the reference rate is on that day. A 2-year IRS with semi-annual resets has its floating payment set four separate times over the trade's life, so what you'll receive in period 3 isn't known until period 3 begins. In an **OIS**, the floating rate resets daily off an overnight rate, and the daily rates are compounded together to produce each payment. This keeps the floating leg closer to current conditions than a term rate that was only set once, months earlier.

### Rate risk and hedging

Take a bond: notional £10m, fixed coupon 5% a year. A 1% rise in rates cuts its price by an amount that depends on how long the bond has left to run; a bond with 10 years left falls further than one with 6 months left, because it's locked into a below-market rate for longer. That sensitivity is **duration**, expressed in years, and the same idea applies to a swap's own market value.

To hedge the bond above, the bondholder enters a swap to **pay fixed, receive floating**. When rates rise, the floating payments received rise too, offsetting the fall in the bond's value. The counterparty on the other side, who **receives fixed, pays floating**, gains instead if rates fall.

<details>
<summary><b>Worked example: hedging a €500m bond portfolio</b></summary>

A portfolio holds €500m of bonds with an average duration of 4.5 years. Rates rise by 1%:

$$ \Delta V_{\text{bond}} = -N \times D_{\text{bond}} \times \Delta y = -500\text{m} \times 4.5 \times 1\% = -\$22.5\text{m} $$

The portfolio hedges with a €500m, 5-year IRS, paying fixed and receiving floating. A swap has its own duration, here 4.4, slightly different from the bond's because a swap has no principal repayment and different cash flow timing. When rates rise 1%, the payer of fixed gains:

$$ \Delta V_{\text{swap}} = N \times D_{\text{swap}} \times \Delta y = 500\text{m} \times 4.4 \times 1\% = +\$22\text{m} $$

Net effect: a $0.5m loss instead of $22.5m. The hedge isn't exact because the two durations, 4.5 and 4.4, don't quite match: closing that gap is a core part of a rates trader's job.

</details>

### Mark to market

A swap's **mark to market (MTM)** is its present value today: sum every future payment on both legs, discount each back to today, and net the two legs. The floating leg's future resets are estimated from today's market curve, so the MTM changes daily as rates move, even though no cash has changed hands.

> **Example: a swap starting today**
>
> Notional £10m. Trade date 13 Sep 2025, start date 15 Sep 2025, maturity 15 Sep 2027 (2 years). Buy side pays fixed 3% annually, receives floating 6m EURIBOR semi-annually, currently 3%. No upfront amount is paid.
>
> That's not a coincidence. The fixed rate, 3%, is chosen so the present value of what's paid exactly equals the present value of what's expected to be received, on day one. That is what "done at market" means: the deal is priced to be worth zero to both sides at inception. Only later, as rates move away from 3%, does the swap gain or lose value.

### Unwinds

An **unwind** terminates the swap before its scheduled maturity, settling the current MTM as a single cash payment. The unwind's trade date naturally falls after the swap's original start date, since it only makes sense to unwind a swap that's already running.

<details>
<summary><b>Worked example: unwinding the £10m swap</b></summary>

Same swap as above. On 20 Jan 2026 the buy side wants out. Rates have risen by 1% since inception, and roughly 1.6 years remain.

The buy side is paying fixed at 3% while the market rate for the remaining term is now around 4%, a good deal for them, so the swap has positive value to the buy side. Using the remaining-term annuity (≈1.6):

$$ \text{DV01} = 10\text{m} \times 1.6 \times 0.0001 = £1{,}600 \text{ per bp} $$

$$ \text{MTM} = 100\text{bp} \times £1{,}600 = £160{,}000 $$

The counterparty, paying floating at ~4% but only receiving 3% fixed, is on the losing side, so they pay the buy side about £160,000 to close the trade. That's roughly 1.6% of notional, proportional to how far rates moved and how much time was left.

</details>

> **Sanity check**
>
> An upfront figure many times the notional is a sign a field has been entered wrongly, not a real trade. It should scale with the size of the market move and the time remaining, typically a low single-digit percentage of notional for a move of this size.

## Credit default swaps: SCDS and ICDS

A CDS exchanges a fixed payment, the **protection premium**, for a payoff if the underlying borrower defaults.

- **SCDS (single-name):** protection on one borrower, a corporate or a sovereign. If that name defaults, the contract pays out; otherwise it runs to term collecting premium.
- **ICDS (index):** protection on a standardised basket of 50-125 borrowers at once, each an equal slice of the notional. If one name defaults, that slice pays out and the rest of the contract continues, effectively many small SCDS contracts bundled into one trade.

Credit quality is scored by rating agencies from AAA down to D. AAA/AA names are the small group of governments and blue-chip companies that historically almost never default. Single-B and below is "high yield" or "junk" territory, where a meaningful minority of issuers do default over a multi-year horizon. The better the rating, the less likely a payout, so the cheaper the protection: a AAA name might cost a few basis points a year to insure, a stressed single-B name many multiples of that.

A CDS works like insurance on the bond: the protection buyer pays a periodic premium, and the protection seller pays out if the borrower defaults. If nothing happens, the buyer has paid for cover and the contract runs to term with no payout.

<details>
<summary><b>Worked example: hedging a €500m credit portfolio</b></summary>

A €500m portfolio of investment-grade US corporate bonds, average maturity 5 years, credit duration 4.5. Credit spreads widen by 0.5% (50bp) across the market:

$$ \Delta V_{\text{bonds}} = -500\text{m} \times 4.5 \times 0.5\% = -\$11.25\text{m} $$

The portfolio buys €500m of protection via a 5-year index CDS, credit duration 4.4. As protection buyer, the position gains when spreads widen:

$$ \Delta V_{\text{CDS}} = 500\text{m} \times 4.4 \times 0.5\% = +\$11\text{m} $$

The mismatch between 4.5 and 4.4 leaves a small residual, for the same reason as the rates hedge above. What a trader watches day to day is the index spread level, quoted in basis points: if it's trending wider, the hedge is doing its job, and the premium paid for it is a small, steady cost against the loss it offsets.

</details>

CDS MTM and unwinds work on the same principle as an IRS: present value the difference between today's market spread and the trade's spread, over the remaining life, and convert to money using credit duration (also called CS01). If the index above widened from 60.5bp at trade to 70bp, and CS01 on the $25m position is $11,250 per bp, the position has gained $106,875, and an unwind at that level would pay the protection buyer that amount.

## How these are traded

IRS, OIS and CDS trade over the counter, not on an exchange. A trader requests a quote from one or more dealers, or uses an electronic platform, and gets a two-way price (bid and offer) rather than a single public price like a listed stock. The level quoted can depend on which dealer is asked and how much size is shown.

Most standardised swaps and index CDS clear through a central counterparty (CCP): once agreed, the trade is given up to the CCP, which becomes the counterparty to both sides. Both parties post initial margin up front and daily variation margin equal to that day's change in MTM, which is why MTM is calculated daily rather than only at unwind.

ICDS, and increasingly SCDS, trade on fixed coupons set by market convention (typically 100bp or 500bp), not on whatever spread the market currently prices. The gap between the market spread and the fixed coupon is exactly what the upfront payment settles, which is why an upfront is the norm for these trades rather than the exception.

If the underlying borrower defaults, the position doesn't simply pay out at face value. ISDA runs a credit event auction to set a recovery value for the defaulted debt, and the protection seller pays the protection buyer par minus that recovery value.

> **Why benchmarking matters here**
>
> Because these trades are OTC, there's no single public tape of "the price" at a given moment, the way there is for a listed stock. Judging whether an execution was fair means comparing it to other data captured close to the same time: dealer quotes, index levels, or a clearer's published settlement price. That comparison is the basis for cost analysis on this asset class.

## Required data fields

Every trade record needs enough fields to identify exactly what was traded and reprice it later. The set below is general; specific workflows add more on top.

| Field | Meaning |
|---|---|
| Order ID | Client order ID. |
| Transact time | Timestamp of the trade, in an agreed format, GMT. |
| Effective date | Start date of the swap. Can be in the past for an unwind, T+2 for a fresh spot-starting swap, or in the future for a forward-starting one. |
| Maturity | Date the swap terminates. |
| Trans type | Trade type: IRS, CDS, etc. |
| Underlying index | Required for ICDS and SCDS; used for IRS when detailed trade economics aren't otherwise captured, e.g. "EUR IBOR 3m" or "6m". |
| Side | Receive fixed / pay fixed (rates), or sell protection / buy protection (credit). |
| Coupon | The fixed rate, as a percentage: 5% is entered as 5.0. |
| Notional | Trade notional, in units of currency. |
| Upfront | Cash paid or received by the client to enter the trade, in settlement currency. Usually zero for a fresh spot-starting swap, since it's done at market (see above), but typically non-zero for unwinds and for ICDS, where the standard coupon rarely matches the fair market rate. |
| Upfront clean | True/false flag for whether the upfront figure includes accrued interest. Usually false, except in some ICDS reporting. See below. |
| Currency | Currency the notional is denominated in. |
| Settlement currency | Currency the upfront and other cash payments actually settle in. |

<details>
<summary><b>What "upfront clean" and accrued interest mean</b></summary>

Coupons are paid on fixed dates, for example the 20th of March, June, September and December, but a trade can happen on any day between two payment dates. Whoever holds the position on the next payment date receives the whole coupon for that period, including the days before they entered the trade. To make that fair, the buyer pays the seller that unearned slice of interest up front, the same way a bond buyer pays the seller accrued interest when trading between coupon dates.

A **clean** upfront strips this accrued amount out and shows only the compensation for the change in rate or spread. A **dirty** upfront (clean = false) bundles the two together into the single cash figure that actually settles.

</details>
