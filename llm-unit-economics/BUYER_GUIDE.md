# LLM Unit Economics: 15-Minute Buyer Guide

Use this guide to decide whether an AI feature can support its proposed price before committing more engineering time.

## 1. Collect one workflow's inputs

Estimate input/output tokens per call, calls per successful run, non-model API and infrastructure cost per run, retry rate, monthly runs per customer, and monthly customer price. Prefer measured p50 and p95 values; label guesses as assumptions.

## 2. Calculate contribution economics

```text
model cost/run = calls × ((input tokens × input price/token)
                 + (output tokens × output price/token))
retry-adjusted model cost = model cost/run ÷ (1 - failure rate)
variable cost/run = retry-adjusted model cost + API cost + infrastructure cost
monthly variable cost/customer = variable cost/run × monthly runs
contribution margin/customer = monthly price - monthly variable cost/customer
contribution margin % = contribution margin/customer ÷ monthly price × 100
break-even runs/customer = monthly price ÷ variable cost/run
```

Payment fees, support, refunds, sales expense, and fixed overhead are separate. This is a product-level estimate, not a full P&L.

## 3. Run three scenarios

| Scenario | Usage | Token volume | Retry rate | What it tests |
|---|---:|---:|---:|---|
| Expected | measured p50 | measured p50 | measured mean | normal economics |
| Heavy user | 3× expected | measured p95 | measured p95 | limits and fair use |
| Stress | 10× expected | 2× p95 | 2× measured p95 | downside exposure |

## 4. Interpret the result

- **Green:** at least 70% contribution margin in the expected case and positive contribution from heavy users.
- **Yellow:** 40–70%, or heavy users lose money. Test routing, caching, output caps, or overages.
- **Red:** below 40% in the expected case. Change pricing or architecture before scaling.

These are planning heuristics, not accounting or financial advice. Products with expensive support or acquisition need more headroom.

## Worked example

For a $29/month assistant with 100 runs per customer:

```text
input token cost/run  $0.018
output token cost/run $0.042
other cost/run        $0.015
failure rate          8%
```

```text
retry-adjusted model cost = ($0.018 + $0.042) ÷ 0.92 = $0.0652
variable cost/run = $0.0652 + $0.015 = $0.0802
monthly variable cost = 100 × $0.0802 = $8.02
contribution margin = $29 - $8.02 = $20.98 (72.3%)
break-even usage = $29 ÷ $0.0802 ≈ 361 runs
```

The expected case is viable under the heuristic, but a user near 361 runs consumes the subscription price before support and overhead. Consider a lower allowance, metered overage, or cheaper route.

## Cheapest experiments first

1. Cap output tokens 25% lower and compare acceptance rates.
2. Route classification, extraction, and rewriting to a cheaper model.
3. Cache stable context where the provider supports it.
4. Log retry causes and fix malformed output or timeout loops.
5. Compare fair-use limits, credit packs, and metered overages.

Change one variable at a time. Track cost per **successful** run and task-quality acceptance—not cost alone.

## Weekly instrumentation

Record model/route, input/output/cached tokens, tool and infrastructure cost, latency, retry count, failure category, and an anonymized quality result. Review p50, p95, and maximum cost, then compare forecast with actual cost.

Do not publish customer data, prompts, credentials, or proprietary logs. For feedback, share only approximate, non-sensitive assumptions in a repository issue.
