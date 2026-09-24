# Customer Profiling for Treadmill Products

Who buys which treadmill, and how can sales and marketing use that?

This project profiles 180 customers of a fitness equipment company across its three treadmill models (KP281, KP481, KP781). It turns the profiles into simple, data-backed rules for recommending a model to a new customer and for finding groups that marketing is missing.

**Type:** Exploratory data analysis, customer segmentation, applied probability
**Stack:** Python, pandas, NumPy, seaborn, matplotlib

---

## Key findings

- **KP781 buyers are a clearly separate group.** 86% of customers earning above $65K a year bought KP781, against 22% overall. The same pattern holds for customers with 18+ years of education (85%) and customers who plan 5 or more sessions a week (81%).
- **Some customers never buy KP781.** Not one customer with income of $45K or less, planned usage of 2 or fewer sessions a week, or self-rated fitness of 1 or 2 bought it.
- **KP281 and KP481 buyers look almost the same.** Their age, education, usage, fitness and miles overlap heavily. Income is the one clear split: 69% of customers earning $45K or less chose KP281.
- **Gender matters only for KP781.** 32% of men bought KP781 against 9% of women. For KP281 and KP481 the gender split is close to even.
- **Marital status has no effect.** Single and partnered customers buy the three models in the same ratio as the overall mix.

---

## Business problem

A fitness equipment company sells three treadmill models and wants to know:

1. What does a typical buyer of each model look like?
2. Given a new customer's profile, which model are they most likely to buy?
3. Which customer groups are under-served?

The output has to be simple enough for a sales team to use in a conversation with a customer. So the analysis focuses on clear segments and conditional probabilities that anyone can read, rather than a model whose output is hard to explain.

---

## Data

180 purchase records, 9 columns, no missing values.

| Column | Type | Description |
|---|---|---|
| `product` | category | KP281, KP481 or KP781 |
| `age` | int | Age in years (18 to 50) |
| `gender` | category | Male / Female |
| `education` | int | Years of education (12 to 21) |
| `marital_status` | category | Single / Partnered |
| `usage` | int | Planned treadmill sessions per week (2 to 7) |
| `fitness` | int | Self-rated fitness, 1 (poor) to 5 (excellent) |
| `income` | int | Annual income in $ ($29.6K to $104.6K) |
| `miles` | int | Planned miles per week (21 to 360) |

**Sales mix:** KP281 80 (44%), KP481 60 (33%), KP781 40 (22%).

---

## Approach

### 1. Data quality and outlier review

- Checked for nulls and fixed data types (`product`, `gender` and `marital_status` cast to categorical).
- Flagged outliers in every numeric column with the 1.5 x IQR rule, then reviewed the flagged rows one by one instead of dropping them.

**Why the outliers were kept:** 34 of 180 rows fall outside the IQR bounds on at least one column, and 27 of those are KP781 buyers. All 19 income outliers and all 9 usage outliers are KP781 buyers. These are not data errors. They are the most active, highest-income customers in the data. A blanket IQR filter would have removed 27 of the 40 KP781 buyers and hidden the strongest signal in the dataset.

### 2. Distribution analysis per product

Compared every feature across the three products using KDE plots, labelled histograms and count plots. Small reusable helpers (`plot_hist`, `plot_bar`) keep the per-product views consistent.

### 3. Correlation analysis

Pearson correlation across the numeric features:

- `usage`, `fitness` and `miles` move together strongly (0.67 to 0.79). Together they describe one thing: how active the customer is.
- `education` and `income` are linked (0.63).
- `age` has almost no link with usage, fitness or miles (all below 0.07), so age is a weak signal for product choice.
- There are no negative correlations.

### 4. Feature binning

Each numeric feature was split into low / medium / high buckets. Cut points were picked from each feature's distribution.

| Feature | Low | Medium | High |
|---|---|---|---|
| Age | 25 or less | 26 to 40 | 41 and above |
| Education (years) | 14 or less | 15 to 17 | 18 and above |
| Usage (sessions/week) | 2 or less | 3 to 4 | 5 to 7 |
| Fitness (1 to 5) | 1 to 2 | 3 | 4 to 5 |
| Income | $45K or less | $45K to $65K | above $65K |
| Miles (per week) | 50 or less | 51 to 120 | above 120 |

### 5. Contingency tables and probabilities

Built a two-way table of every bucketed feature against `product` and computed:

- **Joint and marginal probabilities**, for example P(Male and KP781) = 0.18 and P(KP781) = 0.22.
- **P(product | segment)** for every feature. This answers the sales question: "for a customer like this, which model?"
- **P(gender | product)**. This answers the marketing question: "who is buying this model?"

---

## Results

### Chance of buying each model, by customer segment

