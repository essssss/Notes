# Flask JINJA

We are going to stop writing our HTML in striing form!

GOALS

-  Explain what HTML templates are, and why they are useful
-  Use Jinja to create HTML templates for our Flask apps
-  debug our Flask apps using Flask Debug Toolbar

---

Templates:

-  Produce HTML
-  Allow responses to be dynamically created
   -  Can use variables passed from your `views`
   -  For loops, if/else statements
-  Can inherit from other templates to minimize repetition

---

Jinja is a popular template system for use with python, comes with Flask

---

### Templates Directory

```py
my-project-directory/
    venv/
    app.py
    templates/
        hello.html
```

---

# RENDERING A TEMPLATE:

```py
@app.route('/')
def index():
    """Return homepage"""

    return render_template("hello.html")
```

---

# Flask Debug Toolbar

First we must install with `pip3 install flask_debugtoolbar`

Add the following to your Flask app.py

```py
from flask import Flask, request, render_template
from flask_debugtoolbar import DebugToolbarExtension

app = Flask(__name__)

# the toolbar is only enabled in debug mode:
app.debug = True

# set a 'SECRET_KEY' to enable the Flask session cookies
app.config['SECRET_KEY'] = '<replace with a secret key>'

toolbar = DebugToolbarExtension(app)
... # Rest of app
```

---

# Dynamic Templates

-  the good stuff

   -  variables n stuff

-  Jinja will replace `{{double curly braces}}` with an evaluation via python
   -  pass variables from our `view` function.

---

### templates/lucky.html

```html
<h1>Hi!</h1>

<p>Lucky number: {{ lucky_num }}</p>
```

### app.py

```py
@app.route("/lucky")
def show_lucky_num():
    "Example of a simple dynamic template"

    num = randint(1, 100)
    return render_template("lucky.html", lucky_num=num)
```

-  Our variable is added as a keyword argument!

---

Or if we want to send a form and have it generate a new page:

```html
<!DOCTYPE html>
<html lang="en">
   <head>
      <meta charset="UTF-8" />
      <meta http-equiv="X-UA-Compatible" content="IE=edge" />
      <meta name="viewport" content="width=device-width, initial-scale=1.0" />
      <title>Greeter Form</title>
   </head>
   <body>
      <h1>Greeter Form</h1>
      <form action="/greet">
         <input type="text" placeholder="your name" name="username" />
         <button>Submit</button>
      </form>
   </body>
</html>
```

NOTE: the form `action` is set to the `greet` page, it sends the data as a query string

---

# Jinja conditionals!

-  ie. you get a different page if you are logged in or not, or if you have upvoted a thing, or if you have messages, etc.

Let's go back to our `/lucky` route:

```py
@app.route('/lucky')
def lucky_number():
    num = randint(1, 10)
    return render_template('lucky.html', lucky_num = num, msg="you are lucky!")
```

In our TEMPLATE `lucky.html` file, we want the logic:

```
{% if lucky_num == 3 %}
...
{% else %}
...
{% endif %}
```

---

# Looping!

-  We won't hard-code each comment, lol. We create a comment template and repeat it!

```
{% for VAR in ITERABLE %}
...
{% endfor %}
```

New route:

-  `/spell/something`
-  loop over the word and create an `<h1>` for each letter

```
{% for char in word %}
<h1>{{char}}</h1>
{% endfor %}
```

-  just a note, it's still best to minimize code inside ur template.

---

# A better greeter!

-  now we will add to our form to give a checkmark if we want 3 randomized compliments
   -  none of the compliments show up if the box isn't checked.

---

# Template inheritance:

-  Usually, different pages on a site are mostly the same!

Therefore:

-  make a `base.html` that will hold the repetitve stuff
-  "Extend that base template in your other pages
-  Substitute blocks in your extended pages

`{% block BLOCKNAME %} ... {% endblock %}`

```html
{% extends 'base.html' %} {% block title %}My awesome page title{% endblock %}
{% block content %}
<h2>I'm a header!</h2>
<p>I'm a paragraph!</p>
{% endblock %}
```

---

# Where do other files live

-  Do i need routes for style sheets? JS files?
   -  nah, include it in a folder called `static`
