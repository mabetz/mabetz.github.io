

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

Before the bigger question, a quick warm-up that is fast and ends with a plot. 
How many movies were released each year since 2000? SQL counts the rows and R draws the picture:

```r
movies_per_year <- dbGetQuery(conn, "
SELECT startYear, COUNT(*) AS n
FROM basics
WHERE titleType = 'movie' AND startYear >= 2000 AND startYear <= 2026
GROUP BY startYear
ORDER BY startYear;
")

ggplot(movies_per_year, aes(x = startYear, y = n)) +
  geom_line(color = "dodgerblue3") +
  geom_point(color = "dodgerblue3") +
  labs(title = "Movies Released Each Year",
       x = "Year", y = "Number of Movies") +
  theme_minimal()
```
Notice that the database returned only 27 rows, even though it looked at millions of 
rows to produce them. This is the main reason to use SQL before R.

Start asking some questions: which types of titles get the most votes?
We count the titles with at least 100,000 votes for each `titleType`.
The votes are in the `ratings` table and the type is in the `basics`
table, so we join them on `tconst`.

```r
system.time(
  votes_by_type <- dbGetQuery(conn, "
SELECT b.titleType, COUNT(*) AS n
FROM ratings AS r
JOIN basics AS b ON r.tconst = b.tconst
WHERE r.numVotes >= 100000
GROUP BY b.titleType
ORDER BY n DESC;
")
)

votes_by_type
```

`system.time()` measures how long the query takes (look at the `elapsed`
value, in seconds). This query should be fast: `ratings` is a small table
(about 1.7 million rows) and the `WHERE` condition keeps only the titles
with many votes, so the database only has to look up those few titles in
`basics`.

|  |  |
|--|------------|
| Tip | A query is fast when the database can jump directly to the rows it needs. Every table in `imdbSept2026.db` has an index that starts with its ID column (`tconst`, `titleId` or `nconst`), so joining on `tconst`, as we did here, is fast. Searching the `principals` table (more than 100 million rows) for a person by `nconst` cannot use that index, so the database has to read the whole table. In our test, finding all the title types of one person this way took about 6.5 minutes (391 seconds), which is too long for a live demo. |

|  |  |
|--|------------|
| Tip | A slow query does not have to run again and again. Save the result once and load it next time: `saveRDS(votes_by_type, "votes_by_type.rds")` and later `votes_by_type <- readRDS("votes_by_type.rds")`. |

Plot title types

```r
ggplot(votes_by_type, aes(x = reorder(titleType, n), y = n)) +
  geom_bar(stat = "identity", fill = "dodgerblue3") +
  coord_flip() +
  labs(title = "Titles with at Least 100,000 Votes, by Type",
       x = "Title Type", y = "Number of Titles") +
  theme_minimal()
```

Once the result is in R, we can keep working on it with `dplyr` without
going back to the database. For example, add the share of each title type
(we learned in the aggregation section that we can calculate this in SQL
too, in R it is a single line):

```r
votes_by_type %>%
  mutate(percent = round(100 * n / sum(n), 1))
```

When we are done, close the connection:

```r
dbDisconnect(conn)
```

|  |  |
|----|----|
| Note | SQL syntax is NOT case sensitive. However, remember that R is case sensitive, so please be careful with the capitalization of data frame and variable names. |

### Common mistakes

| Mistake | What happens | Fix |
|----|----|----|
| Running `dbGetQuery` before the library is loaded or before `conn` is created | `could not find function` or `object 'conn' not found` error | Run the setup cells first, in order |
| Writing `Conn` or `Votes_by_type` instead of `conn` or `votes_by_type` | `object not found` error, because R is case sensitive | Copy the object name exactly as it was created |
| Using double quotes for the text value inside the SQL, such as `WHERE b.titleType = "movie"` inside an R string that also uses double quotes | The R string ends too early and R shows a syntax error | Use double quotes for the R string and single quotes for the SQL text, as in `'movie'` |
| The column name in `ggplot2` does not match the alias in SQL, for example `aes(y = count)` when the query used `AS n` | `object 'count' not found` error | Use the same name that the query gives to the column (check with `names(votes_by_type)`) |
| Pulling a huge table into R with `SELECT *` | R becomes slow or runs out of memory | Filter, aggregate and use `LIMIT` in SQL first |

## Practice on your own