Overall mix for comparison: **KP281 44%, KP481 33%, KP781 22%**.

| Segment | Customers | KP281 | KP481 | KP781 |
|---|---:|---:|---:|---:|
| Income above $65K | 28 | 7% | 7% | **86%** |
| Education 18+ years | 27 | 7% | 7% | **85%** |
| Usage 5 to 7 per week | 26 | 8% | 12% | **81%** |
| Miles above 120 per week | 42 | 14% | 19% | **67%** |
| Fitness 4 to 5 | 55 | 20% | 15% | **65%** |
| Income $45K to $65K | 103 | 43% | 42% | 16% |
| Male | 104 | 38% | 30% | 32% |
| Female | 76 | 53% | 38% | 9% |
| Single | 73 | 44% | 33% | 23% |
| Partnered | 107 | 45% | 34% | 21% |
| Fitness 1 to 2 | 28 | 54% | 46% | 0% |
| Usage 2 or less per week | 33 | 58% | 42% | 0% |
| Income $45K or less | 49 | **69%** | 31% | 0% |
| Miles 50 or less per week | 17 | **71%** | 29% | 0% |

### Buyer profiles

| | KP281 | KP481 | KP781 |
|---|---|---|---|
| Share of sales | 44% (80) | 33% (60) | 22% (40) |
| Income range | $29.6K to $68.2K | $31.8K to $67.1K | $48.6K to $104.6K |
| Low-income buyers ($45K or less) | 43% | 25% | 0% |
| Age | 18 to 50, peak in mid 20s | 19 to 48, peaks in mid 20s and early 30s | 22 to 48, 30 of 40 aged 22 to 30 |
| Education | 14 or 16 years (69 of 80) | 14 or 16 years (54 of 60) | 16+ years (38 of 40) |
| Usage per week | 2 to 4 (78 of 80) | 2 to 4 (57 of 60) | 4 or more (39 of 40) |
| Fitness | mostly 3 (54 of 80) | mostly 3 (39 of 60), none at 5 | mostly 5 (29 of 40), none below 3 |
| Miles per week | 51 to 120 for 62 of 80 | 51 to 120 for 47 of 60 | above 120 for 28 of 40 |
| Gender | 50% male | 52% male | 33 of 40 male |
| Marital status | 60% partnered | 60% partnered | 58% partnered |

**In short:**

- **KP281:** lower to middle income, average fitness, about 3 sessions a week. The largest group and the default choice.
- **KP481:** the same activity profile as KP281, but with fewer low-income buyers (25% against 43%).
- **KP781:** high income, more years of education, very active and very fit, mostly men.

---

## Recommendations

**For sales**

1. **Lead with KP781** for customers who earn above $65K, have 18+ years of education, or plan 5+ sessions a week. Each of these on its own points to KP781 in 81% to 86% of cases.
2. **Do not lead with KP781** for customers who plan 2 or fewer sessions a week, rate their fitness 1 or 2, or earn $45K or less. None of these customers bought it. Start with KP281.
3. **In the middle income band ($45K to $65K)**, KP281 and KP481 are close to a coin toss (43% against 42%). The customer profile alone cannot pick the model here, so the pitch has to explain clearly what KP481 adds over KP281.

**For marketing**

4. **Women are under-represented among KP781 buyers.** Only 7 of 40 KP781 buyers are women, and 9% of women bought it against 32% of men. This is worth testing as a growth area, for example with campaigns aimed at fit, high-usage women. First check whether income or usage differences explain the gap (see limitations).
5. **Drop marital status as a targeting field.** It does not change the product mix. Age changes the mix only for customers over 40, a group of just 12 people, so it is also a weak targeting field.

**For product**

6. **KP281 and KP481 compete for the same customer.** Their buyer profiles overlap on almost every feature. The company should check whether the two models are different enough to justify both, or position KP481 more clearly as a step up.

---

## Limitations and next steps

**Limitations**

- **Small sample.** 180 customers in total, and some segments are small (17 customers with 50 or fewer miles a week, 12 aged over 40). Percentages for small segments can shift a lot with a few more records.
- **Hand-picked bucket edges.** Cut points were chosen by looking at the distributions. Different edges can change the numbers.
- **One feature at a time.** Each table looks at a single feature against product. For example, the gender gap for KP781 may partly come from differences in income or usage. This analysis does not control for that.
- **Planned, not actual, activity.** `usage` and `miles` are what customers expect to do, not measured behavior.

**Next steps**

- Chi-square tests of independence to confirm which feature and product links are statistically significant.
- A small, explainable classifier (decision tree or multinomial logistic regression) to turn these profiles into one recommendation rule, with cross-validated accuracy measured against the 44% baseline of always picking KP281.
- Look at features together (for example gender within each income band) to separate real effects from overlap between features.