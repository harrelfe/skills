# Caching Reference

Source: hbiostat.org/rflow/caching (Ch. 16)

---

## knitr Chunk-Level Caching

Simple, within-document caching. Sufficient for most reports.

```r
# In chunk header:
#| cache: true
#| cache.extra: !expr file.mtime('data/analysis.qs')

# The cache.extra expression is re-evaluated each render;
# if file modification time changes, cache is invalidated.

# Multiple dependencies:
#| cache.extra: !expr list(file.mtime('data/a.qs'), file.mtime('data/b.qs'))
```

Caveats:
- `cache=TRUE` caches the *chunk result*, not individual object states.
- Side effects (plots, printed output) may not be cached correctly.
- Use `dependson` to invalidate when a prior chunk's objects change.

---

## targets Pipeline Caching

For multi-script projects or long-running computations. Each target is cached independently; only invalidated targets re-run.

```r
# _targets.R (project root)
library(targets)
library(tarchetypes)

tar_option_set(packages = c('Hmisc','data.table','ggplot2'))

list(
  tar_target(raw_file, 'data/raw.csv', format = 'file'),
  tar_target(d_raw,    csv.get(raw_file, lowernames=TRUE)),
  tar_target(d,        annotate_data(d_raw)),    # custom function
  tar_target(desc,     describe(d)),
  tar_target(report,   tar_render(tarchetypes::tar_quarto('report.qmd')))
)
```

```r
# Run pipeline
targets::tar_make()

# Inspect
targets::tar_read(d)          # load cached target
targets::tar_visnetwork()     # visualize dependency graph
targets::tar_outdated()       # which targets need re-running
```

---

## Parallel targets

```r
tar_option_set(controller = crew::crew_controller_local(workers = 4))
targets::tar_make()
```

---

## When to Use What

| Scenario | Recommendation |
|---------|----------------|
| Single Quarto document, moderate runtime | knitr `cache: true` with `cache.extra` |
| Multi-step pipeline, multiple outputs | `targets` |
| Long simulation (minutes–hours) | `targets` or `future`/`qs` manual caching |
| Interactive exploration | No caching; re-run interactively |
