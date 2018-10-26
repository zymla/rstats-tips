# rstats-tips
Repository of what I end up googling for again and again...

## sparklyr
### remove NAs
`drop_na()` isn't available. Instead, use
```
flights %>% na.omit()
```


## readr
### Clean column names
`rename_all(funs(str_replace(., '^[^.]*\\.', '')))`

## data.table
### Read large CSV files faster than with readr
```
data.table::fread(
    file = 'data_raw/huge_csv_file.csv', 
    na.strings = c('', 'null', 'Null', 'NULL', '1980-01-01', '1980-01-01 00:00:00.0')
    ) %>% 
  as.tibble() %>% 
  rename_all(funs(str_replace(., '^[^.]*\\.', ''))) %>% 
  mutate(date_col = ymd(date_col))
```
### general format
```
dt[rows, cols, by]
dt[,.(col1, col2)]
dt[col1 > 0, .(col1, prod = col1 * col2)]
```

### spread / pivot & index within group
```
dcast(flights[, as.list(lm(air_time ~ distance)[c('coefficients', 'df.residual')]), by = .(month)][,.(coefficients, df.residual, .I-.I[1]), by = .(month)], month + df.residual ~ V3, value.var="coefficients")
```

### help
```
?`[.data.table`
```

## GGplot2
### Vectors
#### Get a vector out of a tibble column
`pull(data, col_name)`
### Labels
#### Remove labels
`+ guides(color = "none")`
#### Rotate labeles
`+ theme(axis.text.x = element_text(angle = 90)`
### Scales
#### Log gradient scale
`+ scale_fill_gradient(trans = "log")`
### Aesthetics
Variables, within `aes(color = var_color)`, constant, outside: `geom_point(aes(x, y), color = "red")`

# Office
## Copy table to excel via clipboard
`%>% write.table("clipboard", sep="\t")`

# Watch out!
## `%>%` and `.`
`tibble %>% full_join(. %>% filter(), . %>% filter())` doesn't work.
But `tibble %>% full_join(filter(.), filter(.))` does. Not clear why yet :(

# Package(s) for MS office interaction
[David Gohel's `officer`](https://davidgohel.github.io/officer/)


# TODO
## `purrrlyr::by_row()`
Check it out
