# Probability & Statistics (WIA2003)

> Measuring uncertainty and making decisions from data: from coin tosses to hypothesis tests in SPSS.

**Topics:** [Probability Basics](#1-probability-basics) · [Conditional Probability & Bayes](#2-conditional-probability-independence--bayes-theorem) · [Discrete Random Variables](#3-discrete-random-variables--distributions) · [Continuous Random Variables](#4-continuous-random-variables--the-normal-distribution) · [Intro to Statistics](#5-introduction-to-statistics-data-types--scales) · [Descriptive Statistics](#6-descriptive-statistics) · [Hypothesis Testing](#7-hypothesis-testing) · [t-tests & ANOVA](#8-t-tests--anova) · [Correlation & Regression](#9-correlation--simple-linear-regression) · [Non-parametric Tests](#10-non-parametric-tests) · [Which Test?](#which-test-should-i-use)

---

## 1. Probability Basics

**Probability** = how likely an event is, from **0** (impossible) to **1** (certain).

- **Experiment:** a process with uncertain outcomes (toss a coin).
- **Sample space S:** all possible outcomes. Toss 3 coins: `S = {HHH, HHT, HTH, HTT, THH, THT, TTH, TTT}` (8 outcomes).
- **Event:** a subset of S. "Exactly 2 heads" = {HHT, HTH, THH}.

**Classical probability** (equally likely outcomes):
```
P(A) = number of outcomes in A / total outcomes      P(exactly 2 heads) = 3/8 = 0.375
```
> 🎲 **Analogy:** probability is "what fraction of the time this would happen if you repeated the experiment forever".

### Rules
| Rule | Formula |
|---|---|
| Complement | `P(A') = 1 − P(A)` |
| Addition (general) | `P(A ∪ B) = P(A) + P(B) − P(A ∩ B)` |
| Mutually exclusive (can't both happen) | `P(A ∩ B) = 0`, so `P(A ∪ B) = P(A) + P(B)` |
| Multiplication | `P(A ∩ B) = P(A) · P(B|A)` |

Counting tools ([permutations and combinations](../year1-sem1/CM-computing-mathematics.md#8-counting)) help count outcomes:
*P(drawing 2 aces from a deck)* = `C(4,2) / C(52,2) = 6/1326 ≈ 0.0045`.

---

## 2. Conditional Probability, Independence & Bayes' Theorem

**Conditional probability:** the probability of A **given** that B happened:
```
P(A|B) = P(A ∩ B) / P(B)
```
> 🌧️ P(traffic jam) might be 0.2, but P(traffic jam | heavy rain) might be 0.7. Knowing it's raining changes the odds.

**Independent events:** knowing B tells you nothing about A: `P(A|B) = P(A)`, equivalently `P(A ∩ B) = P(A)·P(B)`.
*Example:* two coin tosses are independent; drawing two cards without replacement is not.

### Law of total probability
If B₁, B₂, … split the sample space:
```
P(A) = Σ P(A|Bᵢ) · P(Bᵢ)
```

### Bayes' theorem
Reverse the condition: from P(A|B) to P(B|A).
```
P(Bᵢ|A) = P(A|Bᵢ) · P(Bᵢ) / Σⱼ P(A|Bⱼ) · P(Bⱼ)
```
**Worked example (factory defects):** machine A makes 60% of parts (3% defective); machine B makes 40% (5% defective). A part is defective. Which machine probably made it?
```
P(D) = 0.6×0.03 + 0.4×0.05 = 0.018 + 0.020 = 0.038
P(A|D) = 0.018 / 0.038 ≈ 0.474
P(B|D) = 0.020 / 0.038 ≈ 0.526   → more likely machine B
```
**Tree diagrams** make these calculations easy: multiply along the branches, add across the branches that end in the event you want.

> 🕵️ **Analogy:** Bayes is how a detective updates suspicion. Start with prior odds, then adjust based on how well each suspect explains the clue.

---

## 3. Discrete Random Variables & Distributions

A **random variable X** assigns a number to each outcome. It's **discrete** if it takes countable values (0, 1, 2…).

**Probability distribution:** a table or formula giving `P(X = x)`. All probabilities must be ≥ 0 and **sum to 1**.

| x (adults in home) | 1 | 2 | 3 | 4+ |
|---|---|---|---|---|
| P(x) | 0.25 | 0.50 | 0.15 | **?** |
→ `P(4+) = 1 − 0.90 = 0.10`

### Mean (expected value) and variance
```
E(X) = μ = Σ x · P(x)
Var(X) = σ² = Σ (x − μ)² P(x) = Σ x² P(x) − μ²
σ = √Var(X)
```
*Example:* hits per player: x = 0,1,2,3,4 with P = 0.10, 0.20, 0.30, 0.25, 0.15
`E(X) = 0 + 0.20 + 0.60 + 0.75 + 0.60 = 2.15` hits.

> 🎰 **Analogy:** the expected value is the long-run average, like the average winnings per spin if you played a slot machine a million times.

### Binomial distribution
**n** independent trials, each with **two outcomes** (success/failure) and the **same probability p** of success.
```
P(X = k) = C(n,k) · pᵏ · (1 − p)ⁿ⁻ᵏ
Mean = np,   Variance = np(1 − p)
```
*Example:* roll a fair die 4 times. P(exactly 2 twos)? n = 4, p = 1/6:
`C(4,2)(1/6)²(5/6)² = 6 × 0.0278 × 0.694 ≈ 0.116`
*P(no twos)* = `(5/6)⁴ ≈ 0.482`.

### Poisson distribution
Counts events in a fixed interval (time, area) when they happen independently at an average rate **λ**.
```
P(X = k) = e^(−λ) λᵏ / k!         Mean = Variance = λ
```
*Example:* a call centre gets 3 calls/minute on average. P(exactly 5 calls in a minute) = `e⁻³ 3⁵ / 5! ≈ 0.101`.

> ☎️ Binomial = "how many successes out of n tries". Poisson = "how many events in this time window" (no fixed n).

```python
from scipy.stats import binom, poisson
binom.pmf(2, n=4, p=1/6)     # 0.1157
poisson.pmf(5, mu=3)         # 0.1008
```

---

## 4. Continuous Random Variables & the Normal Distribution

A **continuous** random variable can take any value in a range (height, time, weight). `P(X = exact value) = 0`; we only talk about probabilities over **intervals**.

### Probability density function (PDF) f(x)
- `f(x) ≥ 0`
- Total area under the curve = 1: `∫ f(x) dx = 1`
- `P(a ≤ X ≤ b) = ∫ₐᵇ f(x) dx` (the area between a and b)

### Cumulative distribution function (CDF)
```
F(x) = P(X ≤ x) = ∫₋∞ˣ f(t) dt         and   f(x) = F'(x)
P(a < X < b) = F(b) − F(a)
```
### Mean and variance
```
E(X) = ∫ x f(x) dx          Var(X) = ∫ x² f(x) dx − μ²
```

**Worked example:** `f(x) = a x²` for 0 ≤ x ≤ 5.
1. Find a: `∫₀⁵ a x² dx = a · 125/3 = 1` → `a = 3/125`.
2. CDF: `F(x) = x³/125` for 0 ≤ x ≤ 5.
3. `E(X) = ∫₀⁵ x · (3/125) x² dx = (3/125)(625/4) = 3.75`.
4. `P(X ≥ 2) = 1 − F(2) = 1 − 8/125 = 0.936`.

> 💧 **Analogy:** for discrete variables probability sits in "buckets" at specific values. For continuous variables it's spread out like sand; you can only measure how much sand is in a stretch, not at a single point.

### The normal distribution
The famous **bell curve**, described by its mean μ and standard deviation σ: `X ~ N(μ, σ²)`.
- Symmetric around μ; mean = median = mode.
- **Empirical rule (68–95–99.7):** about 68% of values fall within 1σ of μ, 95% within 2σ, and 99.7% within 3σ.

**Standardising (z-score):** `z = (x − μ) / σ` (how many standard deviations x is from the mean). Then use the standard normal table `Z ~ N(0,1)`.
*Example:* exam marks ~ N(65, 10²). P(mark > 80)? `z = 1.5` → `P(Z > 1.5) ≈ 0.0668` (about 6.7%).

---

## 5. Introduction to Statistics: Data Types & Scales

- **Population:** everyone/everything of interest. **Sample:** the subset you actually measure.
- **Parameter:** a number describing a population (μ, σ). **Statistic:** a number describing a sample (x̄, s).
- **Descriptive statistics:** summarise data. **Inferential statistics:** draw conclusions about a population from a sample.

> 🍲 **Analogy:** tasting a spoonful (sample) to judge the whole pot of soup (population). Stir well first (random sampling)!

### Types of variables
| Type | Description | Examples |
|---|---|---|
| **Categorical (qualitative)** | labels/groups | gender, race, smoker (yes/no) |
| **Numerical (quantitative)**, discrete | countable | number of pages, number of absences |
| **Numerical**, continuous | measurable, any value | age, blood pressure, weight, temperature |

### Measurement scales (NOIR)
| Scale | Properties | Example |
|---|---|---|
| **Nominal** | categories, no order | sex, blood type, race |
| **Ordinal** | ordered, but gaps not equal | rank, satisfaction (1–5 Likert), grade A/B/C |
| **Interval** | equal gaps, **no true zero** | temperature in °C (0 °C ≠ "no temperature") |
| **Ratio** | equal gaps **and** a true zero | weight (g), weight gain, height, income |

*Fish example from the tutorial:* sex → nominal; initial weight (g) → ratio; body temperature (°C) → interval; weight gain (g) → ratio.

---

## 6. Descriptive Statistics

### Measures of central tendency
| Measure | How | Best when |
|---|---|---|
| **Mean** x̄ | sum ÷ n | data is symmetric, no outliers |
| **Median** | the middle value when sorted | **skewed** data or **outliers** (incomes, waiting times) |
| **Mode** | the most frequent value | categorical data |

*Waiting times:* 2, 5, 5, 8, 9, 11, 16, 17, 22, 55 → mean = 15, median = (9+11)/2 = 10, mode = 5. The 55 drags the mean up, so the **median** is the better summary here.

> 🏠 **Analogy:** if a billionaire walks into a café, the *mean* wealth of the customers skyrockets, but the *median* barely changes.

### Measures of spread
| Measure | Formula / meaning |
|---|---|
| **Range** | max − min |
| **Quartiles** | Q1 (25th percentile), Q2 = median, Q3 (75th) |
| **IQR** | `Q3 − Q1` (the spread of the middle 50%; robust to outliers) |
| **Variance** (sample) | `s² = Σ(x − x̄)² / (n − 1)` |
| **Standard deviation** | `s = √s²` (typical distance from the mean) |

Why `n − 1`? Using the sample mean makes the deviations slightly too small; dividing by n − 1 corrects this bias (**Bessel's correction**).

**Five-number summary:** min, Q1, median, Q3, max → drawn as a **box plot**.
**Outliers:** values below `Q1 − 1.5×IQR` or above `Q3 + 1.5×IQR`.

**Shape:** symmetric, **right/positive skew** (long tail to the right: mean > median), left/negative skew (mean < median).

**Charts:** bar/pie charts (categorical), histograms, box plots, stem-and-leaf plots (numerical), scatter plots (two numerical variables).

```python
import numpy as np, pandas as pd
x = pd.Series([0,2,0,1,0,0,2,0,0,0,0,2,4,4,5,1,0,2,0,0,2,2,0])
x.describe()          # count, mean, std, min, 25%, 50%, 75%, max
x.mode()
```

---

## 7. Hypothesis Testing

A formal way to decide whether sample evidence is strong enough to reject a claim about a population.

> ⚖️ **Analogy:** a courtroom. The defendant is presumed **innocent** (H₀) until the evidence is strong enough to convict "beyond reasonable doubt" (p < α).

### Steps
1. **State the hypotheses:**
   - **H₀ (null):** no effect / no difference (`μ = 70`)
   - **H₁ (alternative):** there is an effect (`μ ≠ 70`, `μ > 70`, or `μ < 70`)
2. Choose the **significance level α** (usually 0.05).
3. Compute the **test statistic** (z, t, F, χ²…) from the sample.
4. Find the **p-value**: the probability of getting results at least this extreme **if H₀ were true**.
5. **Decide:** if `p < α` → **reject H₀** (statistically significant). If `p ≥ α` → **fail to reject H₀** (not "accept"!).
6. State the conclusion in context.

### One-tailed vs two-tailed
- **Two-tailed:** H₁: `μ ≠ μ₀`. Reject if the sample mean is significantly higher **or** lower.
- **Upper-tailed:** H₁: `μ > μ₀`. Reject if significantly **higher**.
- **Lower-tailed:** H₁: `μ < μ₀`. Reject if significantly **lower**.

### Errors
| | H₀ actually true | H₀ actually false |
|---|---|---|
| **Reject H₀** | ❌ **Type I error** (false positive), probability = α | ✅ correct (power = 1 − β) |
| **Fail to reject H₀** | ✅ correct | ❌ **Type II error** (false negative), probability = β |

> 🚨 **Type I** = convicting an innocent person / a fire alarm with no fire. **Type II** = letting a guilty person go / missing a real fire.

**Confidence interval** (95%): a range likely to contain the true parameter: `x̄ ± t* · s/√n`. If a 95% CI for the difference excludes 0, the two-tailed test at α = 0.05 is significant.

---

## 8. t-tests & ANOVA

Use a **t-test** to compare means when σ is unknown (almost always).

| Test | Question | Example |
|---|---|---|
| **One-sample t-test** | Is the sample mean different from a known value? | Is the average satisfaction score different from 3 (neutral)? |
| **Independent-samples t-test** | Do two **separate groups** differ? | Do male and female employees differ in satisfaction? |
| **Paired-samples t-test** | Do **the same subjects** differ between two conditions? | Scores before vs after training |
| **One-way ANOVA** | Do **3 or more** group means differ? | Satisfaction across 4 departments |

```
One-sample:  t = (x̄ − μ₀) / (s / √n),     df = n − 1
```
**Assumptions:** roughly normal data (or large n), independent observations; for the independent t-test, equal variances (check with **Levene's test**: if p < 0.05, use the "equal variances not assumed" row in SPSS).

**ANOVA** compares variance *between* groups with variance *within* groups (the **F statistic**). If significant, run **post-hoc tests** (Tukey) to find *which* groups differ.

### Reading SPSS output (typical tutorial task)
1. `Analyze → Descriptive Statistics → Frequencies` (counts, charts).
2. `Analyze → Compare Means → One-Sample / Independent-Samples / Paired-Samples T Test`.
3. Look at **Mean**, **SD**, **t**, **df**, and **Sig. (2-tailed)** (the p-value).
4. Write it up: *"Employees were significantly more satisfied with their work (M = 3.8, SD = 0.9) than the neutral value of 3, t(49) = 6.29, p < .001."*

```python
from scipy import stats
stats.ttest_1samp(scores, 3)
stats.ttest_ind(group_a, group_b, equal_var=False)
stats.ttest_rel(before, after)
stats.f_oneway(g1, g2, g3)
```

---

## 9. Correlation & Simple Linear Regression

### Correlation
Measures the **strength and direction** of the relationship between two variables ("co-relation").

**Pearson's r** ranges from **−1 to +1**:
- `+1`: perfect positive (both go up together)
- `0`: no linear relationship
- `−1`: perfect negative (one goes up, the other goes down)

Rough guide: |r| < 0.3 weak, 0.3–0.7 moderate, > 0.7 strong.

| Correlation type | Data |
|---|---|
| **Pearson product-moment** | two **continuous** variables, linear, roughly normal |
| **Spearman's rank (ρ)** | **ranked/ordinal** data, or non-normal |
| Point-biserial | one dichotomous (yes/no) + one continuous |
| Phi | two dichotomous |

⚠️ **Correlation ≠ causation.** Ice-cream sales and drowning are correlated because both rise in hot weather (a **confounding variable**), not because ice cream causes drowning.

### Simple linear regression
Find the **best-fit line** that predicts Y from X by minimising the squared distances from the points to the line (**least squares**).
```
Ŷ = bX + a          b = slope, a = intercept
b = Σ(x − x̄)(y − ȳ) / Σ(x − x̄)²        a = ȳ − b·x̄
```
| Role | Symbol | Other names |
|---|---|---|
| **Independent** variable | X | predictor, explanatory, regressor (IV) |
| **Dependent** variable | Y | response, outcome, criterion (DV) |

**R² (coefficient of determination):** the **proportion of variance in Y explained by X**. R² = 0.64 → 64% of the variation in exam score is explained by study hours. (In simple regression, R² = r².)

*Example:* `Score = 4.5 × Hours + 40`. A student who studies 8 hours is predicted to score `4.5×8 + 40 = 76`.

> 📏 **Analogy:** regression is laying a ruler across a scatter plot in the position that's "closest to all the dots at once", then using it to make predictions.

(Also see [ML: Linear Regression](../year1-sem2/ML-machine-learning.md#3-regression-linear--logistic).)

---

## 10. Non-parametric Tests

**Parametric tests** (t-test, ANOVA, Pearson) assume things about the population, e.g. **normal distribution** and **equal variances**.
**Non-parametric ("distribution-free") tests** make **no assumption** about the population's parameters or shape. They usually work on **ranks**.

**Use them when:**
- the normality or equal-variance assumptions are violated,
- the data is **ordinal or nominal** (e.g. Likert ranks),
- the sample is **small**,
- there are extreme outliers.

**Trade-off:** they're **less powerful** (less likely to detect a real effect) because ranking throws away some information.

**Decision rule for normality** (Shapiro–Wilk or Kolmogorov–Smirnov test):
- `p ≥ 0.05` → fail to reject H₀ → data looks normal → a **parametric** test is OK.
- `p < 0.05` → reject normality → use the **non-parametric equivalent**.

| Parametric test | Non-parametric equivalent |
|---|---|
| One-sample t-test | **Wilcoxon signed-rank** (one sample) / Sign test |
| Independent-samples t-test | **Mann–Whitney U** |
| Paired-samples t-test | **Wilcoxon signed-rank** |
| One-way ANOVA | **Kruskal–Wallis H** |
| Repeated-measures ANOVA | **Friedman** |
| Pearson correlation | **Spearman's rank** |
| (categorical association) | **Chi-square (χ²)** test of independence |

> 🏃 **Analogy:** instead of comparing runners' exact times (which might include a freak result), compare their finishing *positions*. You lose some detail but you're protected from weird values.

```python
stats.shapiro(data)                    # normality check
stats.mannwhitneyu(group_a, group_b)
stats.wilcoxon(before, after)
stats.kruskal(g1, g2, g3)
stats.spearmanr(x, y)
stats.chi2_contingency(table)
```

---

## Which test should I use?
```
What's your question?
├── Compare means / groups
│   ├── 1 group vs a known value ........ one-sample t   (non-param: Wilcoxon)
│   ├── 2 independent groups ............ independent t  (non-param: Mann–Whitney U)
│   ├── 2 related measurements .......... paired t       (non-param: Wilcoxon signed-rank)
│   └── 3+ groups ....................... one-way ANOVA  (non-param: Kruskal–Wallis)
├── Relationship between 2 variables
│   ├── both continuous, normal ......... Pearson r → regression for prediction
│   ├── ordinal / non-normal ............ Spearman ρ
│   └── both categorical ................ Chi-square
└── Check assumptions first: normality (Shapiro–Wilk), equal variances (Levene)
```

---

### ✅ Quick self-check
1. Toss 3 coins. P(at least one head)? *(1 − 1/8 = 7/8)*
2. What do the probabilities in a discrete distribution sum to? *(1)*
3. Which measure of centre is best for skewed data? *(median)*
4. p = 0.03 with α = 0.05: decision? *(reject H₀)*
5. Rejecting a true H₀ is which error? *(Type I)*
6. Non-parametric alternative to the independent t-test? *(Mann–Whitney U)*
7. R² = 0.81: what does it mean? *(81% of the variance in Y is explained by X)*
