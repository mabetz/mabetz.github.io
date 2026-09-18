<a id="filtering-and-sorting"></a>

# 2. Filtering and Sorting

<a id="_concepts"></a>

## Concepts

<a id="_clauses"></a>

### Clauses

| Command | Description | Example | Example Description |
|----|----|----|----|
| `WHERE` | Used to filter records based on a condition. | `SELECT * FROM table WHERE column = value` | Selects all columns from the table called `table` where the `column` equals `value` |
| `ORDER BY` | Used to sort the result set in ascending or descending order. | `SELECT * FROM table ORDER BY column ASC` | Selects all columns from the table called `table` and orders the results by `column` in ascending order |
| `LIMIT` | Used to specify the number of records to return. | `SELECT * FROM table LIMIT 10;` | Selects all columns from the table called `table` and limits the results to 10 records |

<a id="_demo"></a>

## Demo

At this point, you might guess that `tt0111161` is the ID IMDb uses to
identify a movie or TV show. Let’s test that assumption by running a
query on our database to find any titles in the `titles` table that
match the `title_id` provided in the [link we used
before](https://www.imdb.com/title/tt0111161):

```
SELECT * FROM titles
WHERE title_id = 'tt0111161';
```

It is The Shawshank Redemption, the top rated movie in IMDb!

Let’s try this one:

```
SELECT * FROM titles
WHERE title_id = 'tt0108778';
```

You can even find some episodes, when you go to search in
<a href="https://imdb.com" class="bare">https://imdb.com</a> for your
favorite one. For example: "the one where Rose pivots" and we see the
episode is this one (The one where Ross moves in):

<a href="https://www.imdb.com/title/tt0583486"
class="bare">https://www.imdb.com/title/tt0583486</a>

<img src="friends-ross.gif" width="292" height="100"
alt="Image from https://tenor.com/search/ross-pivot-gifs" />

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<tbody>
<tr>
<td class="icon">Note</td>
<td class="content"><p>We can also check for how many unique types we
have in our dataset by using <code>SELECT DISTINCT</code>:</p>
<pre class="highlight" data-lang="sql"><code>SELECT DISTINCT type
FROM titles
ORDER BY type;</code></pre>
<p>If you want to see how many unique combinations of <code>type</code>
and <code>genres</code> exist, you can use the <code>COUNT()</code>
function by combining both columns into one expression using the
concatenation operator <code>||</code> and count the distinct values in
a single line:</p>
<pre class="highlight" data-lang="sql"><code>%%sql
SELECT COUNT(DISTINCT type || genres) FROM titles;</code></pre></td>
</tr>
</tbody>
</table>

Let’s check if `title_id` also works for `episodes` table, too:

```
SELECT * FROM episodes
WHERE episodes_title_id = 'tt0108778';
```

No, since it is the title of the show not the title of the episode. Let
us change it to `show_title_id`:

```
SELECT * FROM episodes
WHERE show_title_id = 'tt0108778';
```

If you try `show_title_id` for `tt0111161`, you will not get any result,
since it is a movie instead of a show.

Let’s see who was staring in Friends:

```
SELECT * FROM crew
WHERE show_title_id = 'tt0108778';
```

We can create a new code cell and write a SQL query to select all
columns from the `titles` table where the `original_title` is 'Friends'
and premiered 1994:

```
SELECT * FROM titles
WHERE original_title = 'Friends' AND premiered = '1994';
```

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<tbody>
<tr>
<td class="icon">Note</td>
<td class="content"><p>You can use the <code>AND</code> operator to
combine multiple conditions within a <code>WHERE</code> clause. Simply
place the operator between each condition you want to evaluate. You can
also use other logical operators. For example, <code>OR</code> checks if
at least one condition is true, and <code>NOT</code> is used to reverse
a condition.</p>
<p>However, orders matters in here. The order of evaluation (operator
precedence) for logical operators is:</p>
<p><code>NOT</code> — evaluated first<br />
<code>AND</code> — evaluated second<br />
<code>OR</code> — evaluated last<br />
</p>
<p>You can test it with the following two commands:</p>
<pre class="highlight" data-lang="sql"><code>SELECT COUNT(*) FROM titles
WHERE original_title = &#39;Friends&#39; OR premiered = 1994 AND ended = 2004;</code></pre>
<pre class="highlight" data-lang="sql"><code>SELECT COUNT(*) FROM titles
WHERE original_title = &#39;Friends&#39; AND premiered = 1994 OR ended = 2004;</code></pre></td>
</tr>
</tbody>
</table>

We can ask more questions and use a new command `ORDER BY`: Find the top
10 longest movies in the dataset:

```
SELECT * FROM titles
WHERE type = 'movie' ORDER BY runtime_minutes DESC LIMIT 10;
```

We can even have some cluse which movies are coming soon:

```
SELECT * FROM titles
WHERE type = 'movie' ORDER BY premiered DESC LIMIT 10;
```

|  |  |
|----|----|
| Note | You should generally use the `LIMIT` command to restrict the number of rows returned by your query. This is especially important when working with large datasets, as it helps improve performance and reduces the amount of data that needs to be processed. |

Example 1. Deliverables

1.1. Run the SQL query to see all tables in the IMDb dataset.\
1.2. Write a SQL query to select all columns from the `titles` table,
limiting the results to 10 records.\
1.3. Write a SQL query to find the duration of Titanic movie premiered
in 1997?\
1.4. What is the oldest movie according to this database?

<a id="_practice_on_your_own"></a>

## Practice on your own

Example 2. Deliverables

2.1. Write a SQL query to select all columns from the People table where
the player’s last name is "Sanders".

2.2. Write a SQL query to select all columns from the People table where
the player’s last name is "Sanders" and the birth year is before 1900.

2.3. Write a SQL query to select all columns from the People table where
the player’s last name is "Sanders" and the birth year is before 1900 or
after 1975.

2.4. Write a SQL query to select all players born after 1970, who are
over 70 inches tall, ordered by their last name in descending order,
limiting the results to 10 records.

2.5. Write a SQL query to select all players born after 1970, who are
over 70 inches tall, ordered by their last name in descending order,
limiting the results to 20 records. (This will throw an error.)

Last updated 2026-09-18 19:23:08 UTC
