# 3. Aggregation and Grouping

<!--
By the end of this section you will be able to:

- summarize data with the aggregate functions `COUNT`, `SUM`, `AVG`, `MIN` and `MAX`
- group rows with `GROUP BY` and calculate a summary for each group
- filter rows with `WHERE` and filter groups with `HAVING`, and explain the difference
- avoid the integer division trap when you calculate proportions and percentages
-->

## Concepts

| Command | Description | Example | Example Description |
|----|----|----|----|
| `GROUP BY` | Used to group rows with the same values in specified columns into summary row(s). | `SELECT column, COUNT(*) FROM table GROUP BY column;` | Selects the `column` and counts the number of records in each group, grouping by the `column` |
| `HAVING` | Used to filter the groups after the aggregation. It works like `WHERE`, but on the summarized data. | `SELECT column, COUNT(*) FROM table GROUP BY column HAVING COUNT(*) > 10;` | Counts the records in each group and keeps only the groups that have more than 10 records |
| `CAST` | Used to convert a value to another data type. | `SELECT CAST(column AS REAL) FROM table;` | Returns the values of `column` as decimal numbers |


### Aggregate functions

An aggregate function takes many rows and returns a single value. When it is
used together with `GROUP BY`, it returns one value for each group.

| Function | Description | Example |
|----|----|----|
| `COUNT` | Counts rows. `COUNT(*)` counts all rows, `COUNT(column)` counts only the rows where `column` is not empty (not `NULL`). | `SELECT COUNT(*) FROM table;` |
| `SUM` | Adds up the values of a numeric column. | `SELECT SUM(column) FROM table;` |
| `AVG` | Calculates the average of a numeric column. | `SELECT AVG(column) FROM table;` |
| `MIN` | Returns the smallest value of a column. | `SELECT MIN(column) FROM table;` |
| `MAX` | Returns the largest value of a column. | `SELECT MAX(column) FROM table;` |

|  |  |
|--|------------|
| Tip | `SUM`, `AVG`, `MIN`, `MAX` and `COUNT(column)` skip the empty (`NULL`) values. For example, `AVG(runtimeMinutes)` is the average of only the titles that have a runtime. `COUNT(*)` does not skip anything, it counts every row. |

### GROUP BY

Another useful command in SQL is the `GROUP BY`. This command allows us
to group rows together if they have the same values in one or more
specified columns. It is especially helpful when we want to summarize or
aggregate data, for instance, counting the number of titles per genre or
calculating the average rating for each year.

Similar to how the `WHERE` command can be combined with logical
operators such as `AND` and `OR`, the `GROUP BY` command is often used
together with aggregation functions such as `COUNT`, `SUM`, `AVG`,
`MIN`, and `MAX`. These functions let us perform calculations on each
group of data.

<!--
A good rule of thumb: every column in the `SELECT` part should either be
in the `GROUP BY` or be inside an aggregate function. SQLite does not
stop you if you break this rule, but the value it shows for the other
columns is taken from an arbitrary row of the group, so it is not
reliable.
-->

### HAVING

`HAVING` is similar to the `WHERE` clause but instead of being applied
to every row *before* grouping, it is applied on the aggregated data. A
single query can have both a `WHERE` clause and a `HAVING` clause. The
`WHERE` clause will filter the data *before* aggregation, and the
`HAVING` clause will filter the data *after* aggregation.

This is also the reason why you cannot use an aggregate function in
`WHERE`: when `WHERE` runs, the groups do not exist yet. The database
processes the parts of a query in this logical order:

| Order | Part | What happens |
|--|----|-------|
| 1 | `FROM` | Picks the table |
| 2 | `WHERE` | Filters the rows |
| 3 | `GROUP BY` | Forms the groups |
| 4 | `HAVING` | Filters the groups |
| 5 | `SELECT` | Calculates the columns and aggregates to show |
| 6 | `ORDER BY` | Sorts the result |
| 7 | `LIMIT` | Keeps only the first rows |

## Demo

If you are starting a new notebook, connect to the database first (use the
`seminar` kernel):

```
%sql sqlite:////anvil/projects/x-cis220051/data/wsds/imdbSept2026.db
```

