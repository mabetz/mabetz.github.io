

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
then keep working with it in R, for example to make a plot. The `imdbSept2026.db` database is about 22 GB, too big to load into R memory, so the idea is to let the database do the heavy work and bring back only a small result:

| Step | Where | What happens |
|--|----|-------|
| 1 | R | We send a SQL query to the database as a text string |
| 2 | Database | SQLite filters, joins and aggregates millions of rows |
| 3 | R | The small result comes back as a data frame |
| 4 | R | We plot or reshape it with `ggplot2` and `dplyr` |

The R functions we need come from the `DBI` package (and `RSQLite`, which
provides the SQLite driver):

| Function | Description | Example | Example Description |
|----|----|----|----|
| `dbConnect` | Opens a connection to a database. | `conn <- dbConnect(RSQLite::SQLite(), "file.db")` | Connects to the SQLite database file `file.db` and stores the connection in `conn` |
| `dbListTables` | Lists the tables of a database. | `dbListTables(conn)` | Returns the names of all tables |
| `dbListFields` | Lists the columns of a table. | `dbListFields(conn, "table")` | Returns the column names of the table called `table` |
| `dbGetQuery` | Runs a SQL query and returns the result as a data frame. | `dbGetQuery(conn, "SELECT * FROM table LIMIT 5;")` | Returns the first 5 rows of `table` as a data frame |
| `dbDisconnect` | Closes the connection. | `dbDisconnect(conn)` | Closes the connection stored in `conn` |

The SQL query itself is just a text string between quotes, so the SQL we
learned in the previous sections works here without any changes.



## Demo

|           |                                       |
|-----------|---------------------------------------|
| Important | Please use `seminar-r` for this part. |

We will show how to make SQL calls from R, so that we can (for example)
make plots related to our SQL queries. When we make a SQL call in R, the
data is returned as an R database.

Connect to the IMDb SQLite database

```
conn <- dbConnect(RSQLite::SQLite(), "/anvil/projects/tdm/data/movies_and_tv/imdb.db")
```

Also load libraries we will be using in R:

```
library(RSQLite)
library(dplyr)
library(ggplot2)
```

Check available tables:

```
dbListTables(conn)
```

Write the first SQL query in R:

```
dbGetQuery(conn, "SELECT * FROM titles LIMIT 5;")
```

Start asking some questions: what type of titles Quentin Tarantino
mainly works on (following query takes some time)

```
tarantino_types <- dbGetQuery(conn, "
SELECT t.type, COUNT(*) AS count
FROM titles AS t
JOIN crew AS c ON t.title_id = c.title_id
JOIN people AS p ON c.person_id = p.person_id
WHERE p.name = 'Quentin Tarantino'
GROUP BY t.type
ORDER BY count DESC;
")

tarantino_types
```

Plot title types

```
ggplot(tarantino_types, aes(x = reorder(type, count), y = count)) +
  geom_bar(stat = "identity", fill = "#0073C2FF") +
  coord_flip() +
  labs(title = "Types of Titles Quentin Tarantino Worked On",
       x = "Title Type", y = "Count") +
  theme_minimal()
```

|  |  |
|----|----|
| Note | SQL syntax is NOT case sensitive. However, remember that R is case sensitive, so please be careful with the capitalization of data frame and variable names. |


## Practice on your own


