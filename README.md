# rstats-tips
Repository of what I end up googling for again and again...

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
