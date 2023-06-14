# Relationships in SQL

-  Learn what make SQL 'relational'
-  Understand one-to-many and many-to-many relationships
-  Describe and make use of different types of `joins` (inner, outer)

---

## Sample DB:

### Movies

| id  | title                        | studio                              |
| --- | ---------------------------- | ----------------------------------- |
| 1   | Star Wars: The Force Awakens | Walt Disney Studios Motion Pictures |
| 2   | Avatar                       | 20th Century Fox                    |
| 3   | Black Panther                | Walt Disney Studios Motion Pictures |
| 4   | Jurassic World               | Universal Pictures                  |
| 5   | Marvel's The Avengers        | Walt Disney Studios Motion Pictures |

-  This has a lot of duplication
-  What if we want more info about the studios? Do we write all that a bunch of times too?
-  Let's maybe make a new `table` for the studio information.

---

### `movies`

| id  | title                        | studio_id |
| --- | ---------------------------- | --------- |
| 1   | Star Wars: The Force Awakens | 1         |
| 2   | Avatar                       | 2         |
| 3   | Black Panther                | 1         |
| 4   | Jurassic World               | 3         |
| 5   | Marvel's The Avengers        | 1         |

### `studio`

| id  | name                                | founded_in |
| --- | ----------------------------------- | ---------- |
| 1   | Walt Disney Studios Motion Pictures | 1953-06-23 |
| 2   | 20th Century Fox                    | 1935-05-31 |
| 3   | Universal Pictures                  | 1912-04-30 |

-  Now we can have SO MUCH info about each studio!
-  But this means we will probably need to gather and combine info from multiple tables

---

# One-to-Many (1:M)

Each studio corresponds to many different movies.

## PRIMARY KEY - A unique identifier to each row. Typically called `id`

-  Our `studio_id` in the **_movies_** table is a reference to the `id` in the **_studio_** table
   -  This is called a `foreign key`
-  Typically this is implemented with a `foreign key constraint` which makes sure every `studio_id` exists somewhere in the `studios` table.
   -  SQL will be able to verify this.
-  One-to-Many in the sense that **one** studio has **many** movies, but each movie still only has one studio.
-  In this example we can say that `movies` is the **referencing** table, and `studios` is the **refereneced** table.

---

# The Foreign Key Constraint

Setting up a foreign key constraint with DDL:

```sql
CREATE TABLE studios
  (id SERIAL PRIMARY KEY,
  name TEXT,
  founded_in TEXT);

CREATE TABLE movies
  (id SERIAL PRIMARY KEY,
  title TEXT,
  studio_id INTEGER REFERENCES studios (id));
```

Constraints are specified by the DDL, but affect DML query behavior:

```SQL
INSERT INTO studios (name, founded_in) VALUES
    ('Walt Disney Studios Motion Pictures', '1953-06-23'),
    ('20th Century Fox', '1935-05-31'),
    ('Universal Pictures', '1912-04-30');

-- Reference Disney's primary key
INSERT INTO movies (title, studio_id)
    VALUES ('Star Wars: The Force Awakens', 1);
```

SQL will **_not_** let us delete a `studio` while `movies` still reference it.  
To get around this, we can do a few things:

-  Option 1: Clear out the `studio_id` columns of movies that reference it

   ```sql
   UPDATE movies SET studio_id = NULL
   WHERE studio_id=1;

   DELETE FROM studios WHERE id=1;
   ```

-  Option 2: Delete all the `movies` associated with that studio first:

   ```SQL
   DELETE FROM movies WHERE studio_id = 1;
   DELETE FROM studios WHERE id=1;

   ```

---

# JOIN TABLES: `INNER JOINS`

OK SO, Say we want to select all movies from a given studio, and we have the studio name. `'Walt Disney Studios Motion Pictures'`  
We start with:

```sql
SELECT id FROM studios WHERE name = 'Walt Disney Studios Motion Pictures';
```

Where we recieve our `id` of `1`  
And then we have to select using that id:

```sql
SELECT * FROM movies WHERE studio_id = 1;
```

There is a better way:

## `JOIN` operation

-  The `JOIN` operation allows us to creat a table _in memory_ by combining information from different tables.
-  Data from tables is matched according to a `join condition`
-  Most commonly, the join condition involves comparing a `foreign key` from one table and the `primary key` in another.

```sql
SELECT title, name
FROM movies
JOIN studios
ON movies.studio_id = studios.id;
```

In order to see duplicate column names:

```sql
SELECT movies.id, studios.id
FROM movies
JOIN studios
ON movies.studio_id = studios.id;
```

`JOIN` and `INNER JOIN` are the same, the `INNER` keyword is optional.

---

# `OUTER JOIN`

-  Outer joins can be `left`, `right`, or `full`

   -  `LEFT` - All of the rows from the first table (left), combined with matching rows from the second table (right)
   -  `RIGHT` - The matching rows from the first table (left), combined with all of the rows in the second table (right)
   -  `FULL` - All rows from both tables

