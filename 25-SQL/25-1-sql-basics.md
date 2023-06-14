# Database

-  A database is a collection of data organized to be efficiently stored, accessed, and managed.

-  People will interchangeably use the term "database" to refer to database server software, the data records themselves or the physical machine that is holding the data.

-  We will be using `PostgreSQL` but there are many others, such as _MySQL_, _SQLite_, _MongoDB_, _Cassandra_, and more!

---

## Terminology:

-  Generall two types of databases:

   -  Relational Databses

   -  No-SQL databases
      -  Don't even use rows and columns.

-  Relational Databases:

   -  Model data as rows and columns of tabular data (like a spreadsheet)

   -  Key terms include:
      -  RDBMS - Relationa Database Management System
      -  schema - a logical representation of a database including its tables. (An expectation of what our data looks like)
      -  SQL - Structured Query Language, a human-readable language for querying
      -  table - the rows and columns
      -  column - a single attribute, like `id`, or `year`, or w/e
      -  row - a single record, with info in each attribute

---

## Visualizing Relational Databases:

### Example 1: A Books Table

| id  | title      | author  | page_count |
| --- | ---------- | ------- | ---------- |
| 1   | The Hobbit | Tolkien | 350        |

---

Relational databases is like a set of linked tables, say a `movies` table with a `users` table and a `ratings` table that takes info from each.

---

# Actually using `PostgreSQL`

## Managing databases:

Postgres will typically store many databases. Each 'database' is a collection of tables. Typically you have a database per application.

You can run Postgres from the command line: `psql`  
Also you can add the name of your databse: `psql fake_database` and we are connected to that particular database.

---

## Create Databases:

EAch project gets a separate database.

```
createdb my_database_name
```

Good database names are short and straightforward, and in **_lower_snake_case_**

This does NOT happen in Postgres, do it in the REGULAR TERMINAL SHELL

---

## Where is the database stored?

-  The database is **_NOT_** a file in your current directory
-  It's a _bunch_ of files/folders elsewhere on your computer
-  It's not even human-readable. It is optimized for speed!

---

## Seeding a database:

You can feed `.sql` scripts into the program psql:  
`psql < my_database.sql`  
This is often used to **_seed_** an empty database by building tables, filling in rows, or both

---

## Common Commands:

-  `\l` - List all databases
-  `\c DB_NAME` - connect to DB_NAME
-  `\dt` - List all tables in current db
-  `\d TABLE_NAME` - Get details about TABLE_NAME (in current db)
-  `\q` - Quit (also `ctrl-D`)

## Dropping Databases:

A database that is "dropped" is completely and utterly deleted.  
`dropdb DB_NAME`

## Backing up a database:

You can make a backup of your database by 'dumping it' to a file:  
`pg_dump -C -c -O DB_NAME > file_name.sql`

---

# SQL INTRO:

`SELECT * FROM people WHERE age > 21 AND id IS NOT NULL;`

-  SQL (Structured Query Language) is a human readable language for relational databases
-  Strings in SQL:
   -  case-sensitive: `Bob` is not the same as `bob`
   -  Surround strings with SINGLE quotes, not double quotes
-  Commands end with a semicolon
-  SQL keywords are conventionally written in ALL CAPS, but that is not required. It makes it easier to read.

---

SQL DML:

-  DML is a subset of SQL that involves querying and nmanipulating records in existing tables
-  Most of the DML you'll be doing will be related to CRUD operations on rows.

---

## **CRUD**

| Letter | Verb   | SQL Command       |
| ------ | ------ | ----------------- |
| **C**  | Create | `INSERT INTO`     |
| **R**  | Read   | `SELECT ... FROM` |
| **U**  | Update | `UPDATE ... SET`  |
| **D**  | Delete | `DELETE FROM`     |

---

# **SELECT**

