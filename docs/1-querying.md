<a id="basics-of-querying"></a>

# 1. Basics of Querying

By the end of this section you will be able to:

- explain what `SELECT`, `SELECT DISTINCT` and `LIMIT` do, and combine them in one query
- give a column or a table a temporary name with an alias (`AS`)
- connect to a SQLite database from a JupyterLab notebook and explore its tables
- count the records in a table with `COUNT(*)`

<a id="_concepts"></a>

## Concepts

| Command | Description | Example | Example Description |
|----|----|----|----|
| `SELECT` | Used to select data from a database. | `SELECT * FROM table;` | Selects all columns from the table called `table` |
| `SELECT DISTINCT` | Used to select unique values from one or more columns. Duplicate rows (for the given columns) will not be selected. | `SELECT DISTINCT column1, column2 FROM table;` | Selects all rows that are not duplicate combinations from columns 1 and 2 from the table called `table` |
| `LIMIT` | Used to specify the number of records to return. | `SELECT * FROM table LIMIT 10;` | Selects all columns from the table called `table` and limits the results to 10 records |
| `AS` | Used to give a column or a table a temporary name (an alias). | `SELECT column1 AS c1 FROM table;` | Selects `column1` from the table called `table` and displays it under the name `c1` |
| `COUNT(*)` | Used to count the number of rows. | `SELECT COUNT(*) FROM table;` | Returns the number of records in the table called `table` |

<a id="_demo"></a>

## Demo

For today’s practices, please select the `seminar` kernel in Anvil. To run SQL
queries in a JupyterLab notebook, first run the following in a cell at
the top of your notebook to establish a connection with the database:

```
%sql sqlite:////anvil/projects/x-cis220051/data/wsds/imdbSept2026.db
```

|  |  |
|--|------------|
| Tip | Start every SQL cell with `%%sql` as its first line, otherwise Jupyter treats the cell as Python code and returns a syntax error. Run each query in its own cell: when a cell contains several queries, only the result of the last one is displayed. |

This is a snapshot of the IMDb public datasets from September 2026. It
is a large database (about 22 GB), so always limit the number of rows
you display. It consists of seven tables:

| Table | What it contains | Key columns |
|----|----|----|
| `basics` | One row per title (movie, TV series, episode, ...): type, titles, years, runtime, genres | `tconst` |
| `akas` | Alternative titles of a title, e.g. translated or regional titles | `titleId` (same as `tconst`) |
| `crew` | The directors and writers of a title | `tconst` |
| `episode` | Links each episode to its parent series, with season and episode number | `tconst`, `parentTconst` |
| `principals` | The principal cast and crew of a title and their role | `tconst`, `nconst` |
| `ratings` | The average rating and the number of votes of a title | `tconst` |
| `name` | One row per person: name, birth and death year, professions | `nconst` |


<table>
<colgroup>
<col style="width: 12%" />
<col style="width: 88%" />
</colgroup>
<tbody>
<tr>
<td class="icon">Tip</td>
<td class="content"><p>In sqlite, you can show the tables using the
following query:</p>
<pre class="highlight" data-lang="sql"><code>.tables</code></pre>
<p>Unfortunately, sqlite-specific functions can’t be run in a JupyterLab
cell like that. Instead, we need to use a different query as
follows:</p>
<pre class="highlight" data-lang="sql"><code>SELECT tbl_name FROM sqlite_master WHERE type=&#39;table&#39;;</code></pre></td>
</tr>
</tbody>
</table>

See first five lines of `name` table and `basics`:

```sql
%%sql
SELECT * FROM name LIMIT 5;
```

```sql
%%sql
SELECT * FROM basics LIMIT 5;
```

It is a good idea to run this query on each table in the dataset to
identify the common keys that can be used to connect the tables in SQL.
It is also helpful to limit the number of rows you display (`LIMIT`),
viewing an entire table at once is not practical.

|  |  |
|--|------------|
| Tip | `LIMIT` does not choose *which* rows you get. Without further instructions, the database returns rows in whatever order is most convenient for it, so do not assume the first rows are the oldest, the best, or the most popular titles. |

To list the column names and data types of a table without displaying
any rows, use:

```sql
%%sql
PRAGMA table_info(basics);
```

In SQL and certain other languages like bash or batch, the asterisk (\*) acts as a wildcard that represents "everything". For instance, the command `SELECT * FROM table` retrieves all columns from the table named table, regardless of their names, data types, or values. This feature is convenient when you want to extract every piece of data from a table without listing each column one by one. When you only need a few columns, list them by name instead (for example `SELECT primaryTitle, genres FROM basics LIMIT 5;`). The result is easier to read and, on large tables, faster to produce. 

Now, let’s check how many records are in the `name` table:

```sql
%%sql
SELECT COUNT(*) FROM name;
```
There are more than 15 million records in the table, so displaying
the full table would not be feasible. (The `principals` table is even
larger, with more than 100 million rows.)

Let’s take a closer look at the basics table by running the following
query:

```sql
%%sql
SELECT * FROM basics LIMIT 5;
```

As you can see, each row includes a `tconst` that corresponds to the
title of a movie, TV show, or another type of media. But what exactly is
this `tconst`? Take a look at the following link:

