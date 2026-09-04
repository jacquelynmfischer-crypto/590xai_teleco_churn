# 590xai_teleco_churn
Assignment for 590 Teleco Churn

# Name: Jacquelyn Fischer

## Dataset

I analyzed the Telco Customer Churn dataset from Kaggle — 7,043 customers from a telecom company, with demographics, account info (tenure, contract type, payment method), and what services they're subscribed to (phone, internet, streaming, security add-ons, etc). The target is whether a customer churned (left for a competitor) — about 26.5% did. The goal is figuring out what's actually driving people to leave and building models that can flag at-risk customers before they go.

## Assumption Checks

| Model | Key Assumptions Checked | Evidence | Concern |
|---|---|---|---|
| Linear regression | No multicollinearity, normal residuals, predictions should actually be valid probabilities | VIF under 5 once I dropped MonthlyCharges/TotalCharges; Omnibus and Jarque-Bera tests both came back p≈0.000; test predictions ranged from -0.22 to 0.77 | Residuals aren't normal, and the model spit out a negative churn probability (-22%), which doesn't mean anything in real life |
| Logistic regression | No multicollinearity, linearity of the logit (each feature's effect should be constant on the log-odds scale) | Same VIF results; the GAM shows tenure's real effect isn't constant | Tenure's actual relationship with churn is curved — steep drop early, then flattens out — so logistic's single tenure coefficient is really just an average, not the full picture |
| GAM | Same multicollinearity concerns, plus it assumes the learned curves are smooth/well-behaved | Tenure's spline term had an EDoF of 13.4 (a straight line would be 1), so the non-linearity is real; the partial dependence plot shows a steep drop 0-10 months, a plateau through the mid-tenure range, then another decline late | pyGAM straight up warns in its own output that its p-values aren't reliable; a couple of the small wiggles in the curve (around month 40 and 60) might just be noise since there's less data out there |

## Model Comparison

| Model | Performance Evidence | Interpretability Strength | Interpretability Weakness |
|---|---|---|---|
| Linear regression | Test R² = 0.304, RMSE = 0.368 | Coefficients read directly as probability-point changes, easy to explain | Predictions aren't bounded to 0-1 (got a -22% prediction), residuals fail normality |
| Logistic regression | Test accuracy 82.1% (baseline of just guessing "no churn" is 73.5%), AUC 0.861 | Coefficients turn into odds ratios that are easy to explain (two-year contract = ~76% lower odds of churn); predictions are always valid probabilities | Assumes each feature's effect is constant, so it undersells how risky brand-new customers actually are; missed 42% of customers who actually churned |
| GAM | Test accuracy 81.8%, AUC 0.864 — basically tied with logistic | Actually shows the real, curved shape of tenure's effect instead of flattening it into one number | Harder to explain to someone non-technical than "here's an odds ratio"; all that extra flexibility didn't really buy us better predictions |

## Recommendation

Recommended model: Logistic Regression

Why this model: It performs basically the same as the GAM (82.1% vs 81.8% accuracy, 0.861 vs 0.864 AUC) but is way easier to explain to people who aren't data scientists — you can just say "two-year contracts cut churn odds by about 75%." And unlike linear regression, it never spits out a nonsense prediction like a negative probability.

What the company can responsibly conclude: Contract length matters a lot — locking people into longer contracts really does reduce churn. Fiber optic customers churn more than everyone else, and that showed up consistently across all three models, so it's not a fluke. New customers (under about 10 months) are the highest-risk group and are worth targeting with retention offers.

What the company should not conclude yet: None of this proves causation, just correlation. We can't say "switch people off fiber and they'll stop leaving" — fiber customers might just be a different kind of customer to begin with (pricier plan, higher expectations, whatever). Also, the model still misses over 40% of people who actually churn, so it shouldn't be trusted as a precise, one-to-one predictor without more tuning.

One next analysis we would run: Try lowering the probability cutoff for flagging someone as "at risk" below 0.5. Missing a real churner probably costs the company more than wasting a retention offer on someone who was going to stay anyway, so it's worth trading some false positives for catching more real ones.