**_SELECT_** is the most powerful and flexible command in SQL  
It selects rows (included summary data, roll-up data, etc)
from table(s)  
Select statements have subclauses, which are performed in this order:
|#|Clause|Purpose|Required?|
|---|---|---|---|
|1|FROM|Select and join together tables where data is|NO
|2|WHERE|Decide which rows to use|NO|
|3|GROUP BY| Place rows into groups|NO|
|4|SELECT|Determine values of result|`YES`|
|5|HAVING| Determine which grouped results to keep|NO|
|6|ORDER BY|Sort output data|NO|
|7|LIMIT| Limit output to _n_ rows|NO|
|8|OFFSET|Skip _n_ rows at start of output|NO|

---

## `FROM`

-  determine which table(s) to use to get data

```sql
SELECT * FROM books
```

You can get data from more than one table by "joining" them. More on that later.

---

## `WHERE`

-  Filter which rows are included.

```sql
SELECT *
   FROM books
   WHERE price > 10;
```

We hava a lot of operators and functions available.

---

## Aggregate Functions

-  evaluations that get passed into `SELECT` to return one output.

```sql
SELECT MIN(price), MAX(price) FROM books;

SELECT AVG(page_count) FROM books;

SELECT AVG(page_count) FROM books WHERE author = 'J. K. Rowling';
```

---

## `GROUP BY`

-  Reduce the amount of rows returned by grouping rows together

```sql
SELECT author, COUNT(*)
   FROM books
   GROUP BY author;
```

This gives us a list of all authors, with the count of each author's titles.

---

## `HAVING`

-  Subclause to `GROUP BY`
-  Decide which groups, if grouped, to keep

```sql
SELECT author, COUNT(*)
   FROM books
   GROUP BY author
   HAVING COUNT(*) >= 2;
```

---

# `ORDER BY`

-  Specify the order of results

```sql
SELECT id, author
   FROM books
   ORDER BY author;
```

-  Strings will be ordered alphabetically
-  You can add in more `ORDER BY`s to 'tiebreak'
-  You can do `desc` to do descending order or w/e

---

# `LIMIT`

-  Only show _n_ number of rows

```sql
SELECT title, author, price
   FROM books
   ORDER BY price
   LIMIT 5;
```

---

# `OFFSET`

-  Skip _n_ number of rows. Often used in conjunction with `LIMIT` to do pagination

```sql
SELECT id, author
   FROM books
   LIMIT 5
   OFFSET 5;
```

---

# SQL OPERATORS:

# `IN`

```sql
SELECT id, title
   FROM books
   WHERE id IN (1, 7,9);
```

Gives us the named ids.

---

# `BETWEEN`

```sql
SELECT id, title
   FROM books
   WHERE id BETWEEN 20 AND 25;
```

---

# `LIKE`

-  allows us to match things a bit more generally or pattern based.
   -  `%` replaces any number of characters, or no characters at all.
   -  `_` replaces one and exactly one character.

```sql
SELECT id, title FROM books WHERE title LIKE 'T%';
```

`LIKE` is case sensitive.  
In PotgreSQL, the keyword `ILIKE` is case INsensitive.

---

# `AS`

Allows us to define an `alias`. Relabel a column.  
We can reference this alias later on in the same query

---

# Creating Data with `INSERT`

```sql
-- Inserting a new book with title and author
INSERT INTO books (title, author)
   VALUES ('The Iliad', 'Homer');
```

```sql
-- Inserting several books with only title values:
INSERT INTO books (title) VALUES
   ('War and Peace'),
   ('Emma'),
   ('Treasure Island');
```

---

# `UPDATE`

```sql
-- Matt is a prolific writer
UPDATE books SET author = 'Matthew';
```

```sql
-- Or we just update one item
UPDATE books SET author = 'Jane Austen' WHERE title = 'Emma';
```

---

# `DELETE`

```sql
--delete Emma
DELETE FROM books WHERE title = 'Emma';

-- delete long books
DELETE FROM books WHERE page_count > 200;

-- delete all books!!
DELETE FROM books;
```

---
