# rstats-tips
Repository of what I end up googling for again and again...
# Ouput
```
options(max.print=100)
options(tibble.print_max=100)
```
# JupyterLab
## Install
### Install `R` on Debian
```
sudo apt install r-base r-base-dev
```

### On GCP Vertex AI needed the following
#### `tidyverse`
```
sudo apt install libssl-dev
sudo apt install libfontconfig1-dev
sudo apt install libharfbuzz-dev libfribidi-dev
```

## Misc
To resize `ggplot` outputs in the notebook:
```
options(repr.plot.width=11, repr.plot.height=7)
```
```
%%R -w 15 -h 15 --units in
```
# Tensorflow/Keras
## To install with conda
```
install.packages("tensorflow")
install_tensorflow()
```
## If issue with numpy version arises:
Then I had issues with numy version, so:
```
conda remove -n r-tensorflow numpy
```
Then instal numpy 1.14.5
```
conda install -n r-tensorflow numpy=1.14.5
```

## Always set which python to use just after restarting R session. On a Mac:
```
library(reticulate)
reticulate::use_python('/Users/XXXXX/anaconda3/envs/r-tensorflow/bin/python', required = TRUE)
reticulate::py_config()
```
## Hello world
```
library(tensorflow)
#use_condaenv('r-tensorflow')
h = tf$constant("Hello")
w = tf$constant(" World!")
hw = h + w

init <- tf$global_variables_initializer()
session <- tf$Session()
session$run(init)

ans <- session$run(hw)
ans
```

# Summaries
```
dplyr::glimpse(df)
dplyr::tbl_sum(df)
skimr::skim(df)
Hmisc::describe(df)
summarytools::descr(spam)
summarytools::dfSummary(df)
```

# Misc
By default, `1` is a float. To create an int, the syntax is
```
1L
```

# Tidyverse
## readr
### Clean column names
`rename_all(funs(str_replace(., '^[^.]*\\.', '')))`

## `dplyr`
### Remove duplicates
`distinct()`

## Vectors
### Get a vector out of a tibble column
`pull(data, col_name)`

## Timeseries `tsibble`
```
library(tsibble)
df %>%
  as_tsibble(index='dt_utc') %>%
  fill_gaps(km = 0, nb_cars = 0) %>% 
  mutate(dt_tzfr=if_else(is.na(dt_tzfr), with_tz(dt_utc, 'Europe/Paris'), dt_tzfr)) %>%
  as_tibble()
```

