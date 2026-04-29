---
name: r-stats
description: >
  Use this skill whenever the user wants to perform statistical analysis using R. This includes:
  descriptive statistics, regression (linear/logistic/multiple), hypothesis testing (t-test, ANOVA,
  chi-square, Wilcoxon), time series analysis and forecasting, data exploration and visualization,
  correlation analysis, and any task involving running R code on data. Trigger when the user mentions
  R, statistics, regression, p-value, ANOVA, forecasting, or asks to "analyze my data", "run stats",
  "test significance", or uploads a CSV/Excel file for analysis. Also trigger when the user pastes
  tabular data and asks for statistical insights, or provides an R script to execute. Always use
  this skill rather than approximating statistical output from memory — actual R execution is required.
---

# R Statistical Analysis Skill

## Overview

This skill executes real R code in the local environment, captures results, and provides
expert statistical interpretation. It supports:

- **Descriptive stats / EDA**: summary, distributions, outliers, correlations
- **Regression**: linear, multiple, logistic, polynomial
- **Hypothesis testing**: t-test, ANOVA, chi-square, Wilcoxon, Shapiro-Wilk, etc.
- **Time series**: decomposition, ARIMA, forecasting
- **Data sources**: CSV/Excel uploads, user-pasted data, R built-in datasets, database outputs

---

## Step 0 — Environment Setup (Run Once Per Session)

> **IMPORTANT: Always use script files to run R code.** Never use `Rscript -e "..."` —
> inline `-e` strings have escaping issues in Windows/Git Bash environments. Instead,
> write R code to a temp file and execute it with `Rscript /tmp/script.R`.

### 1. Verify R is available
```bash
which Rscript && Rscript --version
```

If missing, install:
```bash
apt-get install -y r-base 2>&1 | tail -3
```

### 2. Check available packages
```bash
cat > /tmp/check_pkgs.R << 'EOF'
cat("Installed packages:", paste(installed.packages()[,"Package"], collapse=", "), "\n")
EOF
Rscript /tmp/check_pkgs.R 2>&1
```

### 3. Package strategy — Base R First

**Base R (`stats`, `graphics`, `utils`) is always available** and covers ~80% of needs:

| Need | Base R | External package |
|------|--------|-----------------|
| Summary stats | `summary()`, `var()`, `sd()` | `psych::describe()` |
| Regression | `lm()`, `glm()` | `car` for VIF |
| Hypothesis tests | `t.test()`, `chisq.test()`, `aov()` | `FSA` for post-hoc |
| Correlation | `cor()`, `cor.test()` | `corrplot` for viz |
| Time series | `ts()`, `decompose()`, `arima()` | `forecast::auto.arima()` |
| Plots | `plot()`, `hist()`, `boxplot()` | `ggplot2` |
| Read CSV | `read.csv()` | `readr::read_csv()` |

**Always attempt base R first.** Try external packages only if needed:
```bash
cat > /tmp/install_pkgs.R << 'EOF'
pkgs <- c('readr','readxl','dplyr','ggplot2','forecast','lmtest','car','psych')
missing <- pkgs[!pkgs %in% installed.packages()[,'Package']]
if(length(missing) > 0) {
  tryCatch(
    install.packages(missing, repos='https://cloud.r-project.org', quiet=TRUE),
    error = function(e) cat('Note: Could not install packages (network restricted). Using base R.\n')
  )
}
cat('Available:', paste(pkgs[pkgs %in% installed.packages()[,'Package']], collapse=', '), '\n')
EOF
Rscript /tmp/install_pkgs.R 2>&1
```

> ⚠️ In network-restricted environments, CRAN may be unavailable. All templates below include base R alternatives.

---

## Step 1 — Data Ingestion

Determine data source and load accordingly.

### A) CSV / Excel upload
```r
# File will be at /mnt/user-data/uploads/<filename>
library(readr); library(readxl)

# CSV
df <- read_csv("/mnt/user-data/uploads/data.csv")

# Excel
df <- read_excel("/mnt/user-data/uploads/data.xlsx", sheet = 1)

# Quick peek
cat("Shape:", nrow(df), "rows x", ncol(df), "cols\n")
glimpse(df)
```

### B) User-pasted data
```r
# Write paste to temp file first, then read
data_text <- '
col1,col2,col3
1,2,3
4,5,6
'
df <- read.csv(text = trimws(data_text))
```

