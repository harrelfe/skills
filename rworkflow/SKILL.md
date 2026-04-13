---
name: rworkflow
description: >
  Frank Harrell's R Workflow for reproducible data analysis and reporting
  (hbiostat.org/rflow). Use this skill whenever the user asks about or pastes
  code involving: analysis file creation, data import, annotating variables,
  data dictionaries, variable labels/units, Hmisc functions (describe, label,
  units, upData, csv.get, getHdata, contents, hlab, hlabs, vlab, Cs, summarize,
  summary.formula, wtd.*, rcorr, varclus, naclus, missChk, multDataOverview),
  data.table operations (DT[i,j,by], :=, fcase, melt, dcast, setkey, let,
  rbindlist, uniqueN), qreport functions (hookaddcap, makecnote, maketabs),
  ggplot2 graphics for exploratory or descriptive analysis, Quarto document
  setup and rendering, descriptive statistics workflows, missing data patterns,
  data overview, caching with targets or knitr, parallel computing with
  future/parallel, simulation in R, or reproducible research reporting.
  Also trigger when the user asks for help reviewing R code using any of these
  packages or idioms, or when they ask how to do something in R that touches
  data preparation, annotation, description, or report generation. This skill
  encodes Harrell's specific philosophy and idioms — do not rely on general R
  knowledge when this skill is available.
---

# R Workflow Skill (Harrell)