|  |  |
|--|------------|
| Tip | The `basics` table has about 12.7 million rows and `akas` has almost 60 million rows. Queries that group a whole table can take a few seconds (`basics`) or even a minute or more (`akas`). This is normal, be patient and do not run the same cell again while it is running. |

For example, we want to find the number of rows for each `titleType` in
`basics` table:

```sql
%%sql
SELECT titleType, COUNT(*) FROM basics GROUP BY titleType;
```

The result has one row for each type of title, and the `COUNT(*)` column
tells us how many rows were in that group.

Add average run time in minutes for each type:

```sql
%%sql
SELECT titleType, COUNT(*), AVG(runtimeMinutes) FROM basics GROUP BY titleType;
```

The headers of the calculated columns are not easy to read. As we learned
in the aliasing section, we can give them clear names with `AS`:

```sql
%%sql
SELECT titleType, COUNT(*) AS n, AVG(runtimeMinutes) AS avg_runtime
FROM basics
GROUP BY titleType;
```

Or we only care about the movies:

```sql
%%sql
SELECT titleType, COUNT(*) AS n, AVG(runtimeMinutes) AS avg_runtime
FROM basics WHERE titleType = 'movie';
```

Here `WHERE` keeps only the movies before the calculation, so only one
group is left and we do not need a `GROUP BY`.

In another example, suppose we want to find the average runtime of
movies in each genre. We can use the `WHERE` command to filter the
results to only include movies (for instance, `titleType = 'movie'`), the
`GROUP BY` command to group the data by genre, and the `AVG` function to
calculate the average runtime for each group. The SQL query would look
like this:

```sql
%%sql
SELECT genres, AVG(runtimeMinutes) AS avg_runtime
FROM basics
WHERE titleType = 'movie'
GROUP BY genres;
```

|  |  |
|--|------------|
| Important | Look closely at the `genres` column in the result. A title can have up to three genres and they are stored together in one text value, for example `Action,Adventure,Sci-Fi`. So `GROUP BY genres` creates one group for each *combination* of genres, not for each single genre. The result has many rows, so add `LIMIT 10` while you explore. |

We can check only for 'Action':

```sql
%%sql
SELECT AVG(runtimeMinutes) AS avg_runtime
FROM basics
WHERE titleType = 'movie'
  AND genres LIKE '%Action%';
```

The `%` in `LIKE` is a wildcard that stands for any text, so
`'%Action%'` matches every genre list that contains the word Action, no
matter where it is in the list. This is how we can look at a single genre
even though the genres are stored together.

Notice that `COUNT(*)` is the only aggregate function that can use the
wildcard (\*), because to count rows we do not need to look at a specific
column. The other functions (`SUM`, `AVG`, `MIN` and `MAX`) always need a
column to work on. In the last example, we use the `AVG` function to
calculate the average of the `runtimeMinutes` column. The `AS` keyword
then lets us assign a clear name, `avg_runtime`, to the resulting column,
which helps make our query output easier to read and interpret.

Another example: find the number of alternative titles (rows in `akas`)
for each language. We also sort the result, largest group first, and show
only the first 10 rows:

```sql
%%sql
SELECT language, COUNT(*) AS n FROM akas
GROUP BY language
ORDER BY n DESC
LIMIT 10;
```

Dr. Ward found all the tconst for Marvel Cinematic Universe (MCU)
before, we use them for this example to find how many we have each year:

```sql
%%sql
SELECT startYear AS year, COUNT(*) AS n FROM basics
WHERE tconst IN ('tt0371746', 'tt0800080', 'tt1228705', 'tt0800369', 'tt0458339',
                 'tt0848228', 'tt1300854', 'tt1981115', 'tt1843866', 'tt2015381',
                 'tt2395427', 'tt0478970', 'tt3498820', 'tt1211837', 'tt3896198',
                 'tt2250912', 'tt3501632', 'tt1825683', 'tt4154756', 'tt5095030',
                 'tt4154664', 'tt4154796', 'tt6320628', 'tt3480822', 'tt9032400',
                 'tt9376612', 'tt9419884', 'tt10648342', 'tt9114286')
GROUP BY startYear
ORDER BY year;
```

Let’s modify this query to get the percentage of MCU movies released in
each year. First find the total:

