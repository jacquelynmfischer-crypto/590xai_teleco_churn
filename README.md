# Name: Jacquelyn Fischer

## Dataset

This project uses the Telco Customer Churn dataset from Kaggle: 7,043 customers from a telecommunications company, including demographic information, account details (tenure, contract type, payment method), and the services each customer is subscribed to (phone, internet, streaming, and various security/support add-ons). The target variable is whether a customer churned, meaning they left for a competitor; approximately 26.5% of customers in the dataset did. The objective is to identify which factors are associated with churn and to build models that can flag at-risk customers for retention outreach.

## Assumption Checks

| Model | Key Assumptions Checked | Evidence | Concern |
|---|---|---|---|
| Linear regression | No severe multicollinearity; normally distributed residuals; predictions should represent valid probabilities | VIF values were all below 5 after removing MonthlyCharges and TotalCharges; the Omnibus and Jarque-Bera tests both returned p ≈ 0.000; test set predictions ranged from -0.22 to 0.77 | The residuals are not normally distributed, and the model produced a negative predicted probability (-0.22), which is not a valid outcome for this target |
| Logistic regression | No severe multicollinearity; linearity of the logit, meaning each feature's effect should be constant on the log-odds scale | Same VIF results as above; the GAM's results show that tenure's true effect on churn is not constant | Tenure's actual relationship with churn is curved (a steep decline early on, followed by a plateau), so the logistic model's single coefficient for tenure only approximates the true pattern |
| GAM | Similar multicollinearity considerations, along with an assumption that the fitted relationships are smooth | The spline term for tenure had an Effective Degrees of Freedom (EDoF) of 13.4, compared to 1 for a straight line, confirming genuine non-linearity; the partial dependence plot shows a steep decline in the first 10 months, a plateau through the middle tenure range, and a further decline later on | The pyGAM library's own output notes that its p-values are unreliable; a few smaller fluctuations in the fitted curve (around months 40 and 60) may reflect noise rather than a true pattern, given less data in those ranges |

## Model Comparison

| Model | Performance Evidence | Interpretability Strength | Interpretability Weakness |
|---|---|---|---|
| Linear regression | Test R² = 0.304, Test RMSE = 0.368 | Coefficients are directly interpretable as changes in predicted probability | Predictions are not bounded between 0 and 1, and residuals do not meet the normality assumption |
| Logistic regression | Test accuracy of 82.1% (compared to a 73.5% baseline of predicting no churn for every customer), AUC of 0.861 | Coefficients convert into odds ratios that are straightforward to explain (for example, a two-year contract corresponds to roughly a 76% reduction in churn odds); predictions are always valid probabilities | Assumes each feature has a constant effect, which understates the elevated risk among new customers; the model missed 42% of customers who actually churned |
| GAM | Test accuracy of 81.8%, AUC of 0.864, effectively equivalent to logistic regression | Captures the true, non-linear shape of tenure's effect rather than reducing it to a single number | More difficult to explain to a non-technical audience than an odds ratio; the added flexibility did not produce a meaningful improvement in predictive performance |

## Recommendation

Recommended model: Logistic Regression

Why this model: Logistic regression performs essentially the same as the GAM (82.1% versus 81.8% accuracy, 0.861 versus 0.864 AUC), but is considerably easier to explain to a non-technical audience through odds ratios, such as stating that a two-year contract reduces the odds of churn by roughly 76%. It is also easier to maintain: it trains almost instantly using a mature, well-supported library, and has no smoothing parameters that would need to be retuned as new data comes in. The GAM, by comparison, relies on a smaller and less mature library that flags its own p-value calculations as unreliable in its output. Given that the GAM does not offer a meaningful performance advantage, this additional complexity is not justified. Linear regression, meanwhile, can produce predictions outside the valid range for a probability, which makes it unsuitable regardless of its ease of interpretation.

What the company can responsibly conclude: Contract length is strongly associated with churn; longer contracts meaningfully reduce the likelihood that a customer leaves. Customers with fiber optic internet churn at a notably higher rate than other customers, and this finding was consistent across all three models. Customers within their first ten months are at the highest risk and are reasonable candidates for retention outreach.

What the company should not conclude yet: These results reflect correlation, not causation. It cannot be concluded that moving a customer off fiber internet would reduce their likelihood of churning, since fiber customers may differ from other customers in ways the data does not capture, such as price sensitivity or service expectations. The model also misses more than 40% of customers who actually churn, so it should not be treated as a precise predictor for individual customers without further tuning.

One next analysis we would run: Test a lower probability threshold than 0.5 for flagging at-risk customers. Missing an actual churner is likely more costly to the business than extending a retention offer to a customer who would have stayed regardless, so it may be worth accepting more false positives in exchange for identifying more true churners.
