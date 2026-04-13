# Advanced Descriptive Statistics Reference

Source: hbiostat.org/rflow/descript (Ch. 9), hbiostat.org/rflow/sstats (Ch. 11)

---

## Hmisc describe() — Primary Univariate Tool

```r
require(Hmisc)
options(prType = 'html')

w <- describe(d)
w                        # renders HTML with spike histograms in Quarto

# Subset
describe(d[trt == 'A'])
describe(d[, .(sbp, dbp, age, sex)])   # selected variables
```

Output includes: n, missing, distinct values, Info index (information relative to continuous), Gmd (Gini's mean difference), quantiles, and spike histogram.

- **Info:** near 0 = near-constant or highly imbalanced binary
- **Gmd:** mean absolute pairwise difference — more robust dispersion measure than SD

---

## Graphical Summary of Stratified Stats (summary.formula)

```r
# Reverse table style — show distributions by treatment group
s <- summary(sbp + dbp + hr + age ~ trt,
             data   = d,
             method = 'reverse',
             overall= TRUE)
plot(s)         # dot chart — shows medians, IQRs, and p-values

# html rendering
print(s, html = TRUE)
```

---

## Adverse Event Charts (Ch. 9.5)

Two-panel ggplot2 chart: left panel = proportion with event; right panel = event rate.

```r
# See hbiostat.org/rflow/descript#sec-descript-ae for full code
# Key structure:
ae_summary <- d[, .(
  prop  = mean(event == 1),
  rate  = sum(event) / sum(time_at_risk)), by = .(ae_term, trt)]

# Plot with facet_grid(ae_term ~ ., ...) + geom_point() + coord_flip()
```

---

## Longitudinal Continuous Y (Ch. 9.2)

```r
# Spaghetti + loess (see references/graphics.md)
# Representative curves: show median + IQR band
d_sum <- d[, .(
  med = median(y, na.rm=TRUE),
  q25 = quantile(y, 0.25, na.rm=TRUE),
  q75 = quantile(y, 0.75, na.rm=TRUE)), by = time]

ggplot(d, aes(x = time, y = y, group = id)) +
  geom_line(alpha = 0.1) +
  geom_line(data = d_sum, aes(y = med, group = NULL), color = 'blue', linewidth = 1) +
  geom_ribbon(data = d_sum, aes(y = med, ymin = q25, ymax = q75, group = NULL),
              alpha = 0.2, fill = 'blue')
```

---

## Multiple Longitudinal Variables (Ch. 9.4)

### Cross-Correlation

```r
require(Hmisc)
# At each time point, compute pairwise correlations among variables
# Then plot correlation vs time
```

### Variable Clustering by Time

```r
# For each time slice, cluster variables; track cluster membership over time
# See hbiostat.org/rflow/descript#sec-descript-mlong for full example
plot(varclus(~ ., data = d[time == 0, .(sbp, dbp, hr, bmi)]))
```

---

## Variable Clustering (Relationships, Ch. 9.7)

```r
# Hierarchical clustering of variables by Spearman^2 correlation
vc <- varclus(~ sbp + dbp + age + bmi + creatinine + egfr, data = d)
plot(vc)

# Redundancy analysis (identify nearly redundant variables)
redun(~ sbp + dbp + age + bmi + creatinine, data = d, r2 = 0.9)
```

---

## Computing Summary Statistics (Ch. 11)

### data.table — single value per group

```r
d[, .(
  n    = .N,
  mean = mean(sbp, na.rm = TRUE),
  med  = median(sbp, na.rm = TRUE),
  sd   = sd(sbp, na.rm = TRUE),
  q25  = quantile(sbp, 0.25, na.rm = TRUE),
  q75  = quantile(sbp, 0.75, na.rm = TRUE)
), by = trt]
```

### data.table — functions returning multiple rows per group

```r
# quantile() returns a named vector — unlist into columns
d[, as.list(quantile(sbp, c(0.1, 0.5, 0.9), na.rm=TRUE)), by = trt]
```

### Customizing with gt

```r
require(gt)
stats_dt <- d[, .(mean = mean(sbp, na.rm=TRUE),
                  sd   = sd(sbp,   na.rm=TRUE)), by = trt]
gt(stats_dt) |>
  fmt_number(columns = c(mean, sd), decimals = 1) |>
  cols_label(trt = 'Treatment', mean = 'Mean SBP', sd = 'SD')
```

---

## Missing Data Patterns (Ch. 6)

```r
require(Hmisc)

# Full missing data check
missChk(d)          # overall + pattern summary

# Cluster variables by co-missingness
plot(naclus(d))

# Identify rows with many missings
d[rowSums(is.na(d)) > ncol(d) * 0.5]   # rows >50% missing
```
