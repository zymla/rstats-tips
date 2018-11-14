# rstats-tips
Repository of what I end up googling for again and again...

## Passwords
```
rstudioapi::askForPassword("")
```

```
aesc <- digest::AES(charToRaw(stringr::str_sub(digest::sha1(uuid::UUIDgenerate(use.time = TRUE)), 1, 32)), mode = 'ECB')
aesc$decrypt(aesc$encrypt((function(x, n) (x[1:n]=x[1:n]))(charToRaw(rstudioapi::askForPassword()), 256)))
```

## RJDBC
### Create driver for Hive
```
jdbc_jar_path <- "./hive/jdbc/hive-jdbc-xxxx-standalone.jar" 
drv <- JDBC("org.apache.hive.jdbc.HiveDriver", classPath=jdbc_jar_path)
```
### Create connection
```
conn <- dbConnect(drv, glue('jdbc:hive2://xxx.xxx.xx:pppp/{default_hive_db};ssl=true;sslTrustStore={trust_store_path};trustStorePassword={trust_store_pwd};transportMode=http;httpPath=gateway/default/llap'), user, aesc$decrypt(skrt))
```
### Query directly from R notebook
`{sql, connection = conn, output.var="query_result"}`
### Close connection
```
dbDisconnect(conn)
```
## sparklyr
### Start
```
spark_install(version = "2.1.0")
sc <- spark_connect(master = "local", version = "2.1.0")
flights_df <- copy_to(sc, nycflights13::flights, "flights_sp", overwrite = TRUE)
airlines <- copy_to(sc, airlines, "airlines")
src_tbls(sc)
```
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
## `reticulate` (Python)
### Create Python dictionaries/tuples
```
key <- 'some_key'
d <- reticulate::dict()
d[key] <- reticulate::dict(key1 = 'value1', key2 = reticulate::tuple('foo', 2 ))
d
```
outputs: `{'some_key': {'key1': 'value1', 'key2': ('foo', 2.0)}}`

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
