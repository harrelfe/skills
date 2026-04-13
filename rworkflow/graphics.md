# Graphics Reference (ggplot2 + Hmisc)

Source: hbiostat.org/rflow/graphics  
Ch. 14 covers: graphic devices, ggplot2 themes, ECDFs, log axes, ggiraph, multi-variable display.

---

## Philosophy

- Prefer graphics over tables for descriptive analysis.
- Use nonparametric smoothers (loess, regression splines) before fitting models.
- Recommended graph types by data type (from Ch. 14):

| Data type | Recommended graphic |
|-----------|-------------------|
| Continuous univariate | Spike histogram, ECDF, extended box plot |
| Categorical univariate | Frequency dot chart |
| Continuous bivariate | Scatter + loess smoother |
| Continuous × categorical | Extended box plots or ECDFs by group |
| Longitudinal continuous | Spaghetti + loess smoother |
| Longitudinal ordinal | State/transition diagrams |
| Correlations | Graphical correlation matrix (`rcorr` + heatmap), `varclus` dendrogram |
| Events | Multi-category dot charts or timelines |

---

## Hmisc Label Integration

With Hmisc >= 5.1, ggplot2 automatically uses `label()` and `units()` for axis titles when the data are properly annotated. No manual `labs()` needed if labels are set.

```r
# Fallback: extract manually
xlab <- paste0(label(d$age), if(nchar(units(d$age))) paste0(' (', units(d$age), ')'))

# Or use Hmisc helper
ggplot(d, aes(x = age, y = sbp)) + ...
# axis titles come from attributes automatically (Hmisc >= 5.1 + ggplot2 recent)
```

---

## Common Patterns

### Spike Histogram (base Hmisc)

```r
# Hmisc spike histogram — use inside describe() output
# For ggplot2 version:
ggplot(d, aes(x = sbp)) +
  geom_histogram(bins = 50, fill = 'steelblue') +
  labs(x = label(d$sbp))
```

### ECDF

```r
ggplot(d, aes(x = age)) +
  stat_ecdf() +
  labs(x = label(d$age), y = 'Cumulative Proportion')

# By group
ggplot(d, aes(x = age, color = trt)) +
  stat_ecdf() +
  labs(x = label(d$age), color = label(d$trt))

# Transformed axes (e.g., log)
ggplot(d, aes(x = creatinine)) +
  stat_ecdf() +
  scale_x_log10(labels = scales::label_comma()) +
  labs(x = 'Creatinine (log scale)')
```

### Scatter + Smoother

```r
ggplot(d, aes(x = age, y = sbp)) +
  geom_point(alpha = 0.3, size = 1) +
  geom_smooth(method = 'loess', se = TRUE) +
  labs(x = label(d$age), y = label(d$sbp))
```

### Extended Box Plot / Violin

```r
ggplot(d, aes(x = trt, y = sbp)) +
  geom_violin(fill = 'lightblue', alpha = 0.5) +
  geom_boxplot(width = 0.1, outlier.shape = NA) +
  labs(x = label(d$trt), y = label(d$sbp))
```

### Longitudinal Spaghetti + Smoother

```r
ggplot(d, aes(x = time, y = y, group = id)) +
  geom_line(alpha = 0.15, color = 'grey50') +
  geom_smooth(aes(group = NULL), method = 'loess', color = 'steelblue') +
  labs(x = 'Time', y = label(d$y))
```

### Plotting Inside a for-loop

```r
for (v in c('sbp','dbp','hr')) {
  p <- ggplot(d, aes(x = .data[[v]])) + stat_ecdf() + labs(x = v)
  print(p)
}
```

---

## Log Scale

```r
# Nice log10 axis
scale_x_log10(
  breaks = scales::trans_breaks('log10', function(x) 10^x),
  labels = scales::trans_format('log10', scales::math_format(10^.x))
)
```

---

## Maximal Variables with ggplot2 (Ch. 14 § Maximal Variables)

Use `facet_wrap` or `facet_grid` to display many variables simultaneously:

```r
# Melt first, then facet
require(data.table)
dm <- melt(d, id.vars = c('id','trt'),
           measure.vars = c('sbp','dbp','hr','age'))

ggplot(dm, aes(x = value, color = trt)) +
  stat_ecdf() +
  facet_wrap(~ variable, scales = 'free_x') +
  labs(color = label(d$trt))
```

---

## Interactive Tooltips (ggiraph)

```r
require(ggiraph)
p <- ggplot(d, aes(x = age, y = sbp,
                   tooltip = paste0('ID: ', id, '\nAge: ', age))) +
  geom_point_interactive(alpha = 0.4)
girafe(ggobj = p)
```

---

## Themes and Fonts

```r
theme_set(theme_bw(base_size = 13))  # set globally at top of document
# or
theme_set(theme_minimal())
```

---

## Graphical Correlation Matrix

```r
require(Hmisc)
r <- rcorr(as.matrix(d[, .(sbp, dbp, age, bmi)]))
# Plot with corrplot or ggplot2 heatmap
require(corrplot)
corrplot(r$r, p.mat = r$P, sig.level = 0.05)
```

## Variable Clustering

```r
plot(varclus(~ sbp + dbp + age + bmi + creatinine, data = d))
```
