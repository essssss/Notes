# More Flask Tools!

-  Other common features:
   -  redirecting
   -  flash messaging
   -  returning JSON data
-  Debug Flask errors
-  Set break points in python codewith `pdb`

---

# Redirecting

-  Very common in large web applications.

   -  eg: redirecting a user after a password success or fail. Or maybe redirecting from an old, unused endpoint

-  What is an HTTP redirect?
   -  An HTTP response
   -  Status code is a "redirect code" (often `302`)
   -  response also contains a URL for browser to re-request
-  eg in instagram if you try to load up the account edit page when you are not logged in, you will be redirected to the login page

---

# Flask redirects

```py
# First we must import 'redirect' from Flask
from flask import Flask, request, render_template, redirect

# Then we return the 'redirect' function and insert the new page
@app.route('/old-home-page')
def redirect_to_home():
    """Returns redirect to new home page"""
    return redirect("/")
```

---

# Common Pattern: redirect POST to GET

-  POST requests are often from a form
   -  and change data on the server
-  If you return HTML from a POST request, the browser will show it just fine.
   -  But if the user hits "REFRESH" they will get a weird "OK to resubmit?" dialogue.
-  Better workflow:
   -  Do what you want inside the POST route
   -  then _redirect_ to a page that shows the confirmation

```py
@app.route('/movies')
def show_all_movies():
    """Show list of all movies in DB"""
    return render_template('movies.html', movies=MOVIES)

@app.route('/movies/new', methods =["POST"])
def add_movie():
    title = request.form['title']
    # Add to Pretend DB
    MOVIES.append(title)
    return redirect("/movies")
```

Our form will take us to `/movies/new` but then simultaneously POSTS and REDIRECTS us back to `/movies` which means we can refresh the page without sending another POST request  
We could also redirect to something like a confirmation page or something.

---

# Flash Messaging/Message Flashing

Often you want to provide feedback at the "next page the user sees"

-  Very common on a redirect

```py
from flask import flash

@app.route("/your/route")
def your_route():
    """Some route that redirects"""

    flash("Message for user!!!")
    return redirect("/somewhere/else")
```

Template used by `/somewhere/else`:

```py
{% for msg in get_flashed_messages() %}
    <p>{{msg}}</p>
{% endfor %}
```

JK
We typically move the message to the main template.

ALSO  
We can categorize our messages!

Just add the category string after the message eg:  
`flash("Movie Added!", "success")`  
and then you add the param  
`get_flashed_messages(with_categories-true)` and loop for category and the message.

```py
<section class="messages">
   {% for category, msg in get_flashed_messages(with_categories=true)%}
   <p class="{{category}}">{{msg}}</p>
   {% endfor %}
</section>
```

And look! Our category is our class! Now we can style it!

---

# DEBUGGING

using a plain ol `print()` function is pretty annoying

That is why we have the Flask Debug Toolbar!!  
Flask Debug Toolbar gives us an interactive terminal!! All on its own!!

the PIN is found in server terminal.

---

# Python Debugger (pdb)

-a method that will set a breakpoint in a function

```py
def my_function():
   ...
   import pdb
   pdb.set_trace()
   ...
```

Debugger Commands:
|Key|Command|
|------|------|
|?|Get Help|
|\| | List code where I am |
|p|Print this expression|
|pp|_Pretty_ Print this expression|
|n|Go to next line (Step Over)|
|s|Step into next function call|
|r|Return from function call|
|c|Continue on to next breakpoint|
|w|Print "Frame" (Where Am I?)|
|q|Quit debugger|

Most of the time, the `import` is done at the time of the set_trace() call.

---

# JSON!!

APIs often respond with JSON. So can we!

```py
@app.route("/some/route")
def some_route():
   """Route that returns JSON"""

   return '{"name":"Whiskey". "cute":"hella"}'
```

We could type it ourself, lol.

-  It's pretty bad tho. Prone to mistakes
-  It can be necessary to send a header to the browser that notifies it "Hey, you're looking at JSON"
   -  Some AJAX plugins are better than others at realizing this.

So we have a `jsonify()` funtion as part of Flask!.

```py
from flask import jsonify

@app.route("/some/route")
def some_route():
   """Route that returns JSON"""

   info = {"name":"Whiskey", "cute":"hella"}
   return jsonify(info)
```

`jsonify` actually converts everything into a whole entire complete `response object`. It does **so much** behind the scenes!!
