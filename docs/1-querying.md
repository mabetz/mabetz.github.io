<a id="basics-of-querying"></a>

# 1. Basics of Querying

<a id="_concepts"></a>

## Concepts

| Command | Description | Example | Example Description |
|----|----|----|----|
| `SELECT` | Used to select data from a database. | `SELECT * FROM table;` | Selects all columns from the table called `table` |
| `SELECT DISTINCT` | Used to select unique values from one or more columns. Duplicate rows (for the given columns) will not be selected. | `SELECT DISTINCT column1, column2 FROM table;` | Selects all rows that are not duplicate combinations fro columns 1 and 2 from the table called `table` |
| `LIMIT` | Used to specify the number of records to return. | `SELECT * FROM table LIMIT 10;` | Selects all columns from the table called `table` and limits the results to 10 records |

<a id="_aliasing"></a>

### Aliasing

Aliasing is the process of giving a table or a table column a temporary
name. Aliases are commonly used to either make the query easier to
write, or more readable.

Note that aliases only last for the duration of a single query. If we
were to run the previous query, and subsequently run the following
query, it would fail.

<a id="_demo"></a>

## Demo

|  |  |
|----|----|
| Important | Fulya **Add Aliasing examples below <a href="https://the-examples-book.com/tools/sql/aliasing"
class="bare">https://the-examples-book.com/tools/sql/aliasing</a>** |

For today’s practices, please select the `seminar` kernel. To run SQL
queries in a JupyterLab notebook, first run the following in a cell at
the top of your notebook to establish a connection with the database:

```
%sql sqlite:////anvil/projects/tdm/data/movies_and_tv/imdb.db
```

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
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

See first five lines of `people` table and `titles`:

```
SELECT * FROM people LIMIT 5;

SELECT * FROM titles LIMIT 5;
```

It is a good idea to run this query on each table in the dataset to
identify the common keys that can be used to connect the tables in SQL.
It is also helpful to limit the number of rows you display (`LIMIT`),
viewing an entire table at once is not practical.

|  |  |
|----|----|
| Tip | In SQL and certain other languages like bash or batch, the asterisk (\*) acts as a wildcard that represents "everything". For instance, the command `SELECT * FROM table` retrieves all columns from the table named table, regardless of their names, data types, or values. This feature is convenient when you want to extract every piece of data from a table without listing each column one by one. |

Now, let’s check how many records are in the people table:

```
SELECT COUNT(*) FROM people;
```

As you can see, there are more than 11 million records, so displaying
the full table would not be feasible.

Let’s take a closer look at the titles table by running the following
query:

```
SELECT * FROM titles LIMIT 5;
```

As you can see, each row includes a `title_id` that corresponds to the
title of a movie, TV show, or another type of media. But what exactly is
this `title_id`? Take a look at the following link:

<a href="https://www.imdb.com/title/tt0111161"
class="bare">https://www.imdb.com/title/tt0111161</a>

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

Last updated 2026-09-18 19:21:08 UTC
