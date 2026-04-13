# data.table Reference

Source: hbiostat.org/rflow/manip, /merge, /long  
Cheatsheet: https://raw.githubusercontent.com/rstudio/cheatsheets/master/datatable.pdf  
Vignettes: https://cran.r-project.org/web/packages/data.table/vignettes

---

## Fundamental Syntax

```
DT[ i,  j,  by ]
     |   |   |
     |   |   +---> grouped by what? (or: on = for joins)
     |   +-------> what to do? (select, compute, assign with :=)
     +-----------> which rows?
```

`data.table` operates *by reference* — assignments with `:=` modify in place, no copy.  
`b <- a` is a pointer, NOT a copy. Use `b <- copy(a)` for an independent copy.

---

## Row Operations

```r
d[2]                          # row 2
d[2:4]                        # rows 2-4
d[age > 40]                   # filter
d[trt == 'A' & sex == 'F']    # compound filter
d[month %like% 'mb']          # regex-like match
d[month %ilike% 'mb']         # case-insensitive
d[.N]                         # last row
d[order(age, -sbp)]           # sort
```

## Column Operations

```r
d[, age]                      # returns vector
d[, .(age, sex)]              # returns data.table (two cols)
d[, c('age','sex')]           # same, programmatic names
v <- 'age'; d[, ..v]          # use variable holding column name
i <- 3;     d[, ..i]          # use variable holding column index
d[, (v)]                      # another way to get col named by v

d[, .N]                       # count rows
d[, uniqueN(id)]              # unique count — NOT length(unique(id))
```

## Add / Modify / Remove Variables

```r
d[, newvar := x + y]                          # add/overwrite in place
d[, let(a = x+1, b = y*2)]                   # multiple (DT >= 1.15); let is alias for :=
d[, c('a','b') := .(x+1, y*2)]               # multiple (older syntax)
d[condition, newvar := value]                  # conditional assignment

vars <- c('a','b','c')
d[, (vars) := NULL]           # drop named variables
d[, 2:4 := NULL]              # drop by position (rarely preferred)
setcolorder(d, c('id','time','y'))             # reorder columns
setnames(d, old = 'oldname', new = 'newname') # rename
```

## Special Symbols

| Symbol | Meaning |
|--------|---------|
| `.N` | Number of rows (in group if `by=` present) |
| `.SD` | Subset of Data (current group as data.table) |
| `.SDcols` | Which columns `.SD` should contain |
| `.BY` | List of current by-group values |
| `.I` | Integer vector of row indices |
| `.GRP` | Group counter |
| `..x` | Look up `x` in calling environment (not column) |

## Grouped Operations

```r
d[, .(mean_sbp = mean(sbp, na.rm=TRUE),
       n        = .N), by = trt]

d[, .N, keyby = .(trt, sex)]         # keyby sorts result

# Apply function across columns
d[, lapply(.SD, mean, na.rm=TRUE),
  .SDcols = c('sbp','dbp'), by = trt]

# Add group aggregate back to original rows
d[, mean_sbp_by_trt := mean(sbp, na.rm=TRUE), by = trt]
```

## Recoding

```r
# fcase — vectorized case_when replacement
d[, severity := fcase(
  score < 3,  'mild',
  score < 7,  'moderate',
  score >= 7, 'severe',
  default     = NA_character_)]

# Recode factor levels
require(Hmisc)
d[, race := combine.levels(race, c('Asian','Pacific Islander'), 'Asian/PI')]

# Binary scoring
d[, outcome := score.binary(x, c(0,1))]
```

## Reshaping

```r
# Wide to long
long <- melt(d,
  id.vars      = c('id','time'),
  measure.vars = c('sbp','dbp','hr'),
  variable.name = 'vital',
  value.name    = 'value')

# Long to wide
wide <- dcast(long, id ~ time, value.var = 'value')
wide <- dcast(long, id ~ vital + time, value.var = 'value', fun.aggregate = mean)

# Directly create melted aggregate (avoid double-reshape)
melted <- d[, .(val = c(mean(sbp), mean(dbp)),
                var = c('sbp','dbp')), by = trt]
```

## Keys and Joins

```r
setkey(d, id)               # set key (sorts + indexes)
setkey(d, id, time)         # composite key

# Equi joins
d1[d2, on = 'id']                          # right join (d2 drives rows)
merge(d1, d2, by = 'id', all = TRUE)       # full outer
merge(d1, d2, by = 'id', all.x = TRUE)     # left outer

# Non-equi join (closest match in time)
d1[d2, on = .(id, time >= t_start, time <= t_end)]

# Rolling join
d1[d2, on = .(id, time), roll = TRUE]      # carry last forward
d1[d2, on = .(id, time), roll = 'nearest'] # nearest
```

## Operating on a List of Data Tables (by reference)

```r
tables <- list(a = dt_a, b = dt_b, c = dt_c)
for (w in tables) w[, newvar := oldvar * 2]   # modifies each in place
```

## Fast Disk Storage

```r
require(fst)
write_fst(d, 'data/mydata.fst')
d2 <- read_fst('data/mydata.fst', as.data.table = TRUE)
# Partial column read:
d_sub <- read_fst('data/mydata.fst', columns = c('id','age'), as.data.table = TRUE)
```

---

## Longitudinal Data Patterns (Ch. 13)

```r
# Last observation carried forward
d[, y_locf := nafill(y, type = 'locf'), by = id]

# Compute gap times between intervals
d[, gap := t_start - shift(t_end, type='lag'), by = id]

# Linear interpolation to target times
target_times <- data.table(id = ..., time = ...)
d[target_times, on = .(id, time), roll = TRUE]   # carry forward
# For true linear interp, see hbiostat.org/rflow/long#sec-long-interp

# Summarize multiple baseline measurements per subject
d[time <= 0, .(baseline_mean = mean(y, na.rm=TRUE)), by = id]

# Multiple longitudinal continuous variables — variable clustering by time
# See references/descript-advanced.md
```

## Using AI as a data.table Assistant (Ch. 13.8)

> Paste the data structure (`str(d)`) and desired transformation in plain English.  
> LLMs produce reliable `data.table` code when given a clear target output.  
> Always verify output on a small test dataset.
