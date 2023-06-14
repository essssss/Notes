# Flask

-  We are going to learn web development with python. Back End!

-  Flask is a `framework`

---

## Goals

-  Describe the purpose and responsibilities of a web framework
-  Buld a small web applications using Python and Flask
-  Set environmental variables for local Flask development
-  Handle GET and POST requests with Flask
-  Extract data from different parts of the URL with Flask

---

-  A Web server is a program that is running on a machine and waiting for a web request.

---

## Flask is a Web Framework

-  A set of functions, classes, etc. that help you define

   -  **Which** requests to respond to
      -  **_http://server/about-us_**
      -  **_http://server/post/1_**
   -  **How** to respond to requests
      -  Shows an "About Us" page
      -  Shows the first blog post

-  Like a library, but bigger and more opinionated
-  Usage is similar to the Python Standard Library
   ```py
   from flask import Flask, request
   ```

---

## What do Web Apps need to do?

-  handle web requests
-  produce dynamic HTML
-  handle forms
-  handle cookies
-  connect to databases
-  provide user log-in/log-out
-  cache pages for performance
-  WOW!!! SO MUCH

### Django will do a lot of stuff behind the scenes

### We will start with learning Flask because it is more hands-on

---

## Flask Apps

-  installing flask:
   ```py
   pip install flask
   ```
-  Create an `app.py` file

   -  this must be the name

   ```py
   from flask import Flask

   app = Flask(__name__)
   # ^^We must start with this code^^
   ```

-  Run file in terminal, using `flask run`

-  or we CAN in fact change the name of app.py, but we need to tell Flask we're doing that.
   ```
   FLASK_APP=my_app.py flask run
   ```

---

# Development mode vs. Production

-  to change mode, when we run the app.py,
   ```
   flask run --debug
   ```
-  We can set debug in the `venv`

---

# Adding Routes

-  This is like the fundamental code for Flask.
   -  It controls the routing, where we listen for a request and then we send it around
-  A function that returns a web response is called a **view**
   -  response is a **string**
   -  Usually, a **string** of HTML
-  So, our function retunrs an HTML string:

   ```py
   @app.route('/hello')
   #Look! A Decorator! We need a function RIGHT AFTER IT
   def say_hello():
       """REturn a simple "Hello" Greeting. """

       html = "<html><body><h1>HELLO</h1></body></html>"
       return html
   ```

-  FLASK has constructed our entire response! Wow!!

---

# Query strings and parameters and the like

## Requests:

Flask provides an object, **_request_**, to represent web requests. This object will parse the url and create a dictionary like thing called "requests" and fill it with the key-value pairs of the url request

```py
from flask import request
```

## Handling Query strings:

For a URL like `/search?term=fun`

```py
@app.route("/search")
def search():
   """Handle GET requests like /search?term=fun"""

   term = request.args["term"]
   return f"<h1>Searching for {term}</h1>"
```

<br>
<br>
<br>
<br>
<br>
<br>

---

# Handling POST Requests!

-  By default, a route only responds to GET requests. To accept POST requests, we must specifiy:

```py
@app.route("/my/route", methods=["POST"])
def handle_post_to_my_route():
   ...
```

-  we most commonly make POST requests through a form, or through AJAX, or through CURL
-  It is simple and easy to make a GET and POST request for the same URL that will have different functions

<br>
<br>
<br>
<br>
Example POST Request:

```py
@app.route("/add-comment")
def add_comment_form():
   """Show a form for adding a comment"""

   return """
      <form method="POST">
         <input name="comment">

         <button>Submit</button>
      </form>
      """
# 'name = "comment"' will stash the input in a key-value pair of "comment" = "whatever the input is"


###############################


@app.route("/add-comment", methods=["POST"])
def add_comment():
   """Handle adding comment"""

   comment = request.form["comment"]
   #TODO SAVE INTO DATABASE
   return f"<h1> RECEIVED '{comment}'.</h1>"
```

---

# Variables in a URL

-  If we want a user page for each individual user we do not want to add each possible user as a separate route!
   -  We can define a pattern with a variable. we use `<pointy carrot brackets>`

```py
USERS = {
   "whiskey": "Whiskey the Dog",
   "spike": "Spike The Hedgey",
}

@app.route('user/<username>')
def show_user_profile(username)
   # We must pass in the variable to this function!!!
   """Show user profile for user"""

   name = USERS[username]
   return f"<h1>Profile for {name}"
```

This variable defaults to a string, so sometimes we need to specify that it's something els, eg. an `integer`:

```py
@app.route('/post/<int:post_id>)
```

---

## We can have more than one variable in the URL!

```py
@app.route("/products/<category>/<int:product_id>")
def product_detail(category, product_id):
   """Show the product details"""
   ...
```

---

# Query Parameters vs URL Parameters:

`http://toys.com/shop/spinning-top?color=red`

```py
@app.route("/shop/<toy>")
def toy_detail(toy):
   """Show detail about a toy"""

   # Get color from req.args, falling back to None
   color = request.args.get("color")

   return f"""<h1>{toy}</h1>
            <p>Color: {color}</p>"""
```
---
| URL Parameter | Query Parameter |
| ------------- | --------------- |
|`/shop/<toy>`|`/shop?toy=elmo`|
|Feels more like the "subject" of the page| Feels more like "extra info about the page"|
| | Often used when coming from a form|

---