```sql
%%sql
SELECT COUNT(*) FROM basics
WHERE tconst IN ('tt0371746', 'tt0800080', 'tt1228705', 'tt0800369', 'tt0458339',
                 'tt0848228', 'tt1300854', 'tt1981115', 'tt1843866', 'tt2015381',
                 'tt2395427', 'tt0478970', 'tt3498820', 'tt1211837', 'tt3896198',
                 'tt2250912', 'tt3501632', 'tt1825683', 'tt4154756', 'tt5095030',
                 'tt4154664', 'tt4154796', 'tt6320628', 'tt3480822', 'tt9032400',
                 'tt9376612', 'tt9419884', 'tt10648342', 'tt9114286');
```

The total should be 29. When you divide the counts to total, you get zeros:

```sql
%%sql
SELECT startYear AS year, COUNT(*)/29 AS n FROM basics
WHERE tconst IN ('tt0371746', 'tt0800080', 'tt1228705', 'tt0800369', 'tt0458339',
                 'tt0848228', 'tt1300854', 'tt1981115', 'tt1843866', 'tt2015381',
                 'tt2395427', 'tt0478970', 'tt3498820', 'tt1211837', 'tt3896198',
                 'tt2250912', 'tt3501632', 'tt1825683', 'tt4154756', 'tt5095030',
                 'tt4154664', 'tt4154796', 'tt6320628', 'tt3480822', 'tt9032400',
                 'tt9376612', 'tt9419884', 'tt10648342', 'tt9114286')
GROUP BY startYear;
```

Why?

When we calculate a percentage using integer (numeric data without a
decimal place) columns, SQLite performs integer division. That means it
divides the numbers as integers and truncates any decimal part, instead
of giving a decimal (floating-point) result. In our query, every year has
fewer than 29 titles, so a value like 4/29 is cut down to 0. For example:

```sql
%%sql
SELECT 5/2;
```

This returns the integer part (2) in SQLite, not 2.5, because both 5 and
2 are integers. We can fix it converting at least one of them to real
number as following by :

```sql
%%sql
SELECT CAST(5 AS REAL)/2 AS result;
```

`REAL` is equivalent to a float or double, meaning a number that can
include decimal values. Writing one of the numbers with a decimal point
(`5.0/2`) gives the same result.

Let’s fix it in the real data:

```sql
%%sql
SELECT startYear AS year, CAST(COUNT(*) AS real)/29 AS n FROM basics
WHERE tconst IN ('tt0371746', 'tt0800080', 'tt1228705', 'tt0800369', 'tt0458339',
                 'tt0848228', 'tt1300854', 'tt1981115', 'tt1843866', 'tt2015381',
                 'tt2395427', 'tt0478970', 'tt3498820', 'tt1211837', 'tt3896198',
                 'tt2250912', 'tt3501632', 'tt1825683', 'tt4154756', 'tt5095030',
                 'tt4154664', 'tt4154796', 'tt6320628', 'tt3480822', 'tt9032400',
                 'tt9376612', 'tt9419884', 'tt10648342', 'tt9114286')
GROUP BY startYear;
```

The values are now proportions between 0 and 1. To get real percentages,
multiply by 100 and round the result with `ROUND(value, number_of_decimals)`.
Here `100.0` makes sure that the division uses decimals:

```sql
%%sql
SELECT startYear AS year, ROUND(100.0 * COUNT(*) / 29, 1) AS percent FROM basics
WHERE tconst IN ('tt0371746', 'tt0800080', 'tt1228705', 'tt0800369', 'tt0458339',
                 'tt0848228', 'tt1300854', 'tt1981115', 'tt1843866', 'tt2015381',
                 'tt2395427', 'tt0478970', 'tt3498820', 'tt1211837', 'tt3896198',
                 'tt2250912', 'tt3501632', 'tt1825683', 'tt4154756', 'tt5095030',
                 'tt4154664', 'tt4154796', 'tt6320628', 'tt3480822', 'tt9032400',
                 'tt9376612', 'tt9419884', 'tt10648342', 'tt9114286')
GROUP BY startYear
ORDER BY year;
```

