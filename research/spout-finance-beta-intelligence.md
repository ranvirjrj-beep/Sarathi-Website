# Spout Finance Beta Intelligence — Economic Truth Layer

**Hands-on product feedback for the Spout Finance Solana Testnet beta**  
Prepared by **@ranvirjrj-beep** · 15 September 2026

## Executive summary

Spout has a strong core idea: let users hold tokenized equities, borrow stablecoins at **0% cash interest**, and fund the economics through a covered-call engine rather than borrower APR.

After testing the current gated Solana Testnet beta, walking through the full 11-step onboarding tour, reviewing the Trade / Borrow / Earn surfaces, inspecting an individual stock page, and interrogating the in-product **Ask Spout** assistant, my main conclusion is:

> **The product explains the headline better than it explains the trade.**

The 0% interest claim can be true while the borrower still bears a real contingent economic cost: if a covered call is assigned, collateral shares can be sold at the strike and upside above that strike is forgone for that cycle. Spout's Terms and Ask Spout explain this, but the primary Borrow / Trade decision flow does not surface it before wallet connection.

I found three high-confidence live-beta issues:

1. **Covered-call assignment / capped-upside economics are not surfaced in the main borrowing flow.**
2. **The stock table shows a non-zero annualized `Borrow Cost` metric without explaining what it means.**
3. **The Earn page says lending is “coming soon”, while onboarding presents Earn, weekly interest, and Senior / Junior selection as if they are currently usable.**

My proposed fix is an **Economic Truth Layer**: before a user signs a borrow or lend action, show the economic trade in plain language — not just the headline rate.

---

## What I tested

I used the official gated beta at **https://beta.spout.finance/** on **Solana Testnet**.

I directly reviewed:

- Trade stock list and Buy panel
- Borrow landing page
- Earn landing page
- GOOG / Alphabet detail page
- the complete 11-step guided tour
- the `Borrow Cost` column and attempted tooltip / detail explanations
- Ask Spout responses on borrowing cost, covered-call exercise, and why Spout uses covered calls

I did **not** use mainnet funds, make a real-money trade, or claim a transaction flow I did not execute. The findings below distinguish observed beta behavior from documentation analysis.

---

# Finding 1 — 0% cash interest is clear; the upside tradeoff is not

### What the main product flow says

The live beta repeatedly emphasizes:

- **“0% Interest on borrowing. Always, no matter the market conditions.”**
- Trade: **“Buy/Sell stocks, then borrow against them at 0% interest, without selling.”**
- Borrow: every **$10,000** of eligible stock can support up to **$5,000 USDC** at 0% interest.
- Tour step 5: **“Leverage: buy more with 0% interest.”**
- Tour step 6: unused leverage can later become a **0% cash loan**.
- Tour step 7: borrow USDC against stock, **keep the stock and pay 0% interest**.

That is a compelling onboarding story — but it is incomplete at the moment of decision.

### What Ask Spout and the Terms say

When asked **“What happens if a covered call gets exercised?”**, Ask Spout explains that if the stock finishes above the strike at expiry:

- the call is assigned;
- the shares are sold at the strike;
- proceeds cover the debt first;
- the residual belongs to the user;
- with Auto-Roll on by default, the residual is used to repurchase the same stock and re-enroll the position.

Ask Spout also explains that Spout **automates covered calls against deposited collateral**, and that the option premium funds 0% borrower interest and lender yield.

The Terms are even clearer. Section 9.2 says that on assignment the underlying shares are sold at the strike, below the prevailing market price, and the borrower loses the difference between market and strike. The Terms explicitly call that difference the **“borrower cost”** and say the user forgoes upside above the strike in exchange for 0% borrowing.

### Why this matters

`0% APR` and `0% economic cost` are not the same statement.

A simplified example:

- collateral: $10,000 of stock at $100/share
- borrow: $5,000 at 50% LTV
- call strike: $105
- stock ends the cycle at $120

If assigned, the shares are sold at $105 rather than being worth $120. Ignoring premium only to isolate the payoff, the forgone upside is **$1,500**.

This does not mean assignment is expected or frequent. Spout says calls are written far out of the money to keep assignment probability low. The product issue is that the consequence is materially important even if the probability is low.

### Recommended fix: Upside Tradeoff Card

Before a borrower confirms leverage or a cash loan, show:

- current stock price
- proposed strike
- option expiry
- distance to strike
- estimated assignment probability / delta range
- expected option premium
- maximum upside retained before assignment
- a simple payoff slider: **“If the stock ends at…”**
- two distinct labels:
  - **Cash interest: 0%**
  - **Contingent assignment / opportunity cost: possible**

Suggested copy:

> **0% cash interest. Spout funds the loan by writing covered calls against your collateral. If the stock finishes above the strike, your shares can be sold at the strike and upside above it can be forgone for that cycle.**

This makes the headline stronger, not weaker, because users can see exactly why 0% is economically possible.

---

# Finding 2 — `Borrow Cost` is decision-critical but undefined in the beta

The live Trade table contains a column labeled **Borrow Cost** with annualized values such as:

- GOOG: **0.54%/yr**
- GS: **0.86%/yr**
- XOM: **0.80%/yr**
- AAPL: **0.37%/yr**

At the same time, the product repeatedly advertises **0% borrowing**.

I attempted to resolve this in three places:

1. Hover / tooltip on `Borrow Cost` — **no tooltip appeared**.
2. GOOG detail page — **no definition of the metric**.
3. Ask Spout — I asked whether GOOG's 0.54%/yr is charged to the borrower, or whether it is a separate stock/security financing cost.

