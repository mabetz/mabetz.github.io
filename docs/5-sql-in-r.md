

# 5. SQL in R

<!--
By the end of this section you will be able to:

- connect to a SQLite database from R and see which tables and columns it has
- run a SQL query from R and understand that the result is a regular R data frame
- let SQL do the heavy work (filter, join, aggregate) and use R to plot and reshape the small result
- measure how long a query takes and avoid re-running slow queries
-->
  
## Concepts

So far we wrote SQL in its own cells. In real projects, the SQL query is usually only the first step: we pull a summary out of the database and
then keep working with it in R, for example to make a plot. The `imdbSept2026.db` database is about 22 GB, too big to load into R memory, so the idea is to let the database do the heavy work and bring back only a small result.

The R functions we need come from the `DBI` package (and `RSQLite`, which provides the SQLite driver):

| Function | Description | Example | Example Description |
|----|----|----|----|
| `dbConnect` | Opens a connection to a database. | `conn <- dbConnect(RSQLite::SQLite(), "file.db")` | Connects to the SQLite database file `file.db` and stores the connection in `conn` |
| `dbListTables` | Lists the tables of a database. | `dbListTables(conn)` | Returns the names of all tables |
| `dbListFields` | Lists the columns of a table. | `dbListFields(conn, "table")` | Returns the column names of the table called `table` |
| `dbGetQuery` | Runs a SQL query and returns the result as a data frame. | `dbGetQuery(conn, "SELECT * FROM table LIMIT 5;")` | Returns the first 5 rows of `table` as a data frame |
| `dbDisconnect` | Closes the connection. | `dbDisconnect(conn)` | Closes the connection stored in `conn` |

In R, we pass the SQL query as a string enclosed in quotes. The SQL syntax remains the same as what we learned in the previous sections.

## Demo

| **Important** | Please use `seminar-r` for this part. |
|:--------------|:--------------------------------------|

We will show how to make SQL calls from R, so that we can (for example)
make plots related to our SQL queries. When we make a SQL call in R, the
data is returned as an R data frame.

Load the libraries we will be using in R:

```r
library(DBI)
library(RSQLite)
library(dplyr)
library(ggplot2)
```

Connect to the IMDb SQLite database

```r
conn <- dbConnect(RSQLite::SQLite(), "/anvil/projects/x-cis220051/data/wsds/imdbSept2026.db")
```

|  |  |
|--|------------|
| Tip | To make sure we never change the data by accident, we can open the database as read-only: `dbConnect(RSQLite::SQLite(), "...", flags = RSQLite::SQLITE_RO)`. The database used for this lesson is already read-only due to file permissions. |

Check available tables:

```r
dbListTables(conn)
```

and the columns of one of them:

```r
dbListFields(conn, "basics")
```

Write the first SQL query in R:

```r
dbGetQuery(conn, "SELECT * FROM basics LIMIT 5;")
```

The result looks like a table, but it is a regular R data frame. We can store it in an object and use it like any other data frame in R:

```r
first_titles <- dbGetQuery(conn, "SELECT * FROM basics LIMIT 5;")

class(first_titles)
first_titles$primaryTitle
```





## Practice on your own


