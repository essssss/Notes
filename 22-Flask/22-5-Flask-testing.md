# Python testing!

-  Discuss the benefits of writing tests
-  compare unit, integration, and end-to-end tests
-  compare different ways to write tests in Python:
   -  `assert` statements
   -  doctests
   -  `unittest` module
-  Write integration tests for our Flask apps

---

# Can't I Just Test My Code Myself?

# NO

---

# Testing is

-  The #1 thing employers ask about

a lot of bootcamps and beginners are lacking in testing

it is a highly skilled art.

---

# Automated tests are particularly good for:

-  Testing things that "should work"
-  Testing edge cases
-  Making sure NEW things don't break OLD things ('regression')

---

# Kinds of Tests:

-  Unit Test: Does this individual component work?

-  Integration Tests: Do the parts work together?

-  End-to-End Test: Do wet clothes turn into dry clothes?

Usually we end up writing a bunch of Unit Tests, a fair few Integration Tests, and then just a few End-to-End Tests

---

# Unit Tests

-  Test one "unit" of functionality
   -  Typically one function or method
-  Don't test integration of components
   -  Don't test a Framework itself (eg Flask)
-  Promotes modular code
   -  Write code with the testing in mind

### Assert statements:

```py
def adder(x, y):
    """Add two numbers"""

    return x + y

assert adder(1, 1) == 2, '1 + 1 is not 2'
```

The thing about assertions is that they stop the code as soon as it fails. Which is bad because when we're testing we want to continue through all tests at once. Also, assertions don't give too much feedback

running Python in 'optimized mode' will ignore `assert`s

---

# DocTests

-  Testable documentation

DocTests are written as if they are the exact input/output from the python terminal (including `>>>` and the line breaks and everything)

```py
def adder(x, y):
    """
    Adds two numbers together.
    >>> adder(4,6)
    10

    >>> adder(-1,50)
    49
    """

    return x + y
```

and then we **run** the doctest module:  
`python -m doctest arithmetic.py`

-  if things don't fail, we will not see anything.

Add `-v` for verbose

### Doctests are not great for testing a full application

They are better for just simple examples to show how a function is supposed to work.

---

# Python unittest module

Unit testing is done via classes. In the Python standard library.

```py
import arithmetic
from unittest import TestCase

class AdderTestCase(TestCase):
    """Examples of unit tests."""

    def test_adder(self):
        assert arithemetic.adder(2, 3) == 5

    def test_adder_2(self):
        # instead of assert arithemetic.adder(2, 2) == 4
        self.assertEqual(arithmetic.adder(2, 2), 4)
```

TestCase is like a base class that has a bunch of testing methods

Test cases are a bundle of tests

-  In a class that subclasses `TestCase`
-  Test methods **_MUST_** start with `test_`
-  `python -m unittest NAME_OF_FILE`

Check the docs for all the different test methods.

-  `assertEqual()` or w/e

The test methods give better feedback than any plain assertion. So use them!

---

VSCode can run our unit tests!

---

# Integration tests

-  Test that components work together

   -  Does this URL path map to a route function?
   -  Does this route return the right HTML?
   -  Does this route return the correct status code?
   -  After a POST route, are we redirected?
   -  After this route, does the session contain the expected info?

-  Use the `unittest` framework to test the integration tests!

---

# `test_client`

```py
from app import app
```

That's just app.py file.

```py
class ColorViewsTestCase(TestCase):
    """Examples of integration tests: testing Flask app"""

    def test_color_form(self):
        with app.test_client() as client
        # can now make requests to flask as 'client'
```

The `test_client` allows us to run our tests w/o doing a whole server thing.

```py
def test_color_form(self):
    with app.test_client as client:
        # cna now make requests to flask via "client"
        resp = client.get('/')
        html = resp.get_data(as_text=True)

        self.assertEqual(resp.status_code, 200)
        self.assertIn('<h1>Color Form</h1>', html)
```

