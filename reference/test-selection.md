# Statistical Test Selection Guide

## Decision Tree

```
What is your research question?
│
├── Compare groups on a continuous outcome?
│   │
│   ├── 2 groups
│   │   ├── Independent? → Check normality (Shapiro-Wilk)
│   │   │   ├── Normal + equal variance → Independent t-test
│   │   │   ├── Normal + unequal variance → Welch t-test (var.equal=FALSE)
│   │   │   └── Non-normal → Mann-Whitney U (wilcox.test)
│   │   └── Paired/matched? →
│   │       ├── Normal → Paired t-test
│   │       └── Non-normal → Wilcoxon signed-rank
│   │
│   └── 3+ groups
│       ├── Independent → Check normality
│       │   ├── Normal + homoscedastic → One-way ANOVA → TukeyHSD post-hoc
│       │   ├── Normal + heteroscedastic → Welch ANOVA (oneway.test)
│       │   └── Non-normal → Kruskal-Wallis → Dunn test post-hoc
│       └── Repeated measures → Repeated-measures ANOVA / Friedman test
│
├── Relationship between two continuous variables?
│   ├── Both normal → Pearson correlation
│   └── Non-normal / ordinal → Spearman correlation
│
├── Predict a continuous outcome from predictors?
│   ├── One predictor → Simple linear regression
│   └── Multiple predictors → Multiple linear regression
│       └── Check: VIF < 5 (no multicollinearity), residual normality, homoscedasticity
│
├── Predict a binary outcome?
│   └── Logistic regression (glm with binomial family)
│
├── Association between two categorical variables?
│   ├── Expected counts ≥ 5 in all cells → Chi-square test
│   └── Small expected counts → Fisher's exact test
│
└── Time-ordered data / forecasting?
    ├── Trend + seasonality → STL decomposition
    ├── Stationary after differencing → ARIMA (auto.arima)
    └── Long seasonal period → TBATS or Prophet
```

---

## R Code Snippets by Test

### Normality Check
```r
shapiro.test(x)  # n < 5000 only
# p > 0.05 → normal; p < 0.05 → non-normal

# For larger samples:
ks.test(x, "pnorm", mean(x), sd(x))
```

### Homogeneity of Variance
```r
library(car)
leveneTest(value ~ group, data = df)
bartlett.test(value ~ group, data = df)
# p > 0.05 → equal variances
```

### Effect Sizes
```r
# Cohen's d (two groups)
library(effsize)
cohen.d(group1, group2)

# η² (ANOVA)
library(sjstats)
eta_sq(aov_model)

# For chi-square: Cramér's V
library(rcompanion)
cramerV(table(df$var1, df$var2))
```

### Post-hoc Tests
```r
# After ANOVA
TukeyHSD(aov_model)

# Non-parametric (after Kruskal-Wallis)
library(FSA)
dunnTest(value ~ group, data = df, method = "bonferroni")
```

---

## Power Analysis (Sample Size)
```r
library(pwr)

# t-test: how many per group needed?
pwr.t.test(d = 0.5, sig.level = 0.05, power = 0.80, type = "two.sample")

# ANOVA
pwr.anova.test(k = 3, f = 0.25, sig.level = 0.05, power = 0.80)

# Correlation
pwr.r.test(r = 0.3, sig.level = 0.05, power = 0.80)
```

---

## Reporting Standards

| Test | Report as |
|------|-----------|
| t-test | t(df) = X.XX, p = .XXX, d = X.XX |
| ANOVA | F(df1, df2) = X.XX, p = .XXX, η² = .XX |
| Chi-square | χ²(df, N=n) = X.XX, p = .XXX, V = .XX |
| Correlation | r(n) = .XX, p = .XXX |
| Regression | β = .XX, SE = .XX, t(df) = X.XX, p = .XXX |

Always report exact p-values (not "p < 0.05") and effect sizes alongside significance.
