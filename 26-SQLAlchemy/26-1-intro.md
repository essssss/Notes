# SQL ALCHEMY

We will be able to make `persistent` websites. And we will be able turn off the server or run from another computer.

`SQL Alchemy` will allow us to _connect_ Python and Postgres/SQL.

---

# GOALS

-   Learn to use object-oriented techniques with relational DBs
-   Without writing any SQL

```py
>>> whiskey = Pet(name='Whiskey', species="dog", hunger=50)

>>> whiskey.hunger
50

>>> whiskey.hunger = 20
```

---

## SQL Alchemy ORM

-   Popular, powerful Python-based ORM (object-relational mapping)
    -   We map our SQL information to a Python object.
-   Translation service between OOP in our server language and relational data in our database.
-   Can use by itself, with Flask, or other web frameworks.

---

## Installing SQL Alchemy:

-   Need the program that lets Python speak with Postgres specifically: `psycopg2`
-   Need the program that provides SQLAlchemy: `flask-sqlalchemy`
-   There's a bunch of specific setup code that we have to add. No need to memorize.

---

## Configuring SQL Alchemy

-   We usually move the database `stuff` into its own file

---

## SETUP

-   SQLALCHEMY_DATABASE_URI - Where is your database?
-   SQLALCHEMY_TRACK_MODIFICATIONS - Set this to False or SQLAlchemy will get mad
-   SQLALCHEMY_ECHO - Prints SQL statements to the terminal. Good for learning.

---

## Model

-   The point of a `Model` is to create a Python Class that we can interact with.

-   We can use SQL Alchemy to map a `table` and an `object` together.

    -   We can interact with our table as if it is a Python Object, and it will translate that into database and SQL structure.

-   Defining a Model can be a lot of code.

    -   We wind up defining the equivalent Table, which is translated into the `CREATE TABLE` structure.

```py
class Pet (db.Model):
	"""A Pet"""

	__tablename__ = "pets"

	id = db.Column(db.Integer,
				   primary_key=True,
				   autoincrement=True)

	name = db.Column(db.String(50),
					 nullable=False,
					 unique=True)

	species = db.Column(db.String(30), nullable=True)

	hunger = db.Column(db.Integer, nullable=False, default=20)
```

-   All models should subclass `db.Model`

-   The Class name is `singular` and the Table name is `plural`

-   Table name is specified with `__tablename__ = "..."`

        	- omg use the versions in the videos, because everything is a liiiiittle bit different now.

-   To create the databases from CLI:
    ```py
    with app.app_context():
      	db.create_all()
    ```

---

## Making Changes to the Model:

Say we want to add a new column or update something about a column.

-   The easiest option is to just drop the table and remake it in psql.

---

## Using our Model!!!

To add data to the database, is a multi-step thing.

1. We create an instance of the Class:
    - `stevie = Pet(name="Stevie Chicks", species="chicken", hunger=13)`
    - this doesn't add it to the database, but it DOES create the Object.
2. THEN, we add and commit.
    - `db.session.add(stevie)`
    - `db.session.commit()`
3. Make sure you use `with app.app_context():`
    - I find I have to do it all at once:
        ```py
        with app.app_context():
        	db.session.add(stevie)
        	db.session.commit()
        ```

To add multiple items, we can create an array of each parameters, and then iterate over a `zip`-created tuple.

---

## Errors and Rollback

If we have say a duplicate where we cannot, we will get an error. Python will not care, but when we try to add it into the DB it will raise a lil error.  
`db.session.rollback()` will _roll back_ a bad add or commit.

---

## Updating rows:

We won't have python objects representing a line automatically. But we will get to that later.

Now we can just access our objects and update a property, then add and commit:

```py
blue.name = "Blue 2"
db.session.add(blue)
db.session.commit()
```

---

---

---

# BREAKING IMPORTANT INFO

# PUSH THE APP CONTEXT WHEN YOU START IPYTHON

# ALL OUR PROBLEMS ARE SOLVED (lol not really but this helps)

# THIS CAN EVEN GO IN OUR app.py FILE

```py
from app import app, db
app.app_context().push()
```

---

## Better representation:

We are going to use a dunder method to change how things are represented:

```py
    def __repr__(self):
    	p = self
    	return f"<Pet id={p.id} name={p.name} species={p.species} hunger={p.hunger}"
```

---

## Querying Basics!

`Pet.query` gives us a funny mysterious object. It comes with a tooooon of methods.

`Pet.query.all()` - will run a query for 'all' and return a set of Python objects.

-   Select based on Primary Key:
    `Pet.query.get(id)` gets us the item by ID:

```py
Pet.query.get(1)
```

```SQL
SELECT * FROM pets WHERE id = 1
```

-   This is useful when we have a url that is formatted say `www.url.com/pet.32`. We will be able to run our Route and use that id to get the info from the database.

-   Also, we could use `filter_by()` to pass in a keyword-argument. then we can "send" it by adding `.all()` or `.first()` or some others.

```py
Pet.query.filter_by(species="cat").all()
```

---

## Filter By vs. Filter