Ask Spout did **not** answer what the metric means. Instead it returned generic protocol economics: 20% protocol fee on gross option premium, 80% to lenders, no borrower interest charge, Solana network fees, and a liquidation penalty.

### The likely intended meaning is important

Spout's public FAQ says borrowers absorb the difference between strike and market price when an option is assigned and notes that this cost has historically averaged around **0.5% annualized** across the portfolio. The Terms call the market-vs-strike difference the **borrower cost**.

That strongly suggests the table is communicating an annualized estimate of assignment / forgone-upside cost rather than loan interest — but the beta does not explain the distinction.

### Recommended fix

Rename or define the metric in-place, for example:

**Estimated assignment cost (annualized)** `ⓘ`

Tooltip:

> Historical / modeled opportunity cost from covered-call assignment for this asset. This is **not USDC loan interest**. Your cash interest rate remains 0%. Actual cost depends on whether calls are assigned and the stock price relative to strike.

Also expose the methodology: lookback period, realized vs modeled basis, update frequency, and whether the number includes option premium retained by the strategy.

This one clarification would eliminate a major moment of confusion created by showing `0.54%/yr` next to a 0% borrowing proposition.

---

# Finding 3 — Earn onboarding is ahead of the actual beta state

The current Earn page says:

- **“Earn is coming soon”**
- **“Lending vaults are on the way.”**

However, the guided tour still presents Earn as usable:

- step 8: deposit spare USDC to earn interest, paid weekly;
- choose **Senior** for steadier / lower return or **Junior** for higher return / more risk;
- step 9 explains Senior vs Junior and says users can compare who earns what, who gets paid first, and who bears losses.

This is a straightforward beta-state inconsistency. The tour teaches an action the live page says is not yet available.

### Recommended fix

Make onboarding feature-aware:

- If Earn is disabled, label the tour step **“Coming soon: Earn”**.
- Explain the intended Senior / Junior model, but do not instruct users to deposit yet.
- Add a waitlist / notify-me CTA instead of an action-oriented deposit walkthrough.

This also changes how the broader lender feedback should be interpreted: yield, tranche risk, exit rules, and buffer visibility are valuable **pre-launch recommendations**, not bugs in a currently active lending flow.

---

# Pre-launch lender recommendations

The documentation contains sophisticated lender mechanics that deserve to be visible in-product when Earn launches.

## 1. Show the loss waterfall before deposit

The Terms specify the loss order:

**Insurance Fund → Junior → Protocol Treasury → Senior**

Senior is safer in the waterfall, not risk-free. Junior is explicitly first-loss after the insurance fund.

A simple visual loss waterfall would be more useful than the broad phrase “no options exposure” because lender returns and tail risk are still economically connected to the covered-call engine.

## 2. Put exit constraints on the product card

Terms currently specify:

- Senior: instant exit with a dynamic **0–3% haircut**, or FIFO at full NAV.
- Junior: **45-day minimum notice**, no instant exit.
- Claim resale: possible, but liquidity / price not guaranteed.

These are decision-level product characteristics and should appear before deposit.

## 3. Use one canonical yield evidence module

Instead of static APY numbers across different documents, show:

- target APY range
- trailing 7d / 30d / 90d realized annualized yield
- gross option premium
- protocol fee
- net lender yield
- utilization / idle capital
- assignment rate
- loss weeks / max drawdown
- Insurance Fund balance **and coverage ratio**
- timestamp and source

The strongest part of Spout's yield story is that it is tied to real options-market activity rather than temporary token incentives. Evidence should be a product feature.

---

# Product proposal — The Economic Truth Layer

Spout's product is unusual enough that risk clarity can become a competitive advantage.

### Borrow — before signing

Show one compact sentence / card:

> **Borrow $X at 0% cash APR → collateral $Y → call strike $Z → expiry date → assignment risk → upside above strike may be capped this cycle → liquidation threshold → collateral release / cooldown.**

### Earn — before deposit

When lending launches:

> **Target APY range → trailing realized yield → Senior / Junior loss position → current Insurance + Junior buffer → exit rule → estimated queue / haircut.**

Then ask for one plain-language acknowledgement rather than forcing the user to infer these mechanics from Terms and FAQs.

---

# What Spout already does well

This review is not a claim that the core model is misleading or broken.

Several things in the beta are strong:

- the 50% max-LTV concept is easy to understand;
- the Health Factor visualization gives users a clear risk anchor;
- the Trade screen exposes per-asset information instead of hiding everything behind a generic borrow button;
- the product openly documents the covered-call engine in its deeper materials;
- the beta states that shares are held 1:1 at Alpaca Securities and shows a live reserve figure (`Reserves 100.2%` during my test);
- Ask Spout can answer deeper mechanism questions without sending the user away from the product.

The opportunity is to move the most important economic truth **forward in the funnel** — from FAQ / legal depth into the actual decision moment.

---

# Priority order

If I could ship only three changes before broader acquisition:

1. **Add the Upside Tradeoff Card to Borrow / leverage confirmation.**
2. **Define `Borrow Cost` directly in the table and distinguish it from 0% cash interest.**
3. **Make the onboarding tour respect current feature availability, especially Earn.**

These are relatively small interface changes with outsized trust impact.

---

## Sources

First-party Spout sources:

- https://spout.finance/
- https://spout.finance/terms/
- https://spout.finance/docs/getting-started/
- https://spout.finance/docs/lending-tranches/
- https://spout.finance/docs/faqs/
- https://beta.spout.finance/

Bounty:

- https://superteam.fun/earn/listing/product-feedback-spout-finance

### Testing note

All beta observations in this report were collected from the official gated Solana Testnet beta on 15 September 2026. No mainnet funds were used. No wallet transaction or signature is claimed where none was executed.