-  Let's add an independent `movie` without a studio:

```sql
INSERT INTO movies(title, release_year, runtime, rating)
VALUES
    ('My First Indie Movie', 2015, 90, 'PG-13'),
    ('My Second Indie Movie', 2020, 110, 'R');
```

-  Now we can add some new studio data.

```sql
INSERT INTO studios (name, founded_in) VALUES ('Chickens Pictures', '2020-12-12');

INSERT INTO studios (name, founded_in) VALUES ('Cat Cat Cat Pictures', '1980-10-11');
```

-  The `INNER JOIN` will not give us the info that is not matched up on both tables, ie. the new stuff. We must use one of the `OUTER JOIN` methods

```sql
SELECT name, COUNT(*)
FROM movies
JOIN studios
ON movies.studio_id = studios.id
GROUP BY studios.name;
```

---

# Many-to-Many (M:M)

-  We've seen an example of a **one-to-many** relationship: One studio has many movies, and one movie belongs to one studio.
-  But not every relationship can be expressed in this way
-  Consider actors: one movie has many different actors, and also each actor has roles in many different movies!
-  This is a `many-to-many` relationship
-  Which is really just two `one-to-many`s back to back.

---

## A `Many-to-Many` relationship will be represented by a separate table to join the separate refences.

---

Look at our actors and roles tables:

`actors`
Column | Type | Collation | Nullable | Default  
------------|---------|-----------|----------|---------------
id | integer | | not null | nextval('actors_id_seq'::regclass)
first_name | text | | not null |
last_name | text | | |
birth_date | date | | not null |

`roles`
Column | Type | Collation | Nullable | Default  
----------|---------|-----------|----------|-----------------------------------
id | integer | | not null | nextval('roles_id_seq'::regclass)
movie_id | integer | | |
actor_id | integer | | |

-  The `roles` table simply exists as a link between actors and movies

```sql
-- We've already created the movies table
CREATE TABLE actors
    (id SERIAL PRIMARY KEY,
    first_name TEXT,
    last_name TEXT,
    birth_date TEXT);

CREATE TABLE roles
    (id SERIAL PRIMARY KEY,
    movie_id INTEGER REFERENCES movie (id),
    actor_id INTEGER REFERENCES actors (id));
```

-  We could also have a `character` column in our roles table.

---

This is pretty difficult to do without a visual representation of the tables.

-  Instead we can do this using python code.
-  Using something called an ORM

---

Deleting:  
`DELETE FROM studios WHERE id = 4;` Is probelmatic, we can't orphan foreign ids.  
HOWEVER, We have a settiing called `ON DELETE CASCADE`  
This means we can delete an actor, and it will delete the corresponding roles.

---

# `JOIN`s on a `MANY-TO-MANY`

## Join Tables:

-  The `roles` table in our current schema is an example of a **_join table_** (AKA an associativr table AKA a mapping table).
-  A join table serves as a way to connect two tables in a many-to-many relationship
-  The join table consists of, at a minimum, two foreign key columns to the two other tables in the relationship.
-  It is completely valid to put other data in the join table (eg. How much was an actor paid for the role, or character name)
-  Sometimes the `JOIN` table has a nice name (when it has meaning on its own, eg. `roles`, but you can also call it `table1_table2`)

---

## Joining the `actors` with `roles`:

```sql
SELECT *
FROM roles JOIN actors
ON roles.actor_id = actors.id;
```

## But really, we want to JOIN the `actors` and `movies`, via the `roles` tables.

```sql
SELECT *
FROM roles JOIN actors
ON roles.actor_id = actors.id
JOIN movies
ON roles.movie_id = movies.id;
```

---

We do not need to disply the `roles` table, but we use it as the 'heart' of our `JOIN` operation.

```sql
SELECT title, first_name, last_name
FROM roles JOIN actors
ON roles.actor_id = actors.id
JOIN movies
ON roles.movie_id = movies.id;
```

We do have an alias shorthand to select certain columns.

```sql
SELECT m.title, a.first_name, a.last_name
FROM movies m
JOIN roles r
ON m.id = r.movie_id
JOIN actors a
ON r.actor_id = a.id;
```

When we call the table name, we can also add in that lil alias.

We can also add a `WHERE` query or w/e

```sql
SELECT m.title, m.release_year, a.first_name, a.last_name
FROM movies m
JOIN roles r
ON m.id=r.movie_id
JOIN actors a
ON a.id=r.actor_id
WHERE m.release_year > 2016
ORDER BY m.release_year;
```

---

# `MANY-TO-MANY` Outer Joins

```sql
SELECT *
FROM roles r
RIGHT JOIN movies m
ON r.movie_id = m.id;
```

This will show us all the movies, even those without roles associated.

---

```sql
SELECT *
FROM roles r
RIGHT JOIN movies m
ON r.movie_id = m.id
RIGHT JOIN actors a
ON r.actor_id = a.id;
```

Start writing queries in a text file and copy paste them!