-   `filter_by` is good when you want to filter by eactly one thing.
-   `filter` doesn't take keyword-arguments, but you have to explicitly reference the Class.attribute.
    -   `Pet.query.filter(Pet.species == 'cat').all()`
    -   `==` works a little different in this context. It doesn't give us True or False, sqlalchemy has given us a string to be converted into a query.
        -   Which makes it possible to use greater than or less than or w/e. Also, we can do multiple expressions separated by a comma, which is equivalent to `AND`.

---

-   `.get(pk)`
    -   Gets item by Primary Key
-   `.all()`
    -   Get all records as a list
-   `.first()`
    -   Get first record or **_None_**
-   `.one()`
    -   Get first record, error if 0 or if > 1
-   `one_or_none()`
    -   Get first record, error if > 1, **_None_** if 0

---

## Adding Methods to our models:

-   We've already added one to change our 'repr' string:

```py
def __repr__(self):
	p = self
	return f"<Pet id={p.id} name={p.name} species={p.species} hunger={p.hunger}>"
```

-   We could add, like a 'greeting' method:
    -   This is pretty self explanatory:

```py
def greet(self):
	return f"hi, I am {self.name} and I am a {self.species}."
```

-   We could add a `feed()` method which would change our pet's `hunger` attribute.

    -   We specify how much we feed in each go and it will subtract.

    ```py
    	def feed(self, amt=20):
    	"""Update hunger based off of amt"""
    	self.hunger -= amt
    	self.hunger = max(self.hunger, 0)
    ```

    -   This has not added or commited the changes to our database, so make sure to do that!!!

---

## Adding Class methods:

-   Use `@classmethod` decorator.
-   Call it on the Pet class, not an individual pet.
    -   eg: `Pet.get_by_species('dog')` will give us all our doggies.
    ```py
    @classmethod
    def get_by_species(cls, species):
    	return cls.query.filter_by(species=species).all()
    ```

---

## Listing and Showing our database using Flask

### Combining Flask routes with our ORM

-   With Flask-SQLAlchemy, all our useful methods are on `db`
    -   With vanilla SQLAlchemy, stuff is spread all over
    -   Also there are specific Web related features

---

```py
@app.route("/")
def list_pets():
    """List pets and show add form"""

    pets = Pet.query.all()
    return render_template("list.html", pets=pets)
```

```html
<ul>
    {% for pet in pets %}
    <li><a href="/{{ pet.id }}">{{ pet.name }}</a></li>
    {% endfor %}
</ul>
```

---

### Next we can add separate 'details' pages for each pet!

```py
@app.route("/<int:pet_id>")
def show_pet(pet_id):
    """Show details about a single pet"""
    pet = Pet.query.get_or_404(pet_id)
    return render_template("details.html", pet=pet)
```

(Pet.query.`get_or_404`(pet_id))

```html
<h1>{{pet.name}} Details</h1>
<ul>
    <li>ID: {{pet.id}}</li>
    <li>Name: {{pet.name}}</li>
    <li>Species: {{pet.species}}</li>
    <li>Hunger: {{pet.hunger}}</li>
</ul>
<p>{{pet.greet()}}</p>
```

---

## Creating a New Pet!

-   A form that will add a new pet.
-   We will use `methods=["POST"]` on the "/" route.

````py
@app.route("/", methods=["POST"])
def create_new_pet():
    name = request.form["name"]
    species = request.form["species"]
    hunger = request.form["hunger"]
    hunger = int(hunger) if hunger else None

    new_pet = Pet(name=name, species=species, hunger=hunger)
    db.session.add(new_pet)
    db.session.commit()

    return redirect(f"/{new_pet.id}")
    ```
````

---

## Deleting an item:

We are going to delete our friend "Carrot Face" who has starved to death ;-;

```py
Pet.query.filter_by(hunger=57).delete()
# this selects via our query, and stages the deletion

db.session.commit()
# this will actually delete it.
```

## Deleting multiples:

Same process:

```py
Pet.query.filter(Pet.species == 'pig').delete()
# Again, we select our animals (this time it's multiple animals), and then stage deletions

db.session.commit()
# And they're gone
```

SQL Alchemy will pull data from our `session`. If we delete something from the session without commiting it, SQL Alchemy will behave as if that thing is gone, unless we use `db.session.rollback`.

---

## Seeds files:

This is a file we can run to empty out and fill a database with sample data or w/e

```py
"""Seed file to make sample data for pets db."""

from models import Pet, db
from app import app

# Create all tables
db.drop_all()
db.create_all()

# If table isn't empty, empty it
Pet.query.delete()

# Add pets
whiskey = Pet(name='Whiskey', species='dog')
bowser = Pet(name='Bowser', species='dog', hunger=10)
spike = Pet(name='Spike', species='porcupine')

# Add new objects to our session, so they'll persist
db.session.add(whiskey)
db.session.add(bowser)
db.session.add(spike)

# Commit!
db.session.commit()
```

Wheeee!!!

---

## TESTING:

This is pretty important.

It is **`very`** important that we use a aseparate database for testing. eg `pet_shop_test`

We use `setUp(self)` and `tearDown(self)` for our tests to empty the databases.

Might be a nice idea to set up a `TestPet` or w/e in the setUp method. Check `test_flask.py` for that example
