# Deep Dive: Goal Forecasting Math & Bank-Data Integration

## Part 1: Goal Forecasting & "On Track" Logic

### 1.1 Core data model per goal
Each goal needs:
- `target_amount`
- `target_date`
- `current_amount` (synced from linked account/category)
- `start_amount` and `start_date` (baseline for measuring progress)
- `contribution_history` (array of {date, amount})
- `assumed_growth_rate` (annualized, e.g., 0% for cash savings, 5–7% for invested goals)

### 1.2 Required contribution calculator
This answers: "How much do I need to save per period to hit my goal?"

**Simple case — no investment growth (e.g., cash savings goal):**
```
remaining_amount = target_amount - current_amount
periods_remaining = months_between(today, target_date)
required_monthly_contribution = remaining_amount / periods_remaining
```

**With investment growth — use the future value of an annuity formula, solved for payment:**
```
r = monthly_rate = assumed_growth_rate / 12
n = periods_remaining (months)
FV_target = target_amount - (current_amount * (1 + r)^n)   // future value current balance grows to

required_monthly_contribution = FV_target * r / ((1 + r)^n - 1)
```
This tells the user the level monthly contribution needed, accounting for compounding on what they've already saved/invested.

### 1.3 "On track / behind / ahead" status
At any point in time, compare **actual progress** to **expected progress**:

```
elapsed_fraction = (today - start_date) / (target_date - start_date)
expected_amount = start_amount + (target_amount - start_amount) * elapsed_fraction
   // straight-line expectation; for growth-bearing goals, use the FV curve instead of linear

variance = current_amount - expected_amount
variance_pct = variance / expected_amount
```

Suggested thresholds:
- `variance_pct >= 0` → **Ahead of schedule**
- `-5% < variance_pct < 0` → **On track** (small buffer avoids flip-flopping status on minor fluctuations)
- `variance_pct <= -5%` → **Behind schedule**

When behind, recompute the *required contribution from today forward* (re-run 1.2 with `current_amount` and `periods_remaining` updated) and show the user the new number — this is more useful than just saying "you're behind."

### 1.4 Forecasting future balance (the "what-if" simulator)
For a given set of assumptions, project the balance forward:

```
projected_balance(t) = current_amount * (1 + r)^t + contribution * ((1 + r)^t - 1) / r
```
where `t` is months from now, `contribution` is the recurring monthly amount, `r` is monthly growth rate.

Let the user adjust three sliders — contribution amount, growth rate assumption, target date — and re-render this curve live. This is the single highest-value interactive feature for engagement; people will play with it.

### 1.5 Handling irregular contributions (real-world data)
Real users don't save the exact same amount every month. Two practical approaches:
- **Trailing average method:** use a 3- or 6-month rolling average of actual contributions as the "current pace," and compare that pace against the required contribution.
- **Linear regression on contribution_history:** fit a trend line to detect if contribution pace is increasing/decreasing over time — more sophisticated, useful for a "trending toward X" insight.

Start with the trailing average; it's simpler and easier for users to understand than a regression-based projection.

### 1.6 Net worth / multi-goal forecasting
If a user has multiple goals drawing from the same income, you'll eventually want a **global forecast**: total monthly free cash flow minus committed contributions across all goals, flagging if goals are over-allocated relative to actual income. This is a v2 feature — don't build it into the MVP, but design the data model (per-goal `monthly_contribution` field) so it's easy to aggregate later.

---

## Part 2: Bank-Data Integration Approach

### 2.1 Build vs. buy decision
Building your own bank-scraping integration is realistically not worth it for a startup-stage app — banks change their interfaces constantly, security/compliance burden is heavy, and aggregators have already solved this. Use an aggregator:

| Provider | Notes |
|---|---|
| Plaid | Most popular in the US, strong developer docs, widest bank coverage, transaction categorization included |
| MX | Strong on data enrichment/categorization, popular with fintech apps |
| Finicity (Mastercard) | Good for verification + transaction data, enterprise-leaning |
| Akoya | Bank-owned network (lower friction with major US banks), newer |
| TrueLayer / Salt Edge | Better options if you need UK/EU coverage |

For a US-focused MVP, **Plaid** is the standard default — best docs, broadest coverage, and most fintech tutorials/support exist for it.

### 2.2 Integration flow (typical Plaid-style pattern)
1. **Link token creation** — your backend requests a short-lived token from the aggregator to initialize the connection flow.
2. **Client-side Link widget** — user selects their bank and logs in through the aggregator's secure widget (you never see or store their bank credentials).
3. **Public token exchange** — after successful login, the client gets a `public_token`, which your backend exchanges for a permanent `access_token` tied to that user's linked account.
4. **Initial data pull** — fetch accounts, balances, and historical transactions (typically last 24 months available).
5. **Ongoing sync** — set up webhooks for transaction updates, balance refreshes, and re-authentication prompts (banks periodically require the user to re-confirm login).
6. **Store only the access_token + account metadata** in your database — never raw bank credentials; the aggregator handles that layer entirely.

### 2.3 What you store on your side
- `access_token` (encrypted at rest)
- `item_id` / `account_id` mappings
- Transaction records (synced copies — you'll want your own copy for fast queries rather than calling the aggregator API on every dashboard load)
- Last-synced timestamp per account, to know when to trigger a refresh

### 2.4 Sync strategy
- **Webhook-driven (preferred):** aggregator pushes a notification when new transactions are available; your backend pulls the delta and updates your database. This keeps your app near-real-time without polling costs.
- **Scheduled fallback:** nightly batch job to catch anything webhooks missed, plus refresh balances for forecasting accuracy.
- Avoid pulling full transaction history on every load — incremental sync only.

### 2.5 Categorization
Aggregators provide auto-categorization (e.g., Plaid's transaction categories), but expect ~80–90% accuracy out of the box. Plan for:
- A manual override/recategorize flow for users
- A rules engine (e.g., "always categorize transactions from 'Starbucks' as Dining") that applies retroactively and to future transactions
- Storing both the aggregator's raw category and your app's resolved category, so you can improve your own categorization model over time using user corrections as training signal

### 2.6 Costs to plan for
Aggregators charge per linked item/account, often with tiered pricing (production access usually requires a paid plan, with a free developer/sandbox tier for building). Factor this into your unit economics early — it scales with user count, so model it against your monetization plan rather than treating it as a fixed cost.

### 2.7 Re-authentication handling
Banks periodically invalidate the connection (password change, security policy, MFA reset). Aggregators send an "item error" / "re-auth required" webhook — surface this clearly in-app ("Reconnect your Chase account") rather than letting data silently go stale, since stale balances directly break your forecasting accuracy.

### 2.8 Compliance note
Even though the aggregator handles credential storage, you are still handling sensitive financial data (transactions, balances) — this brings you into scope for data privacy regulations (GDPR/CCPA if applicable) and warrants the security practices listed in the original checklist (encryption at rest/in transit, access controls, audit logging). Consult the aggregator's compliance documentation, as they typically publish guidance for what obligations fall on you vs. them.

---

**Suggested next build step:** Implement section 1.2–1.3 (required contribution + on-track status) first using manually-entered balances — this lets you validate the forecasting UX before taking on the complexity of a bank-data integration.