Let’s continue to practice more by writing a query that returns the
average number of words in the primaryTitle column, by year. First, we
count the words of each title. The number of spaces in a title is the
difference between its length and its length after we remove all spaces
with `replace`, and the number of words is one more than the number of
spaces:

```sql
%%sql
SELECT startYear, primaryTitle,
length(primaryTitle) - length(replace(primaryTitle, ' ', '')) + 1 AS n_words
FROM basics LIMIT 5;
```

Then, only for years where the average number of words in the
primaryTitle is less than 3:

```sql
%%sql
SELECT startYear,
AVG(length(primaryTitle) - length(replace(primaryTitle, ' ', '')) + 1) AS avg_title_length
FROM basics GROUP BY startYear
HAVING avg_title_length < 3;
```

When querying the IMDb dataset, we found a title with `startYear = 2115`: 100 Years, a short film directed by Robert Rodriguez and starring John Malkovich. The film was produced in 2015 but is scheduled to be released on November 18, 2115, exactly 100 years later. It is being kept in a specially designed safe that is intended to remain locked until its release.

This is a useful reminder that unusual values in a dataset are not necessarily errors. Understanding the context and meaning of a variable is important before treating an observation as an outlier or a data quality issue.


|  |  |
|--|------------|
| Tip | SQLite lets us use the alias `avg_title_length` in `HAVING`. Some other databases (for example PostgreSQL) do not, and you would need to write the whole `AVG(...)` expression again in `HAVING`. This query calculates the words of all 12.7 million titles, so it can take a while. Also note that it assumes the words are separated by a single space. |

### Common mistakes

| Mistake | What happens | Fix |
|----|----|----|
| Using an aggregate function in `WHERE`, such as `WHERE COUNT(*) > 10` | `misuse of aggregate` error | Filter groups with `HAVING COUNT(*) > 10` |
| Selecting a column that is not in `GROUP BY` and not inside an aggregate function | No error, but the value comes from an arbitrary row of the group | Add the column to `GROUP BY` or wrap it in an aggregate function |
| Dividing integer by integer, such as `COUNT(*)/29` | The result is cut to a whole number, often 0 | Convert one number with `CAST(... AS REAL)` or write `100.0 * ...` |
| Grouping by a column that stores lists, such as `genres` | One group for each combination, not each single value | Use `WHERE genres LIKE '%Action%'` to look at one value |
| Expecting `COUNT(column)` and `COUNT(*)` to be equal | `COUNT(column)` skips empty (`NULL`) values | Use `COUNT(*)` to count rows and `COUNT(column)` to count filled values |
| Grouping a very large table without a `WHERE` or `LIMIT` | The cell runs for a long time | Filter first, and use `LIMIT` while you explore |

## Practice on your own

Now, you’ll practice the concepts above on your own with the Lahman Baseball data. Remember to start every SQL cell with `%%sql`.

```
%sql sqlite:////anvil/projects/tdm/data/lahman/lahman.db
```

Example 3. Deliverables

3.1. Write a SQL query to count the number of players born before 1900 who are over 74 inches tall.

3.2. Write a SQL query to return the average weight of players grouped by their birth year, ordered by the average weight in descending order, limiting the results to 10 records.

|  |  |
|--|------------|
| Hint | In 3.1 you only need one number, so you do not need `GROUP BY`: combine the two conditions in `WHERE` with `AND`. In 3.2, the question already tells you the parts of the query: `AVG` for the average, `GROUP BY` for the birth year, then `ORDER BY ... DESC` and `LIMIT`. Use the `People` table, and check the column names with `PRAGMA table_info(People);`. |

More practice with the IMDb data. Connect to the database again:

```
%sql sqlite:////anvil/projects/x-cis220051/data/wsds/imdbSept2026.db
```

3.3. How many titles of each `titleType` are in the `basics` table? Show the most frequent type first.

3.4. What is the average `runtimeMinutes` of movies for each `startYear`, for the years 2015 and later? Sort the result by year.

3.5. Which years have more than 10,000 movies? Show the year and the number of movies. (Hint: this needs both `WHERE` and `HAVING`.)

3.6. In the `ratings` table, how many titles have at least 100,000 votes (`numVotes`), and what is the average `averageRating` of these titles?