## GGplot2
A good ref [[http://zevross.com/blog/2014/08/04/beautiful-plotting-in-r-a-ggplot2-cheatsheet-3/]]
### Labels
#### Remove labels
`+ guides(color = "none")`
#### Rotate labels
`+ theme(axis.text.x = element_text(angle = 90)`

or the more modern:

`scale_x_datetime(breaks = 'week', date_labels = "%b %d", guide = guide_axis(angle = 90))`
### Legend
#### Remove legend
`+ theme(legend.position = "none")`
### Theme
`+ theme(plot.caption=element_text(size=8, family='sans', face='italic', color='gray')`
### Expand limits (e.g. to origin)
`+ expand_limits(x = 0)`
### `xlim` but with clipping
`+coord_cartesian(xlim=c(0, 10))`
### Scales
#### Log gradient scale
`+ scale_fill_gradient(trans = "log")`
`+ scale_x_date(date_breaks = "months" , date_labels = "%b-%y")`
#### Log Breaks
`+ scale_y_log10(minor_breaks=seq(5e-2, 3e7, 1e-2) %>% round(., -floor(log10(.))) %>% unique())`
### Aesthetics
Variables, within `aes(color = var_color)`, constant, outside: `geom_point(aes(x, y), color = "red")`
### geofacet
#### Install issues
```
sudo apt-get install libudunits2-dev
sudo apt-get install gdal-bin
sudo apt-get install libgdal-dev
```

# `data.table`
## Chain/Pipe
### Bracket style
```
dt[a > 1, .(b = a + 1)][b < 3]
```
### `magrittr` style
```
dt[a > 1, .(b = a + 1)] %>% .[b < 3]
```

## help
```
?`[.data.table`
```
For help on `.N`, `.SD`, `.SDcols`
```
?`special-symbols`
```

## List all `data.table`s
```
tables()
```

## Creator
```
dt <- 
  data.table(
    x = runif(100),
    y = rnorm(100),
    g = sample.int(100, n = 2, replace = TRUE)
  )
```
## Read large CSV files faster than with readr
```
data.table::fread(
    file = 'data_raw/huge_csv_file.csv', 
    na.strings = c('', 'null', 'Null', 'NULL', '1980-01-01', '1980-01-01 00:00:00.0')
    ) %>% 
  as.tibble() %>% 
  rename_all(funs(str_replace(., '^[^.]*\\.', ''))) %>% 
  mutate(date_col = ymd(date_col))
```
## general format
```
dt[rows, cols, by]
dt[,.(col1, col2)]
dt[col1 > 0, .(col1, prod = col1 * col2)]
```

## Replace all `NA`s
```
dt[is.na(dt)] <- 0
```

## Operations on rows
### Order/arrange
To reorder inplace
```
setorder(dt, col_name)
```
To reorder without modifying the original data.table
```
dt[order(col_name)] 
```

### Intermediary computations
Works with `=`, but not with `:=`
```
dt[,
  {
    a <- x ^ 2
    b <- y ^ 2
    z <- a + b
    t <- a - b
    .(z = z, t = t)
  }
  ]
```
Can call function with side effects as well, e.g. `hist()`
```
res[, {hist(Result)}]
```
If invalid output/last value, then set the last value to `NULL`
```
res[,
  {
    hist(Result)
    NULL
  }, 
  Round
]
```

## Operations on columns
### Select (get as data.table)
```
dt[, .(y)]
dt[, "y"]
dt[, c("y")]
dt[, 2]
```
```
dt[, .(x, y)]
dt[, c("x", y")]
dt[, 1:2]
```

### Pull (get one column as vector)
```
dt[, y]
```

### Remove column
```
dt[, y := NULL]
```

### Work on multiple columns `.SD`
`.SD` is by default a list with all columns except grouping columns
```
dt[, lapply(data.table(.SD), mean), g]
```
One can also customize `.SD` by setting `.SDcols`
```
dt[, lapply(data.table(.SD), mean), g, .SDcols = -c("y")]
```

## Shift/lag
```
dt[, .(y = shift(x, type = "lag", fill = NA, n = 2L)), by = g]
```

## Aggregation
### Functions
#### tally `.N`
```
dt[, .(.N), g]
```

## spread / pivot & index within group
```
dcast(
  flights[, as.list(lm(air_time ~ distance)[c('coefficients', 'df.residual')]), by = .(month)][,.(coefficients, df.residual, .I-.I[1]), by = .(month)], 
  month + df.residual ~ V3, 
  value.var="coefficients")
```

## Joins
```
DT[X, on="x"]                         # right join
X[DT, on="x"]                         # left join
DT[X, on="x", nomatch=0]              # inner join
DT[!X, on="x"]                        # not join
DT[X, on=c(y="v")]                    # join using column "y" of DT with column "v" of X
DT[X, on="y==v"]                      # same as above (v1.9.8+)

DT[X, on=.(y<=foo)]                   # NEW non-equi join (v1.9.8+)
DT[X, on="y<=foo"]                    # same as above
DT[X, on=c("y<=foo")]                 # same as above
DT[X, on=.(y>=foo)]                   # NEW non-equi join (v1.9.8+)
DT[X, on=.(x, y<=foo)]                # NEW non-equi join (v1.9.8+)
DT[X, .(x,y,x.y,v), on=.(x, y>=foo)]  # Select x's join columns as well

DT[X, on="x", mult="first"]           # first row of each group
DT[X, on="x", mult="last"]            # last row of each group
DT[X, sum(v), by=.EACHI, on="x"]      # join and eval j for each row in i
DT[X, sum(v)*foo, by=.EACHI, on="x"]  # join inherited scope
DT[X, sum(v)*i.v, by=.EACHI, on="x"]  # 'i,v' refers to X's v column
DT[X, on=.(x, v>=v), sum(y)*foo, by=.EACHI] # NEW non-equi join with by=.EACHI (v1.9.8+)
```

## Options
### Limit number of printed rows
```
options(datatable.print.nrows = 20)
```



# Interaction with othe environents
## `reticulate` (Python)
### Create Python dictionaries/tuples
```
key <- 'some_key'
d <- reticulate::dict()
d[key] <- reticulate::dict(key1 = 'value1', key2 = reticulate::tuple('foo', 2 ))
d
```
outputs: `{'some_key': {'key1': 'value1', 'key2': ('foo', 2.0)}}`

## Docker
### Launch a docker container from R
```
client <- docker$from_env()
data_folder_path = normalizePath('local_path', winslash = '/')
vols <- reticulate::dict()
vols[data_folder_path] <- reticulate::dict(bind = '/path_in_container/', mode = 'ro')
ports <- reticulate::dict('5432/tcp' = reticulate::tuple('127.0.0.1', as.integer(5432)))
pg <- client$containers$run(name = 'pg', image = "image:latest", remove = TRUE, environment = reiculate::dict('foo' = 'bar'), ports = ports, volumes = vols, detach = TRUE)
```

## Databases / Big Data
### Passwords
```
rstudioapi::askForPassword("")
```
```
aesc <- digest::AES(charToRaw(stringr::str_sub(digest::sha1(uuid::UUIDgenerate(use.time = TRUE)), 1, 32)), mode = 'ECB')
aesc$decrypt(aesc$encrypt((function(x, n) (x[1:n]=x[1:n]))(charToRaw(rstudioapi::askForPassword()), 256)))
```

### RJDBC
#### Create driver for Hive
```
jdbc_jar_path <- "./hive/jdbc/hive-jdbc-xxxx-standalone.jar" 
drv <- JDBC("org.apache.hive.jdbc.HiveDriver", classPath=jdbc_jar_path)
```
#### Create connection
```
conn <- dbConnect(drv, glue('jdbc:hive2://xxx.xxx.xx:pppp/{default_hive_db};ssl=true;sslTrustStore={trust_store_path};trustStorePassword={trust_store_pwd};transportMode=http;httpPath=gateway/default/llap'), user, aesc$decrypt(skrt))
```

#### Query directly from R notebook
```
{sql, connection = conn, output.var="query_result"}
```
#### Close connection
```
dbDisconnect(conn)
```
### sparklyr
#### Start
```
spark_install(version = "2.1.0")
sc <- spark_connect(master = "local", version = "2.1.0")
flights_df <- copy_to(sc, nycflights13::flights, "flights_sp", overwrite = TRUE)
airlines <- copy_to(sc, airlines, "airlines")
src_tbls(sc)
```
#### remove NAs
`drop_na()` isn't available. Instead, use
```
flights %>% na.omit()
```


# ML
## Data prep
### Center/normalize matrix
```
scale(x, center = TRUE, scale = TRUE)
```
### Transpose matrix
```
t(x)
```
### matrix multiplication
```
x %*% y
```

### Eigen vectors/values
```
eigen(x)
```
### PCA
```
scale(decm)[,] %*% (eigen(t(scale(decm)) %*% scale(decm)))$vectors
```

## Clustering
### Viz
```
factoextra::fviz_cluster(kmeans(Xs, 5, algorithm = 'MacQueen'), data = Xs, labelsize = 0)
```

# Shiny
## JS logs to R console
```
shinyjs::showLog()
```
## Assign to a global variable that will persist in RStudio after the Shiny app is closed (facilitates debugging)
```
global_var <<- 1
```

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

# Related files
https://holtzy.github.io/Pimp-my-rmd/