We can check if the resultant status code is indeed 200. Also that the right `h1` is in the html file

---

# POST request tests

Very similar. Just say, `resp.post('/route')`

---

# Redirects

-  How about we check for the redirect status code `302`?
-  Also, we can have flask **follow** the redirect so we can check on that as well.

---

# Session testing

```py
from flask import session


def test_session_info(self):
    with app.test_client() as client:
        resp = client.get("/")

        self.assertEqual(resp.status_code, 200)
        self.assertEqual(session['count'], 1)
```

## To set the session before the test:

-  `client.session_transaction`

```py
def test_session_info_set(self):
    with app.test_client() as client:
        # Any changes to the session should go in here:
        with client.session_transaction() as change_session:
            change_session['count'] = 999

        # Now those changes will be in Flask's 'session
        resp = client.get('/')

        self.assertEqual(resp.status_code, 200)
        self.assertEqual(session['count'], 1000)
```

---

# `setUp` and `tearDown`

-  This is stuff we will run before/after every single test method.
   -  this will run `setUp` before the test and then `tearDown` after the test. individually, not as a group
-  If we want to run ONE setup we do it as a class method:

```py
    @classmethod
    def setUpClass(cls):
        print("INSIDE SETUP CLASS")

    @classmethod
    def tearDownClass(cls):
        print("INSIDE TEAR DOWN CLASS")
```

---

# Simple lil config things:

-  We can have Flask errors become real Python errors:
   -  `app.config["TESTING"] = True`
-  Also, flask debug toolbar.
   -  The flask debug toolbar will manipulate the HTML which we can't have if we are testing the HTML response!
   -  debug toolbar will intercept redirects which will mess with the status code.
   -  `app.config['DEBUG_TB_HOSTS'] = ['dont-show-debug-toolbar']`

---

# Testable code:

example- how do we test this?

```py
@app.route('/taxes', methods=['POST'])
def taxes():
    """calculate taxes from web form"""

    income = request.form.get('income')

    # Calculate the taxes owed
    owed = income / 45.3 * random.randint(100) / other_stuff

    return render_template("taxes.html", owed=owed)
```

-  If we wanted to test this, we'd have to hard code a scenario or w/e:

```py
    def test_taxes(self):
        with app.test_client() as client:
            response = client.post('/taxes', data={'income' : '1000'})
            html = response.get_data(as_text=True)

            self.assertEqual(response.status_code, 200)
            self.assertIn('<h3>You owe $150</h3>', html)


            response = client.post('/taxes', data={'income' : '0'})
            html = response.get_data(as_text=True)

            self.assertEqual(response.status_code, 200)
            self.assertIn('<h3>You owe $0</h3>', html)

            response = client.post('/taxes', data={'income' : '1000000'})
            html = response.get_data(as_text=True)

            self.assertEqual(response.status_code, 200)
            self.assertIn('<h3>You owe $12034</h3>', html)
```

---

## INSTEAD

It is way better to move the calculate logic into its own function

```py
def calculate_taxes(income):
    """Do Maths"""
    ...

@app.route('/taxes', methods=['POST'])
def taxes():
    """calculate taxes from web form"""

    income = request.form.get('income')
    owed = calculate_taxes(income)

    return render_template("taxes.html", owed=owed)
```

-  That way we can test the calculations on their own!

---

-  Ask yourself if there is too much logic in your View function
-  When you test, you don't need just one assertion per function
-  Remember to test failing things, like forms that don't validate

---

# Organizing/Running tests

### Small Projects:

-  keep tests in one file.
-  run them from the terminal

### Larger Projects:

-  Organize in files named **_tests_something.py_**
-  Run them all from the terminal like this
   -  `python -m unittest`
-  Or individual test files/cases/methods
   -  `python -m unittest tests_cats`
   -  `python -m unittest tests_cats.CatViewTestCase`
   -  `python -m unittest tests_cats.CatViewTestCase.test_meow`
