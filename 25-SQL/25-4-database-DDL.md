# Database Definition Language

## Creating Tables:

```sql
CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title TEXT
    author TEXT,
    price FLOAT,
    page_count INTEGER,
    publisher TEXT,
    publication_date DATE
);
```

1. Keyword `CREATE TABLE`
2. Then the table name `books`
3. Then in parens we type the `column names` followed by the `data type` of that column.

We usually write this all in a separate file. not in command line.

---

## SQL Datatypes:

There are a **lot** of psql data types. These are some common ones we will use:

-  **Integer** - integer numbers
-  **Float** - Floating point numbers (you can specify the precision)
-  **Text** - Text Strings
-  **Varchar** - Text Strings but limited to a certain number of characters
-  **Boolean** - True or False
-  **Date** - Date (without time)
-  **Timestamp** - Date and time
-  **Serial** - Auto-incrementing numbers (used for primary keys)

## A small word about `NULL`:

`NULL` is a special value in SQL for 'unknown'.  
It is **not** the same as an empty string or 0!

We see 'null' when we leave a column blank.

We select this with the keyword `WHERE xxx IS NULL`

Null values are ok when you really might have missing or unkown data, but generally they are a pain so it can be a good idea to make fields 'not nullable'.

---

## Constraints

Constraints are a basic form of validation. The database can prevent basic types of unintended behavior.

-  `Primary Key` (every table must have a unique identifier)
-  `Unique` (Prevents duplicates in the column)
-  `Not Null` (Prevent 'null' in a column)
-  `Check` (do a logical condition before inserting/updating)
-  `Foreign Key` (Column values must reference values in another table)

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    phone_number TEXT UNIQUE,
    password TEXT NOT NULL,
    account_balance FLOAT CHECK (account_balance > 0)
);
```

## Default values

Just add `DEFAULT 1` or w/e value makes sense. Normally, default value will be 'NULL'.

---

## Primary and Foreign Keys:

-  Primary Key is designed to DESIGNATE a unique identifier column for that table.
-  Foreign Key is a reference to another table's Primary Key
   -  `REFERENCES other_table(column)`
   -  The column it references will default to one named 'id' if there is one.

---

## Deletion Behavior

When we have references, we need to handle our deletion behavior.  
Two options:

-  `ON DELETE SET NULL`

```SQL
CREATE TABLE movies (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    release_year INTEGER,
    runtime INTEGER,
    rating TEXT,
    studio_id INTEGER REFERENCES studios ON DELETE SET NULL
);
```

This will merely remove the user. Anything that references the deleted item will also become NULL  
ORRRR

-  `ON DELETE CASCADE`

```SQL
CREATE TABLE movies (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    release_year INTEGER,
    runtime INTEGER,
    rating TEXT,
    studio_id INTEGER REFERENCES studios ON DELETE CASCADE
);
```

This will `cascade` that deletion to any other item that references the deleted item.

---

## Many-to-Many DDL:

```SQL
CREATE TABLE roles (
    id SERIAL PRIMARY KEY,
    movie_id INTEGER REFERENCES movies ON DELETE CASCADE,
    actor_id INTEGER REFERENCES actors ON DELETE CASCADE
);
```

---

## Column Manipulation: `ALTER TABLE`

Adding/Removing/Renaming columns

```sql
ALTER TABLE books ADD COLUMN in_paperback BOOLEAN;

ALTER TABLE books DROP COLUMN in_paperback;

ALTER TABLE books RENAME COLUMN page_count TO num_pages;
```

**`¡CUIDADO!`  
`¡This will instantly kill data!`**

---

## STRUCTURING our databases

LEt's take a look at our `movies` database:

-  One `studio` has many `movies`

-  one `actor` has many `roles`

-  one `movie` has many `actors`

Make sure to VISUALIZE.

Specific diagramming tools: Crows foot notation.  
Some of them will directly output SQL code!  
`tableplus.com`

---

## Some Best Practices:

### Normalization

-  Normalization is a database design technique which organizes tables in a manner that reduces redundancy and dependency of data
-  It divides larger tables to smaller tables and links them using relationships.

### Normalization BAD example:

Consider the following products table, There are strings with multiole values in the `color` column making it hard to query:

| id  | color      | price |
| --- | ---------- | ----- |
| 1   | red, green | 05.00 |
| 2   | yellow     | 10.00 |

### Normalized Example:

`products`

| id  | price |
| --- | ----- |
| 1   | 05.00 |
| 2   | 10.00 |

`colors`

| id  | color  |
| --- | ------ |
| 1   | red    |
| 2   | green  |
| 3   | yellow |

`products_colors`

| id  | color_id | product_id |
| --- | -------- | ---------- |
| 1   | 1        | 1          |
| 2   | 2        | 1          |
| 3   | 3        | 2          |

In general it is best to add more tables!

---

## Indexing

-  A `database index` is a special data structure that efficiently stores column values to speed up row retrieval via `SELECT` and `WHERE` (ie 'read') queries.
-  For instance, if you place an index on a `username` column in a `users` table, any query using username will execute faster since fewer rows have to be scanned due to the efficient structure.

-  In general, database software (including postgreSQL) use tree-like data structures to store the data, which can retrieve values in logarithmic time O(lg(N)) instead of linear time O(N)
-  Translation: If I have 1,000,000 rows and am looking for a single column value, instead of examining every row, we can examine approximately log2(1000000) ~=~ 20 rows to get our answer, which is an incredible improvement!!

## How to create an Index!

Indexing is part of DDL but indeces can be created or dropped at any time. The more records there are at the time of indexing, the slower the indexing process will be.

```sql
CREATE INDEX index_name ON table_name (column_name);
```

Can also do a multi-column index, eg first and last name

```sql
CREATE INDEX index_name ON table_name (column1_name, column2_name);
```