### C) User-provided R script
```bash
# Save script to temp file, then run it
cat > /tmp/user_script.R << 'EOF'
# <user script contents here>
EOF
Rscript /tmp/user_script.R 2>&1
```

### D) R built-in datasets (for demos/testing)
```r
data(mtcars)   # cars
data(iris)     # flowers
data(AirPassengers)  # time series
data(Titanic)  # survival
```

---

## Step 2 — Choose Analysis Path

Read the user's goal and pick the right reference section:

| Goal | Reference Section |
|------|-------------------|
| "Describe / explore / summarize my data" | → [Descriptive Stats](#descriptive) |
| "Predict Y from X", "regression", "correlation" | → [Regression](#regression) |
| "Is there a significant difference?", "compare groups" | → [Hypothesis Testing](#hypothesis) |
| "Forecast", "trend", "time series" | → [Time Series](#timeseries) |
| User provides their own R script | → [Run User Script](#userscript) |

---

## Step 3 — Analysis Templates

**Execution pattern for ALL templates below:**
```bash
cat > /tmp/analysis.R << 'EOF'
# Paste the R template here
EOF
Rscript /tmp/analysis.R 2>&1
```

Always write the R code to a temp file and execute via `Rscript /tmp/analysis.R`.
Never use `Rscript -e "..."` as it breaks in Windows/Git Bash environments.

### <a name="descriptive"></a> Descriptive Statistics / EDA

```r
# --- Base R approach (always works) ---
cat("=== Shape ===\n")
cat(nrow(df), "rows x", ncol(df), "cols\n")

cat("\n=== Summary ===\n")
print(summary(df))

cat("\n=== Missing Values ===\n")
print(colSums(is.na(df)))

# Numeric columns only
num_df <- df[, sapply(df, is.numeric), drop = FALSE]

cat("\n=== Correlation Matrix ===\n")
if (ncol(num_df) >= 2) print(round(cor(num_df, use = "complete.obs"), 3))

# Per-column stats with skewness (base R)
skewness <- function(x) {
  x <- x[!is.na(x)]
  n <- length(x); m <- mean(x); s <- sd(x)
  (sum((x - m)^3) / n) / s^3
}
kurtosis <- function(x) {
  x <- x[!is.na(x)]
  n <- length(x); m <- mean(x); s <- sd(x)
  (sum((x - m)^4) / n) / s^4 - 3
}

cat("\n=== Per-column Distribution Stats ===\n")
for (col in names(num_df)) {
  v <- num_df[[col]]
  cat(sprintf("\n--- %s ---\n", col))
  cat(sprintf("  n=%d, mean=%.3f, sd=%.3f, median=%.3f\n",
              sum(!is.na(v)), mean(v,na.rm=T), sd(v,na.rm=T), median(v,na.rm=T)))
  cat(sprintf("  min=%.3f, max=%.3f, skew=%.3f, kurt=%.3f\n",
              min(v,na.rm=T), max(v,na.rm=T), skewness(v), kurtosis(v)))
}
```

---

### <a name="regression"></a> Regression Analysis

#### Linear Regression
```r
library(car); library(lmtest)

# Fit model — replace outcome/predictors as needed
model <- lm(outcome ~ predictor1 + predictor2, data = df)
cat("=== Model Summary ===\n"); print(summary(model))

# Diagnostics
cat("\n=== Variance Inflation Factors (multicollinearity) ===\n")
if (length(coef(model)) > 2) print(vif(model))

cat("\n=== Breusch-Pagan Test (heteroscedasticity) ===\n")
print(bptest(model))

cat("\n=== Shapiro-Wilk on Residuals (normality) ===\n")
if (nrow(df) <= 5000) print(shapiro.test(residuals(model)))

# Save diagnostic plots
png("/tmp/regression_diagnostics.png", width=1200, height=900)
par(mfrow=c(2,2)); plot(model)
dev.off()
```

#### Logistic Regression
```r
# Binary outcome
model <- glm(outcome ~ predictor1 + predictor2,
             data = df, family = binomial(link = "logit"))
cat("=== Logistic Regression Summary ===\n"); print(summary(model))

# Odds ratios
cat("\n=== Odds Ratios (95% CI) ===\n")
print(round(exp(cbind(OR = coef(model), confint(model))), 3))
```

---

### <a name="hypothesis"></a> Hypothesis Testing

```r
# --- t-test (two groups) ---
# t.test(df$value ~ df$group)           # independent
# t.test(df$before, df$after, paired=TRUE)  # paired

# --- One-way ANOVA ---
# model <- aov(value ~ group, data = df)
# summary(model)
# TukeyHSD(model)  # post-hoc

# --- Chi-square test ---
# tbl <- table(df$var1, df$var2)
# chisq.test(tbl)

# --- Wilcoxon (non-parametric alternative to t-test) ---
# wilcox.test(value ~ group, data = df)

# --- Shapiro-Wilk normality test ---
# shapiro.test(df$column)

# --- Levene's test for equal variances ---
# library(car); leveneTest(value ~ group, data = df)
```

> Select the correct test based on:
> - Data type (continuous vs categorical)  
> - Number of groups (2 vs 3+)
> - Normality (use Shapiro-Wilk first; if p < 0.05 → non-parametric)
> - Paired vs independent

See `references/test-selection.md` for a full decision tree.

---

### <a name="timeseries"></a> Time Series / Forecasting

```r
library(forecast); library(lubridate)

# Convert to ts object — adjust frequency:
# 12 = monthly, 4 = quarterly, 52 = weekly, 365 = daily
ts_data <- ts(df$value, start = c(2020, 1), frequency = 12)

# Decompose
cat("=== STL Decomposition ===\n")
decomp <- stl(ts_data, s.window = "periodic")
print(summary(decomp))

# Save decomposition plot
png("/tmp/ts_decomposition.png", width = 1000, height = 700)
plot(decomp)
dev.off()

# Auto ARIMA
cat("\n=== Auto ARIMA Model ===\n")
arima_model <- auto.arima(ts_data, stepwise = FALSE, approximation = FALSE)
print(summary(arima_model))

# Forecast 12 periods
fc <- forecast(arima_model, h = 12)
cat("\n=== 12-Period Forecast ===\n")
print(fc)

png("/tmp/ts_forecast.png", width = 1000, height = 500)
plot(fc)
dev.off()
```

---

### <a name="userscript"></a> Running User-Provided R Scripts

When a user pastes or uploads an R script:

```bash
# 1. Save to temp file
cat > /tmp/user_analysis.R << 'RSCRIPT'
<paste script content here>
RSCRIPT

# 2. Run with stdout+stderr captured
Rscript /tmp/user_analysis.R 2>&1

# 3. Check for output files
ls /tmp/*.png /tmp/*.csv /tmp/*.rds 2>/dev/null
```

If the script produces errors:
- Read the error, fix obvious issues (missing packages, path problems)
- Re-run and confirm success before interpreting

---

## Step 4 — Output & Capture

### Text output
All `print()`, `cat()`, `summary()` output is captured by bash_tool automatically.

### Plots / images
```bash
# Plots saved to /tmp/ should be copied to outputs
cp /tmp/*.png /mnt/user-data/outputs/ 2>/dev/null
```
Then use `present_files` to show them to the user.

### Saving data results
```r
# Save processed data
write.csv(results_df, "/tmp/results.csv", row.names = FALSE)
```

---

## Step 5 — Interpretation Guidelines

After capturing R output, always provide:

1. **Plain-language summary**: What did the analysis find? (no jargon)
2. **Key statistics highlighted**: Effect sizes, p-values, R², confidence intervals
3. **Statistical significance**: Is p < 0.05? What does it mean in context?
4. **Assumptions check**: Were test assumptions met? (normality, homoscedasticity, etc.)
5. **Limitations**: Sample size issues, potential confounders, data quality notes
6. **Recommendations**: Next steps, alternative analyses to consider

> Always caveat that statistical significance ≠ practical significance. Report effect sizes.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `package not found` | Run install.packages() in Step 0 |
| `object not found` | Check variable names with `names(df)` |
| Memory error on large file | Use `data.table::fread()` instead of `read_csv()` |
| Plot not saving | Check `/tmp/` permissions; try `pdf()` instead of `png()` |
| Encoding issue (CSV) | Add `fileEncoding = "UTF-8"` to read_csv() |
| R not found | Run apt-get install -y r-base |
| `Rscript -e "..."` fails with quoting errors | **Never use `-e` on Windows/Git Bash.** Write code to a temp file with `cat > file.R << 'EOF'` and run `Rscript file.R` instead. |
| `Rscript --version` works but inline code fails | Shell escaping issue — use script file pattern instead of inline `-e` flag. |
| Error: unrecognized escape in character string | Same quoting issue — use script file pattern. |

For detailed test selection logic, see `references/test-selection.md`.  
For ggplot2 visualization templates, see `references/visualization.md`.
