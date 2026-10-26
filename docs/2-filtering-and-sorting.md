
# 2. Filtering and Sorting


## Concepts


### Clauses

| Command | Description | Example | Example Description |
|----|----|----|----|
| `WHERE` | Used to filter records based on a condition. | `SELECT * FROM table WHERE column = value` | Selects all columns from the table called `table` where the `column` equals `value` |
| `ORDER BY` | Used to sort the result set in ascending or descending order. | `SELECT * FROM table ORDER BY column ASC` | Selects all columns from the table called `table` and orders the results by `column` in ascending order |
| `LIMIT` | Used to specify the number of records to return. | `SELECT * FROM table LIMIT 10;` | Selects all columns from the table called `table` and limits the results to 10 records |


## Demo

At this point, you might guess that `tt0111161` is the ID IMDb uses to
identify a movie or TV show. Let’s test that assumption by running a
query on our database to find any titles in the `basics` table that
match the `tconst` provided in the [link we used
before](https://www.imdb.com/title/tt0111161):

```sql
%%sql
SELECT * FROM basics WHERE tconst = 'tt0111161';
```

It is The Shawshank Redemption, the top rated movie in IMDb!

Let’s try this one:

```sql
%%sql
SELECT * FROM basics WHERE tconst ='tt0108778';
```

You can even find some episodes, when you go to search in
<a href="https://imdb.com" class="bare">https://imdb.com</a> for your
favorite one. For example: "the one where Rose pivots" and we see the
episode is this one (The one where Ross moves in): https://www.imdb.com/title/tt0583486 


![GitHub Pages](_images/friends-ross.gif)

Figure 1. Image from https://tenor.com/search/ross-pivot-gifs

```sql
%%sql
SELECT * FROM basics WHERE tconst ='tt0583486';
```

Notice that the first query for `tt0108778` represents the series because `titleType = tvSeries` but the second query for the specific episode  `tt0583486` has `titleType = tvEpisode`

We can also check for how many unique types we have in our dataset by using `SELECT DISTINCT`

```sql
%%sql
SELECT DISTINCT titleType
FROM basics
ORDER BY titleType;
```

If you want to see how many unique combinations of `type`
and `genres` exist, you can use the `COUNT()`
function by combining both columns into one expression using the
concatenation operator `||` and count the distinct values in
a single line:

```sql
%%sql
SELECT COUNT(DISTINCT titleType || genres) FROM basics;
```

Let’s check if `tconst` also works for `episode` table, too:

```sql
%%sql
PRAGMA table_info(episode);
```

```sql
%%sql
SELECT * FROM episode WHERE tconst = 'tt0108778';
```

No, since it is the title of the show not the title of the episode. Let
us change it to `parentTconst`:

```sql
%%sql
SELECT * FROM episode WHERE parentTconst = 'tt0108778';
```

If you try `parentTconst` for `tt0111161`, you will not get any result,
since it is a movie instead of a show.

Let’s see who was staring in Friends:

```sql
%%sql
SELECT * FROM crew 
WHERE tconst = 'tt0108778';
```

We can create a new code cell and write a SQL query to select all
columns from the `episode` table where the `parentTconst` is 'tt0108778' (Friends)
and premiered 1994 (season #1):

```sql
%%sql
SELECT * FROM basics 
WHERE originalTitle = 'Friends' AND startYear = 1994;
```

You can use the `AND` operator to combine multiple conditions within a `WHERE` clause. Simply place the operator between each condition you want to evaluate. You can also use other logical operators. For example, `OR` checks if at least one condition is true, and `NOT` is used to reverse a condition.

However, orders matters in here. The order of evaluation (operator
precedence) for logical operators is:

    `NOT` — evaluated first
    `AND` — evaluated second
    `OR` — evaluated last


You can test it with the following two commands:

```sql
%%sql
SELECT COUNT(*) FROM basics
WHERE originalTitle = 'Friends' OR startYear = 1994 AND endYear = 2004;
```

```sql
%%sql
SELECT COUNT(*) FROM basics
WHERE originalTitle = 'Friends' AND startYear = 1994 OR endYear = 2004;
```

We can ask more questions and use a new command `ORDER BY`. The default is to sort by ascending order so use `DESC` to sort by descending order. Find the top
10 longest movies in the dataset:

```sql
%%sql
SELECT * FROM basics
WHERE titleType = 'movie' AND runtimeMinutes != '\N'
ORDER BY runtimeMinutes DESC LIMIT 10;
```

We can even have some clues which movies are coming soon:

```sql
%%sql
SELECT * FROM basics
WHERE titleType = 'movie' AND startYear != '\N'
ORDER BY startYear DESC LIMIT 10;
```


Note:  You should generally use the `LIMIT` command to restrict the number of rows returned by your query. This is especially important when working with large datasets, as it helps improve performance and reduces the amount of data that needs to be processed. 



## Practice on your own

Now, you’ll practice the concepts above on your own with the Lahman Baseball data.

```sql
%sql sqlite:////anvil/projects/tdm/data/lahman/lahman.db
```

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
limiting the results to 20 records. (This will throw an error. Explain the error.)


