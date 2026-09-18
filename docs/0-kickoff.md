<a id="kick-off"></a>

# 0. Kick-off

<a id="_welcome"></a>

## Welcome!

Welcome to the half day short course titled *Applied SQL for Data
Analysis: A Practical Introduction to a Key Data Skill*.

<a id="_agenda"></a>

## Agenda

Wednesday, November 12, 2025, 8:00 AM to noon

| Time                | Minutes | Topic                             |
|---------------------|---------|-----------------------------------|
| 8:00 - 8:20 AM      | 20      | 0\. Kick off                      |
| 8:20 - 9:00 AM      | 40      | 1\. Basics of Querying            |
| 9:00 - 9:40 AM      | 40      | 2\. Filtering and Sorting         |
| 9:40 - 9:45 AM      | 5       | Break                             |
| 9:45 - 10:25 AM     | 40      | 3\. Aggregation and Grouping      |
| 10:25 - 11:05 AM    | 40      | 4\. Joins: Combining Tables       |
| 11:05 - 11:10 AM    | 5       | Break                             |
| 11:10 - 11:50AM     | 40      | 5\. SQL in R                      |
| 11:50 AM - 12:00 PM | 10      | Wrap up                           |
| On your own         |         | 6\. Real-World Querying Challenge |

<img src="sql_meme.jpg" width="792" height="500" alt="Meme" />

Figure 1. SQL Meme
<a id="_introductions"></a>

## Introductions

- Fulya (fgokalp(AT)purdue.edu)

- Maggie (betz(AT)purdue.edu)

- Kali

- Participants

<a id="_about_structured_query_language_sql"></a>

## About Structured Query Language (SQL)

Structured Query Language (SQL) is a language used for querying and
manipulating data in a database and it provides a relational database
management system. SQL allows users to efficiently create, read, update,
and delete data in databases. It is widely used across applications,
from small projects to large-scale industry systems. For data analysts,
data scientists, and software developers, understanding SQL is essential
for working effectively with databases.

We use [SQLite](https://sqlite.org), which does not require a server to
run. The database is built into the application, and the app reads from
and writes to the database files directly on disk. SQLite source code is
public and free to everyone.

<a id="_datasets"></a>

## Datasets

JupyterLab on [Anvil](https://www.rcac.purdue.edu/compute/anvil), a
supercomputer supported by the National Science Foundation, will be used
for this workshop.

<a id="_imdb"></a>

### IMDb

We will use the
[IMDb](https://developer.imdb.com/non-commercial-datasets/) SQLite data
located at: `/anvil/projects/tdm/data/movies_and_tv/imdb.db`\[1\] for
all **demo** content.

The following is an illustration of the database to help you understand
the data.

<img src="figure14.webp" width="792" height="500"
alt="Database diagram from https://dbdiagram.io" />

Figure 2. Database diagram from
<a href="https://dbdiagram.io" class="bare">https://dbdiagram.io</a>

\[1\] There are 7 data files provided by IMDb:

```
name.basics.tsv.gz
title.akas.tsv.gz
title.basics.tsv.gz
title.crew.tsv.gz
title.episode.tsv.gz
title.principals.tsv.gz
title.ratings.tsv.gz
```

We will use the SQLite database format created from those tsv
(Tab-Separated Values) files.

<a id="_lahman_baseball_database"></a>

### Lahman Baseball Database

We will use the [Lahman Baseball Database](http://seanlahman.com/). The
SQLite database on Anvil is located at
`/anvil/projects/tdm/data/lahman/lahman.db`.

We will use the Lahman data for all **practice on your own** content.

The following is an illustration of the database to help you understand
the data.

<img src="lahmandb.png" width="792" height="500"
alt="Database diagram from https://dbdiagram.io" />

Figure 3. Database diagram from
<a href="https://sabr.org/lahman-database/"
class="bare">https://sabr.org/lahman-database/</a>
<a id="_login_in_to_anvil"></a>

## Login in to Anvil

Let’s launch a Jupyter session on Anvil by navigating to
<https://notebook.anvilcloud.rcac.purdue.edu>.

Use the ACCESS username that you were assigned (when you setup your
account) and the ACCESS password that you chose. You will need to
authenticate via DUO.

Now launch two notebooks. Since sqlite only references the most recent
database, we recommend having two notebooks open:

1.  Notebook 1 - use for the 'demo' sections which utilize the IMDb
    data. Title this file "demo.ipynb" or "IMDb.ipynb".

2.  Notebook 2 - use for the 'practice on your own' sections which
    utilize the Lahman baseball data. Title this file "practice.ipynb"
    or "baseball.ipynb".

Last updated 2026-09-18 19:18:15 UTC
