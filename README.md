# rstats-tips
Repository of what I end up googling for again and again...

## GGplot2
### Vectors
#### Get a vector out of a tibble column
`pull(data, col_name)`
### Labels
#### Remove labels
`+ guides(color = "none")`
### Scales
#### Log gradient scale
`+ scale_fill_gradient(trans = "log")`
### Aesthetics
Variables, within `aes(color = var_color)`, constant, outside: `geom_point(aes(x, y), color = "red")`

# Watch out!
## `%>%` and `.`
`tibble %>% full_join(. %>% filter(), . %>% filter())` doesn't work.
But `tibble %>% full_join(filter(.), filter(.))` does. Not clear why yet :(

# Package(s) for MS office interaction
[David Gohel's `officer`](https://davidgohel.github.io/officer/)


# TODO
## `purrrlyr::by_row()`
Check it out