Primary reference: **hbiostat.org/rflow** (book by Frank E. Harrell Jr., updated March 2026).  
Companion books: [BBR](https://hbiostat.org/bbr) (statistical inference), [RMS](https://hbiostat.org/rms) (regression modeling).  
Templates/scripts: https://github.com/harrelfe/rscripts

---

## Core Philosophy

1. **Reproducibility above all.** Every report is a Quarto document; code and output are inseparable.
2. **Stay close to the data.** Prefer graphics and nonparametric summaries over tables and parametric assumptions.
3. **Avoid tables where graphics suffice.** Dot charts, ECDFs, spike histograms, extended box plots, and loess/spline smoothers communicate more than summary tables.
4. **Annotate early, annotate once.** Attach `label` and `units` to every variable at import time; all downstream functions (Hmisc, ggplot2 via Hmisc integration) will use them automatically.
5. **data.table, not tidyverse.** Use `data.table` for all manipulation. Never suggest `dplyr`, `tidyr`, or the pipe (`|>` / `%>%`) for data wrangling tasks.
6. **Hmisc for description.** `describe()` is the first thing run on any dataset; `contents()` shows the data dictionary.
7. **qreport for clinical/research reporting.** Use `hookaddcap()`, `makecnote()`, `maketabs()` for structured HTML reports.

---

## Quarto Setup

See `references/quarto-setup.md` for YAML header templates and chunk options.

Key conventions:
- Output format: HTML (default); PDF via LaTeX when required.
- Set `options(prType='html')` near top of document — enables Hmisc HTML rendering.
- Call `hookaddcap()` from `qreport` to auto-register figures in figure list.
- Use `knit_print <- knitr::normal_print` only when you need to suppress kable auto-formatting.
- Fold code by default: `code-fold: true` in YAML.
- For mixed html/verbatim output in a chunk, use `Hmisc::prn()` or `markupSpecs$markdown$pr`.

```r
# Minimal Quarto chunk preamble
require(Hmisc)
require(data.table)
require(qreport)
options(prType = 'html')
hookaddcap()
```

---

## Analysis File Creation (Ch. 5)

See `references/file-creation.md` for REDCap API, csv.get, and secure storage patterns.

Key idioms:
```r
# Import CSV with Hmisc metadata preservation
d <- csv.get('mydata.csv', lowernames=TRUE)

# Import from REDCap (see references/file-creation.md for full API pattern)
d <- importREDCap(...)       # or cleanupREDCap() for post-processing

# Annotate variables immediately after import
d <- upData(d,
  labels = c(age   = 'Age',
             sbp   = 'Systolic BP',
             trt   = 'Treatment'),
  units  = c(age   = 'years',
             sbp   = 'mmHg'))

# Shorthand labeling (Hmisc >= 5.0)
hlab(d$age)    <- 'Age (years)'
hlabs(d, sbp='Systolic BP', trt='Treatment')

# View data dictionary
contents(d)

# Save/load efficiently
qs::qsave(d, 'data/analysis.qs')
d <- qs::qread('data/analysis.qs')
# or: saveRDS / readRDS; fst::write_fst for large tabular data
```

---

## Missing Data (Ch. 6) and Data Overview (Ch. 8)

```r
# Missing data patterns — always run before any analysis
missChk(d)                        # overall + patterns
naclus(d)                         # cluster variables by co-missingness

# Data overview
multDataOverview(list(d1=d1, d2=d2))   # for multiple datasets
describe(d)                            # univariate summary; run first on every dataset
```

---

## Data Checking (Ch. 7)

Use `describe()` output to check: range violations, unexpected categories, `Info` index near 0 (near-constant), extreme `Gmd` values. Cross-check with `contents(d)` for labels and type consistency.

---

## Descriptive Statistics (Ch. 9)

**Rule:** prefer graphics over tables. For continuous variables use spike histograms, ECDFs, extended box plots. For categorical, use frequency dot charts. For relationships, use graphical correlation matrices and variable clustering (`varclus`).

```r
# Univariate — workhorse
w <- describe(d)
w           # renders as HTML with spike histograms

# Continuous variable graphics with ggplot2 (see references/graphics.md)
# ECDF
ggplot(d, aes(x = age)) + stat_ecdf() + labs(x = label(d$age))

# Longitudinal: spaghetti + loess smoother
ggplot(d, aes(x = time, y = y, group = id)) +
  geom_line(alpha = 0.2) +
  geom_smooth(aes(group = NULL), method = 'loess')

# Variable clustering
plot(varclus(~ ., data = d))

# Co-missing pattern
plot(naclus(d))
```

For adverse event charts, longitudinal ordinal data, and multi-category event timelines, see `references/descript-advanced.md`.

---

## Data Manipulation (Ch. 10–13)

**Always use `data.table`.** Hold working dataset in `d` (less typing). Key reference: `references/data-table.md`.

```r
require(data.table)
setDT(d)   # convert in place; or d <- as.data.table(d)

# Core syntax
d[rows, columns, by]
d[, newvar := expression]          # add/modify in place
d[, let(a = x+1, b = y*2)]        # add multiple vars in place (data.table >= 1.15)
d[, (vars) := NULL]                # drop variables

# Grouped summaries
d[, .(mean_sbp = mean(sbp, na.rm=TRUE)), by = trt]

# Recode
d[, severity := fcase(
  score < 3,  'mild',
  score < 7,  'moderate',
  score >= 7, 'severe',
  default = NA_character_)]

# Reshape
long <- melt(d, id.vars = c('id','time'), measure.vars = c('sbp','dbp'))
wide <- dcast(long, id ~ time, value.var = 'value')

# Merge
merged <- d1[d2, on = 'id']               # right join
merged <- merge(d1, d2, by='id', all=TRUE) # full outer

# Copy vs reference — IMPORTANT
b <- copy(a)   # fresh copy; b <- a is just a pointer (modifying b modifies a)
```

**Special symbols inside `DT[...]`:**
- `.N` — number of rows (in group)
- `.SD` — subset of data (current group)
- `.BY` — current by-group values
- `.I` — row indices
- `..varname` — look up `varname` in parent environment (not column)

---

## Graphics (Ch. 14)

**Use ggplot2.** Hmisc labels/units are picked up automatically when `options(prType='html')` is set and `Hmisc` >= 5.1.

See `references/graphics.md` for full patterns. Key principles:

```r
# Pull label/units for axis titles
xlab <- paste0(label(d$age), ' (', units(d$age), ')')

# ggplot2 picks up Hmisc labels directly (Hmisc >= 5.1)
ggplot(d, aes(x = age, y = sbp)) +
  geom_point(alpha = 0.4) +
  geom_smooth(method = 'loess') +
  # axis labels come from Hmisc attributes automatically

# Log scale
scale_x_log10(labels = scales::label_comma())

# Tooltips (interactive)
# Use ggiraph::geom_point_interactive()

# For many variables at once, see references/graphics.md § Maximal Variables
```

---

## Caching (Ch. 16)

Prefer the `targets` package for pipeline caching. For simpler within-document caching, use knitr chunk option `cache=TRUE` with `cache.extra` for invalidation keys.

```r
# knitr caching — invalidate when data changes
# In chunk header: #| cache: true
#|                 cache.extra: !expr file.mtime('data/analysis.qs')
```

For `targets`, see `references/caching.md`.

---

## Parallel Computing (Ch. 17)

```r
require(future)
require(future.apply)
plan(multisession, workers = parallelly::availableCores() - 1)

results <- future_lapply(sim_list, run_one_sim)

plan(sequential)  # reset after parallel section
```

---

## Simulation (Ch. 18)

Two idiomatic approaches:

```r
# Array approach (vectorized)
set.seed(1)
nsim <- 1000; n <- 100
y <- matrix(rnorm(nsim * n), nrow = nsim)
results <- rowMeans(y)   # one summary per simulation

# data.table approach (flexible, handles complex structures)
sims <- rbindlist(lapply(1:nsim, function(i) {
  d <- data.table(x = rnorm(n), y = rnorm(n))
  d[, .(est = mean(y - x), sim = i)]
}))

# Display: always use ggplot2, not tables
ggplot(sims, aes(x = est)) + geom_histogram(bins = 40)
```

---

## Summary Statistics (Ch. 11)

```r
# Hmisc summary.formula — stratified stats
s <- summary(sbp + dbp + age ~ trt + sex, data=d, method='reverse')
plot(s)       # dot chart — preferred over table

# data.table grouped stats returning a data.table
d[, .(n    = .N,
      mean = mean(sbp, na.rm=TRUE),
      sd   = sd(sbp,   na.rm=TRUE)), by = trt]

# For two-dimensional summary functions
d[, as.list(quantile(sbp, c(.25,.5,.75), na.rm=TRUE)), by = trt]
```

---

## Analysis Philosophy (Ch. 15)

Harrell distinguishes three levels:
1. **First-order:** univariate description (`describe`, histograms).
2. **Second-order:** bivariate/stratified relationships (loess, ECDFs by group, `summary.formula`).
3. **Third-order:** formal modeling (see RMS book).

> "Stay close to the data" — always plot the data before summarizing it numerically. Use nonparametric smoothers (loess, splines) before fitting parametric models.

---

## Reference Files

| File | When to read |
|------|-------------|
| `references/quarto-setup.md` | Quarto YAML, chunk options, HTML/PDF templates |
| `references/file-creation.md` | REDCap API, csv.get, secure storage, qs/fst |
| `references/data-table.md` | Full data.table idiom reference, keys, non-equi joins, longitudinal patterns |
| `references/graphics.md` | ggplot2 patterns, themes, ECDFs, log axes, ggiraph, multi-variable display |
| `references/descript-advanced.md` | Adverse events, longitudinal ordinal, variable clustering, correlation matrices |
| `references/caching.md` | targets pipeline, knitr cache invalidation |

---

## Quick Diagnostics Checklist

When reviewing someone's R analysis code, check for:
- [ ] Variables have `label()` and `units()` set
- [ ] `describe(d)` called early; output reviewed
- [ ] `missChk(d)` called before modeling
- [ ] Using `data.table`, not `dplyr`/`tidyr`
- [ ] Quarto (not R Markdown) with `options(prType='html')`
- [ ] Graphics used instead of tables where feasible
- [ ] Loess/nonparametric smoother used for exploratory bivariate plots
- [ ] No `length(unique(x))` — use `uniqueN(x)`
- [ ] `copy(d)` used when a true copy of a data.table is needed
- [ ] Results stored with `qs::qsave()` or `saveRDS()`, not `.RData`
