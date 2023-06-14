# Goals for this lesson

-  Define what it means for HTTP to be "stateless"
-  Compare different strategies for persisting state across requests
-  Explain what a cookie is, and how client-server cookie communication works
-  Compare cookies and sessions
-  implement session functionality with Flask

---

# HTTP - STATELESS

-  HTTP remembers nothing between requests. Every request is regarded by the server completely seperately

-  BUT sometimes we want to remember things, eg: login status, banner dismissal, shopping cart, etc.

---

# Some ways to save state:

-  Have persistent query strings. lol
-  Bake it into the URL. ... no
-  Use JS Local Storage?

   -  sure, but only JS can use this data. No server side.

-  Let's use Cookies and Sessions!!

---

# Cookies

Cookies save "States"

-  Cookies are a way to store small bits of info on the client (browser).
-  Flask's `session` is powered by cookies

---

# What is a cookie?

-  Cookies are name/string-value pairs stored by the **client** (browser)
-  The server instructs the browser to store thes
-  the client sends cookies to the server with each request

---

## Simple cookies we could store

| Site            | Cookie Name   | Value        |
| --------------- | ------------- | ------------ |
| rithmschool.com | number_visits | "10"         |
| rithmschool.com | customer_type | "Enterprise" |
| localhost:5000  | favorite_food | "taco"       |

---

# Cookies can be a converstation:

-  `Browser`: I'd like to get the resource **/upcoming-events**
-  `Server`: Here's that HTML. Also, remember: **_favorite_food_** is **_taco_**
-  `Browser`: (stores this data on the computer)
-  `Browser`: etc etc

---

# Seeing Cookies in our browser

-  do it in Dev Tools

---

---

---

# Reading Cookies in Flask

```py
@app.route('/later-cookie')
def later():
    """An example page that can use a cookie"""

    fav_color = request.cookies.get('fav_color', '<unset>')

    return render_template("later-cookie.html", fav_color=fav_color)
```

# !!!!!SPECIAL FLASK DECORATOR!!!!

`@app.before_request` will run before any view functions. Fun!

---

To read cookies from Flask, simply use `request.cookies` which we don't even need to import outside of importing `request`

---

# Setting cookies:

(flask will give us a nicer way to do this later on)

-  Doing this will require us to delve into the entire Response Object. (REMEMBER, we normally have Flask build the whole Response Object when we `render_template(...)`. The Object itself contains all the other bs like the response code 200, and headers and footers and arms and legs)

```py
@app.router('handle-form-cookie')
def handle_form():
    """Return form response; iinclude cookie for our browser."""

    fav_color = request.args['fav_color']

    # Get HTML to send back. Normally, we just return this, but here
    # we need to do in pieces, so we can add the cookie first:
    html = render_template("response-cookie.html", fav_color=fav_color)

    # In order to set a cookie from Flask we need to deal
    # with the response more directly than usual
    # First, we make a response object from that HTML
    resp = make_response(html)

    # Let's add the cookie to the response (apparently there are
    # many ways to do this, see the Flask docs for how to set
    # cookie expiration, domain, it should apply to, or path)
    resp.set_cookie('fav_color', fav_color)

    return resp
```

# Cookie options:

-  **Expiration**: How long should browser remember this?
   -  Can be set to a time; default is "as long as web browser is running" (session cookie)
-  **Domain**: which domains should this cookie be sent to?
   -  Send only to `books.site.com` or everything at `site.com`?
-  **HttpOnly**: HTTP-only cookies aren't accessible via any kind of JavaScript
   -  Useful for cookies that contain server-side information and don't need to be available to JavaScript

---

# SESSION STORAGE:

NOT SESSIONS IN FLASK

THIS IS DUMB CONFUSING

LOLOL

---

# Comparisons of Types of Browser Storage:

-  LocalStorage
   -  Stores data with no expiration date, and gets cleared only through Java Script, or clearing the browser cache
   -  Domain Specific
   -  Storage limit is much larger than that of a cookie
-  SessionStorage
   -  Stores data only while a browser window or tab is open. When it is closed it is gone
   -  Again, more space than in a cookie
-  Cookie
   -  Cookies can be made secure by setting the HttpOnly flag as true. This prevents client side access to that cookie
   -  Sent from the browser to the server for every request that is made
   -  Because of the transmission frequency, there is a size limit.
   -  Expiration date is customizable.

---

---

---

# Flask Sessions:

-  Why better than cookies?
   -  Cookies can be tricky:
      -  They are just strings
      -  Limited by size so there's a limit in info
      -  Pretty low-level. Gotta dig into the code
-  Sessions
   -  Sessions are like a 'magic dictionary' that we can use right from Flask
   -  No extra code
   -  More secure

---

# Sessions

-  Contain info for the current browser
-  Preserve type (lists stay lists, etc)
-  Are "signed", so users can't modify data

---

# Using Session in Flask

-  import `session` from `flask`
-  set a `secret_key`

```python
from flask import Flask, session

app = Flask(__name__)
app.config["SECRET_KEY"] = "shh secret"
```

now in routes we can treat `session` as a dictionary:

```py
@app.route("/some-route")
def some_route():
   """Set fav_number in session"""

   session['fav_number'] = 42
   return "Ok, I put that in the session"
```

---

To get things 'out' of a session, treat it like a dictionary:

```py
from flask import session

@app.route("/my-route")
def my_route():
   """return info using stored fav_number"""

   return f"favorite number is {session['fav_number']}"
```

It will stay the same type

In jinja templates we don't even need to pass in the variables. we just do `{{ session['fav_number']}}`

---

But how does it work?

-  different frameworks do it differently
-  In Flask, the sessions are stored in the browser as a cookie
   -  `session = 'asdkjhfASDFHAsdkjhf,jabsdf.asdkhfgjdb"`
   -  They're "serialized" and "signed"
   -  Users can see, but can't change their session data -- only Flask can change it
-  Server-side sessions
   -  Some web frameworks store session data on the server instead
   -  the cookie is just some sort of id, to associate the user with the data
   -  Often stored in a relational database
   -  useful with lots of session data or for complex setups
   -  Flask supports this with the add-on `flask-session`

---

# Cookies or Sessions?

-  easier for a developer to use a session
-  Also more secure
-  Still know about cookies
