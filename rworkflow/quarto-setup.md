# Quarto Setup Reference

Source: hbiostat.org/rflow/rformat  
Template: https://hbiostat.org/R/reportTemplate.qmd

---

## Recommended YAML Header (HTML report)

```yaml
---
title: "Analysis Title"
author: "Your Name"
date: last-modified
format:
  html:
    toc: true
    toc-depth: 3
    number-sections: true
    code-fold: true
    code-tools: true
    df-print: kable
    self-contained: true
    theme: flatly
execute:
  warning: false
  message: false
  cache: false
---
```

## PDF Output Addition

```yaml
format:
  html: ...
  pdf:
    toc: true
    number-sections: true
```

## Multi-format (HTML + Word collaboration)

```yaml
format:
  html: ...
  docx:
    toc: true
```
See Ch. 4.5 for considerations when sharing with Word users.

---

## Standard Setup Chunk

```r
#| label: setup
#| include: false
require(Hmisc)
require(data.table)
require(ggplot2)
require(qreport)
options(prType = 'html')
hookaddcap()   # auto-register figures for figure list (qreport)
# knit_print <- knitr::normal_print  # uncomment to suppress kable auto-format
```

---

## Chunk Options Cheatsheet

```
#| label: chunk-name        # required for cross-refs
#| fig-cap: "Caption text"
#| fig-width: 7
#| fig-height: 5
#| cache: true
#| cache.extra: !expr file.mtime('data/analysis.qs')   # invalidation key
#| results: asis            # needed for raw HTML/LaTeX output (older Hmisc)
#| echo: false
#| warning: false
```

With Hmisc >= 4.8 and rms >= 6.5-0, `results: asis` is no longer required for most Hmisc HTML output.

---

## Printing Mixed Output

When a chunk mixes HTML objects (from Hmisc/rms) and plain text:

```r
# Define pr helper at top of document
pr <- markupSpecs$markdown$pr

# Then in chunks with results='asis':
pr(obj = some_matrix)       # prints verbatim inside asis chunk
pr(inline = paste(x, collapse=', '))
prn(myobject)               # prints name + value (Hmisc)
```

---

## Callout Notes Around Output

```r
# Wrap describe() output in a Quarto callout note
makecnote(~ describe(d), wide = TRUE)
```

---

## Tabsets

```r
# qreport::maketabs() for tabbed output
maketabs(
  Tab1 = ~ describe(d[, .(age, sbp)]),
  Tab2 = ~ describe(d[, .(sex, trt)])
)
```

---

## Diagrams (Graphviz / Mermaid)

```r
# Flowcharts via Hmisc::makegvflow() or makedot()
# Embed in Quarto chunk with engine: dot or via R string

# Mermaid in Quarto:
```
```{mermaid}
flowchart LR
  A --> B --> C
```
```

---

## CSS Styling

Place in YAML or a `<style>` block:
```css
.bold-red { color: red; font-weight: bold; }
```
Apply inline: `[important text]{.bold-red}`

See Ch. 4.7 for colorizing output and custom callouts.
