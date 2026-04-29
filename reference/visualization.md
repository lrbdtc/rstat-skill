# R Visualization Templates (ggplot2)

## Setup
```r
library(ggplot2)
theme_set(theme_minimal(base_size = 13))
```

All plots should be saved to `/tmp/plot_name.png` and then copied to `/mnt/user-data/outputs/`.

---

## 1. Distribution Plots

### Histogram
```r
p <- ggplot(df, aes(x = value)) +
  geom_histogram(bins = 30, fill = "#4C9BE8", color = "white", alpha = 0.85) +
  geom_vline(aes(xintercept = mean(value, na.rm=TRUE)),
             color = "#E84C4C", linetype = "dashed", linewidth = 1) +
  labs(title = "Distribution of Value",
       subtitle = paste0("Mean = ", round(mean(df$value, na.rm=TRUE), 2)),
       x = "Value", y = "Count")
ggsave("/tmp/histogram.png", p, width = 8, height = 5, dpi = 150)
```

### Boxplot (groups)
```r
p <- ggplot(df, aes(x = group, y = value, fill = group)) +
  geom_boxplot(alpha = 0.8, outlier.color = "red") +
  geom_jitter(width = 0.15, alpha = 0.3, size = 1.5) +
  labs(title = "Value by Group", x = "Group", y = "Value") +
  theme(legend.position = "none")
ggsave("/tmp/boxplot.png", p, width = 8, height = 5, dpi = 150)
```

---

## 2. Relationship Plots

### Scatter with regression line
```r
p <- ggplot(df, aes(x = x_var, y = y_var)) +
  geom_point(alpha = 0.6, color = "#4C9BE8", size = 2) +
  geom_smooth(method = "lm", se = TRUE, color = "#E84C4C", linewidth = 1.2) +
  labs(title = "Scatter: X vs Y",
       subtitle = paste0("r = ", round(cor(df$x_var, df$y_var, use="complete.obs"), 3)),
       x = "X Variable", y = "Y Variable")
ggsave("/tmp/scatter.png", p, width = 7, height = 5, dpi = 150)
```

### Correlation heatmap
```r
library(reshape2)
corr_mat <- round(cor(df[, sapply(df, is.numeric)], use = "complete.obs"), 2)
melted <- melt(corr_mat)
p <- ggplot(melted, aes(Var1, Var2, fill = value)) +
  geom_tile(color = "white") +
  geom_text(aes(label = value), size = 3.5) +
  scale_fill_gradient2(low = "#E84C4C", mid = "white", high = "#4C9BE8",
                       midpoint = 0, limits = c(-1, 1)) +
  labs(title = "Correlation Matrix", x = "", y = "", fill = "r") +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
ggsave("/tmp/correlation_heatmap.png", p, width = 8, height = 7, dpi = 150)
```

---

## 3. Categorical / Count Plots

### Bar chart
```r
p <- df %>%
  count(category) %>%
  mutate(category = reorder(category, n)) %>%
  ggplot(aes(x = category, y = n, fill = category)) +
  geom_col(alpha = 0.85) +
  geom_text(aes(label = n), hjust = -0.2, size = 4) +
  coord_flip() +
  labs(title = "Counts by Category", x = "", y = "Count") +
  theme(legend.position = "none")
ggsave("/tmp/barplot.png", p, width = 8, height = 5, dpi = 150)
```

---

## 4. Time Series Plots

```r
library(lubridate)
p <- ggplot(df, aes(x = date, y = value)) +
  geom_line(color = "#4C9BE8", linewidth = 1) +
  geom_smooth(method = "loess", color = "#E84C4C", se = TRUE, alpha = 0.2) +
  labs(title = "Time Series", x = "Date", y = "Value") +
  scale_x_date(date_labels = "%Y-%m", date_breaks = "3 months") +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
ggsave("/tmp/timeseries.png", p, width = 10, height = 5, dpi = 150)
```

---

## 5. Regression Diagnostics

```r
# Built-in diagnostic plots (base R, fast)
png("/tmp/regression_diagnostics.png", width = 1200, height = 900, res = 120)
par(mfrow = c(2, 2))
plot(model)
dev.off()
```

---

## Copy all plots to output
```bash
cp /tmp/*.png /mnt/user-data/outputs/ 2>/dev/null && echo "Plots copied"
ls /mnt/user-data/outputs/*.png 2>/dev/null
```
