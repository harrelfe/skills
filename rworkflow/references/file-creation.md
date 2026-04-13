# Analysis File Creation Reference

Source: hbiostat.org/rflow/fcreate  
Ch. 5 covers: data import (CSV, Excel, REDCap), annotation, secure storage, qs/fst.

---

## Import Methods

### CSV with Hmisc

```r
# csv.get preserves variable attributes and handles dates better than read.csv
d <- Hmisc::csv.get('data/myfile.csv', lowernames = TRUE, datevars = c('dob','enroll_dt'))
```

### Excel

```r
require(rio)
d <- as.data.table(rio::import('data/myfile.xlsx', sheet = 1))
# or
require(readxl)
d <- as.data.table(readxl::read_xlsx('data/myfile.xlsx'))
```

### General (rio)

```r
require(rio)
d <- as.data.table(rio::import('data/myfile.sas7bdat'))  # SAS, Stata, SPSS, etc.
```

---

## REDCap API

```r
# Modern API (2024+)
require(REDCapR)
token <- keyring::key_get('redcap', 'myproject')   # secure credential retrieval

raw <- redcap_read(
  redcap_uri = 'https://redcap.myinstitution.edu/api/',
  token      = token)$data

d <- as.data.table(raw)
d <- Hmisc::cleanupREDCap(d)    # clean factor levels, fix variable names
# or
d <- Hmisc::importREDCap(...)   # full import with metadata/labels
```

For depositing files back to REDCap, see hbiostat.org/rflow/fcreate#sec-fcreate-rcdep.

---

## Annotating Variables (do this immediately after import)

```r
require(Hmisc)

# Full annotation with upData
d <- upData(d,
  labels = c(
    age    = 'Age at enrollment',
    sbp    = 'Systolic blood pressure',
    trt    = 'Treatment assignment',
    sex    = 'Biological sex'),
  units = c(
    age    = 'years',
    sbp    = 'mmHg'),
  levels = list(
    trt = c('placebo', 'low dose', 'high dose'),
    sex = c('Female', 'Male')))

# Shorthand (Hmisc >= 5.0)
hlab(d$age)  <- 'Age (years)'              # label + units combined
hlabs(d,
  sbp = 'Systolic BP',
  dbp = 'Diastolic BP')                    # multiple at once
vlab(d$trt)  <- 'Treatment'               # label only, no units

# Check annotations
contents(d)                                # data dictionary view
label(d$sbp)                               # check one label
units(d$sbp)                               # check one unit
```

---

## Data Dictionary Workflow

```r
# View full dictionary
contents(d)

# Render in Quarto (HTML)
options(prType = 'html')
contents(d)   # renders as HTML table in report

# Export to CSV for sharing
dict <- as.data.frame(contents(d)$contents)
write.csv(dict, 'data/data_dictionary.csv', row.names = FALSE)
```

---

## Save and Load

### qs (recommended — fast, compressed)

```r
require(qs)
qs::qsave(d, 'data/analysis.qs')
d <- qs::qread('data/analysis.qs')
```

### RDS (portable, no extra package)

```r
saveRDS(d, 'data/analysis.rds')
d <- readRDS('data/analysis.rds')
```

### fst (fastest for large tabular data)

```r
require(fst)
fst::write_fst(d, 'data/analysis.fst', compress = 50)
d <- fst::read_fst('data/analysis.fst', as.data.table = TRUE)

# Read only selected columns (fast partial read)
d_sub <- fst::read_fst('data/analysis.fst',
  columns = c('id','time','sbp'), as.data.table = TRUE)
```

> Prefer `.qs` for general use. Use `.fst` when you need partial column reads from very large files.  
> Do NOT use `.RData` / `save()` — it silently overwrites objects in the global environment on load.

---

## Multiple Datasets

```r
# Summarize a list of datasets at once
Hmisc::multDataOverview(list(baseline = d_base, followup = d_fu, labs = d_labs))
```

---

## Secure File Storage

```r
# Store credentials with keyring (never hardcode passwords)
keyring::key_set('redcap', 'myproject')    # prompts once, stores securely
token <- keyring::key_get('redcap', 'myproject')

# Encrypt sensitive output files
# See hbiostat.org/rflow/fcreate#sec-fcreate-secure for gpg-based approach
```