<a href="https://www.imdb.com/title/tt0111161"
class="bare">https://www.imdb.com/title/tt0111161</a>

The `tconst` is the identifier IMDb itself uses for a title: it is the
last part of the web address of the title’s page (here `tt0111161`, the
page of *The Shawshank Redemption*). Because the same identifier is used
in several tables (`basics`, `crew`, `episode`, `principals`, `ratings`,
and as `titleId` in `akas`), it is the key that lets us connect them
later on. In the same way, `nconst` identifies a person and connects the
`name` table to `principals`.

### Selecting unique values with `SELECT DISTINCT`

A column such as the type of a title contains the same value over and
over. Compare the next two queries:

```sql
%%sql
SELECT titleType FROM basics LIMIT 10;
```

```sql
%%sql
SELECT DISTINCT titleType FROM basics;
```

The first query returns ten rows, repeated values included. The second
one returns every different value that appears in the column exactly
once. (It has to look at all 12 million rows of `basics`, so it may take
a few seconds.) `DISTINCT` also works on several columns at once, in
which case it returns each unique *combination* of the values:

```sql
%%sql
SELECT DISTINCT titleType, isAdult FROM basics;
```

<a id="_aliasing"></a>

### Aliasing

Aliasing is the process of giving a table or a table column a temporary
name. Aliases are commonly used to either make the query easier to
write, or more readable.

Note that aliases only last for the duration of a single query. If we
were to run a query that defines an alias, and subsequently run another
query that uses that alias without defining it again, the second query
would fail.

The general pattern is:

```sql
-- column alias
SELECT column_name AS alias_name FROM table_name;

-- table alias
SELECT alias_name.column_name FROM table_name AS alias_name;
```

The keyword `AS` is optional in SQLite (`FROM basics b` works too), but
writing it makes the intent of the query clearer, so we recommend always
using it.

<a id="_aliasing_examples"></a>

### Aliasing examples

As a next example, let’s put aliases to work on the IMDb tables. 

First, a **column alias** changes the name
under which a column is displayed. Notice the new column headers in the
output:

```sql
%%sql
SELECT primaryTitle AS title, runtimeMinutes AS minutes
FROM basics
LIMIT 5;
```

If an alias should contain spaces or special characters, put it in
double quotes:

```sql
%%sql
SELECT primaryTitle AS "Movie Title", runtimeMinutes AS "Run Time (min)"
FROM basics
LIMIT 5;
```

Aliases are also useful for results that are calculated rather than
stored in a column, since these would otherwise get an unreadable
default name:

```sql
%%sql
SELECT COUNT(*) AS number_of_people FROM name;
```

Next, a **table alias** gives a table a short name that we can use to
refer to its columns. This does not change the output here, but it keeps
longer queries readable, and it becomes essential once we work with
several tables at the same time:

```sql
%%sql
SELECT b.primaryTitle, b.genres
FROM basics AS b
LIMIT 5;
```

Finally, remember that an alias only exists for the query that defines
it. The following query reuses `b` without defining it, so it fails with
an error such as `no such column: b.primaryTitle`:

```sql
%%sql
SELECT b.primaryTitle FROM basics LIMIT 5;
```

|  |  |
|--|------------|
| Tip | Choose aliases that are short but meaningful (`b` for `basics`, `n` for `name`). Single letters are fine in short queries, but descriptive names such as `number_of_people` make the output easier for others to read. |

|  |  |
|--|------------|
| Important | The column names in this demo (for example `titleType`, `isAdult`, `primaryTitle`, `runtimeMinutes`) are the ones of the `imdbSept2026.db` tables, including the capital letters. If a query returns `no such column`, check the exact spelling with `PRAGMA table_info(basics);`. |

<a id="_common_mistakes"></a>

### Common mistakes

| Mistake | What happens | Fix |
|----|----|----|
| Forgetting `%%sql` at the top of a SQL cell | `SyntaxError: invalid syntax`, because the cell is run as Python | Make `%%sql` the first line of every SQL cell |
| Forgetting the `FROM` part (a missing semicolon is fine when a cell contains a single query, but it is needed to separate several queries) | Error, for example `no such column` | Check that the query reads `SELECT ... FROM ...;` |
| Misspelling a table or column name | `no such table` or `no such column` error | List the tables and columns first (see the tips above) |
| Selecting `*` from a very large table without `LIMIT` | The cell takes very long or the notebook becomes unresponsive | Always add `LIMIT` while exploring |
| Using an alias in a later query | `no such column` error | Define the alias again in every query that uses it |

<a id="_practice_on_your_own"></a>

## Practice on your own

Now, you’ll practice the concepts above on your own with the Lahman
Baseball data.

```
%sql sqlite:////anvil/projects/tdm/data/lahman/lahman.db
```

Example 1. Deliverables

1.1. Load the SQL database into a queryable SQLite database.

1.2. How many tables are in the database?

1.3. List the names of all the tables in the database.

1.4. Write a SQL query to select all columns from the `People` table,
limiting the results to 5 records.

Now focus on the `People` table.

1.5. How many columns are in the `People` table?

1.6. What is the name of the column that contains the player’s first
name?

1.7. For a players deathYear, what is put in the column if the player is
still alive?

1.8. What is Don Aase’s playerID?

