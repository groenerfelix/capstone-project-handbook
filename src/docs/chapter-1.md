# Chapter 1: Creating Flask Apps

!!! warning "Remember to set up your environment correctly!"
    In the beginning of every chapter, you will need to go through the steps outlined in Chapter 0. Make sure that you have your virtual environment set up and activated!


## Introduction to Flask

### What is Flask?

You will use Flask as a web framework for this project. Flask is a lightweight python package that lets you build web apps and APIs. It is simple to set up and get going because it handles the complicated functions under the hood.

### Installing Flask
Install Flask in your active virtual envionment with `pip install flask`.

You can check whether Flask is installed with `pip list`.

### A minimal Flask application

In your project folder, create a new file called `app.py`.

```python
from flask import Flask # (1)!

app = Flask(__name__) # (2)!

@app.route('/') # (3)!
def hello(): # (4)!
    return 'Hello, World!' # (5)!
```

1. Importing the `Flask` class from the `flask` library
2. Creating an instance of that class and passing it the name of the module. Look up `__name__ in python` for more info.
3. This is a special function added by Flask to indicate which function should be called when someone visits `/` (i.e., the homepage). This parameter can be any route like `/about`. Look up `decorator functions in python` if you are unfamiliar with the `@` notation.
4. We define the function that gets called. You can name this anything but make sure the name isn't already taken.
5. This string will be returned to whoever accesses the server. In a browser, this will be rendered as HTML.

Start your app with `flask run`. It should show you in the console what URL you need to type into your browser to access the page (typically [http://127.0.0.1:5000/](http://127.0.0.1:5000/)). This should now display "Hello, World!".

You can stop your app with `CTRL + C` or by closing the terminal.

!!! warning "Troubleshooting common issues"
    - "Flask not found" means that either the venv is not activated or Flask is not installed.
    - "Port 5000 already in use" means that another app is using the port (maybe your own flask app in another terminal!). Close whatever other app is using the port or change the port of your new app with `flask run --port=5001`.


### Debug mode

You can enable debug mode for your application during development by running `flask run --debug`. You should never use this in production or submissions!

Debug mode has two advantages for your development process:
- **Hot reload:** It automatically restarts the server when you save changes to your code.
- **Detailed errors:** It shows helpful details in the browser when something goes wrong.


## Working with templates

Templates allow you to separate your Python code from your HTML. Instead of embedding HTML directly in your Python code, you create separate HTML files that Flask renders dynamically. Flask uses **Jinja2**, a powerful templating engine that allows you to include Python-like expressions in your HTML. The Jinja-specific code blocks that differ from standard HTML are indicated by `{% ... %}`.

??? info "Why use templates?"
    - Separates logic from presentation (cleaner code)  
    - Enables template inheritance (DRY principle)  
    - Makes it easy to maintain consistent layouts  
    - Allows dynamic content insertion


### Project structure

We add a folder which contains our HTML templates. It has to be in this exact location and has to be named `tamplates`:

```
my_flask_project/
├── venv/
└── my_flask_app/
    ├── .gitignore
    ├── app.py
    └── templates/
        ├── base.html
        ├── home.html
        └── about.html
```

### Base templates

A base template contains the common HTML structure that several (or all) pages share. Think of it like abstracting shared functionality into a superclass. Other templates extend/inherit this base and fill in specific content blocks.

Copy this code into `templates/base.html` and make sure you understand every single line of it:

- `{% block parameter_name %}{% endblock %}` is a placeholder that will be filled by the child template
- `{{ url_for('resource_name') }}` is a function that the server will replace with the url like `/about`.

```html title="templates/base.html" linenums="1"
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{% block title %}{% endblock %} - My Flask App</title>
</head>
<body>
    <nav>
        <a href="{{ url_for('home') }}">My Flask App</a>
        <ul>
            <li>
                <a href="{{ url_for('home') }}">Home</a>
            </li>
            <li>
                <a href="{{ url_for('about') }}">About</a>
            </li>
        </ul>
    </nav>
    {% block content %}{% endblock %}
    <footer>
        <p>© 2026 My Flask App</p>
    </footer>
</body>
</html>
```



### Template inheritance

Child templates can now extend the base template and fill in the content blocks.

- `{% extends "base.html" %}` means that we are using the base template as a starting point.
- `{% block parameter_name %}...{% endblock %}` fills the placeholder with the content enclosed within the block.

Copy this code into `templates/home.html` and `templates/about.html` respectively and make sure you understand every single line: 

```html title="templates/home.html"
{% extends "base.html" %}

{% block title %}Home{% endblock %}

{% block content %}
<h1>Welcome to My Flask App</h1>
<p>This is a simple example using Flask templates with Bootstrap.</p>
{% endblock %}
```


```html title="templates/about.html"
{% extends "base.html" %}

{% block title %}About{% endblock %}

{% block content %}
<h1>This is the About Page</h1>
<p>This is a simple example showing how template inheritance allows you to reuse common layout elements across multiple pages.</p>
{% endblock %}
```

Notice how you don't have to write the same code for the navigation links and the footer again for the home and about pages. This saves you time, especially if you want to make changes to that code later.

### Using templates in your app

Now, we make a small change to our `app.py` to render and return the HTML code instead of "Hello, World".

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/")
def home(): # (2)!
    return render_template("home.html")

@app.route("/about")
def about():
    return render_template("about.html")
```

The `render_template` function from the Flask package converts our HTML/Jinja templates into HTML responses. If you now start your app and visit the URL, you should see our simple website. You can click on the "about" link at the top to go to the other website.
<figure markdown="span">
![Unstyled Homepage](assets/images/ch1_template_no_bootstrap_home.png)
</figure>
<figure markdown="span">
![Unstyled About page](assets/images/ch1_template_no_bootstrap_about.png)
</figure>



## Styling with Bootstrap

Next, we are going to work on making these websites look better. Bootstrap is a popular CSS framework that helps you create professional-looking, responsive websites quickly. You will use Bootstrap-Flask for easy integration.

Install the required package with `pip install Bootstrap-Flask`.

Add bootstrap to your project by integrating these lines:

```python title="app.py" linenums="1" hl_lines="2 5"
from flask import Flask, render_template
from flask_bootstrap import Bootstrap5

app = Flask(__name__)
bootstrap = Bootstrap5(app)

@app.route("/")
def home():
    return render_template("home.html")

@app.route("/about")
def about():
    return render_template("about.html")
```

This allows us to use class names for which bootstrap has already predefined styles. Replace the code in `/templates/base.html` and `/templates/home.html` with the following, respectively.

- `{{ bootstrap.load_css() }}` loads the default CSS from a content-delivery network (CDN)
- `{{ bootstrap.load_js() }}` does the same with JavaScript
- class names like `bg-dark text-light mt-5 py-3` are now shortcuts to styling colours, fonts, and spacings (e.g., `mt` = margin top)

!!! info "Find all your style options and premade components at [https://getbootstrap.com/](https://getbootstrap.com/)!"

```html title="/templates/base.html" linenums="1" hl_lines="6 37"
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    {{ bootstrap.load_css() }}
    <title>{% block title %}{% endblock %} - My Flask App</title>
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="{{ url_for('home') }}">My Flask App</a>
            <button class="navbar-toggler" type="button"
            data-bs-toggle="collapse" data-bs-target="#navbarNav">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarNav">
            <ul class="navbar-nav">
                <li class="nav-item">
                    <a class="nav-link" href="{{ url_for('home') }}">Home</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link" href="{{ url_for('about') }}">About</a>
                </li>
            </ul>
            </div>
        </div>
    </nav>
    <div class="container mt-4">
    {% block content %}{% endblock %}
    </div>
    <footer class="bg-dark text-light mt-5">
        <div class="container py-3">
            <p class="text-center mb-0">© 2026 My Flask App</p>
        </div>
    </footer>
    {{ bootstrap.load_js() }}
</body>
</html>
```

```html title="/templates/home.html" linenums="1"
{% extends "base.html" %}
{% block title %}Home{% endblock %}
{% block content %}
<div class="row">
    <div class="col-md-12">
        <h1 class="display-4">Welcome to My Flask App</h1>
        <p class="lead">This is a simple example using Flask templates with Bootstrap.</p>
    </div>
</div>
<div class="row mt-4">
    <div class="col-md-4">
        <div class="card">
            <div class="card-body">
                <h5 class="card-title">Feature One</h5>
                <p class="card-text">This is a Bootstrap card component.</p>
                <a href="#" class="btn btn-primary">Learn More</a>
            </div>
        </div>
    </div>
    <div class="col-md-4">
        <div class="alert alert-success" role="alert">
            <h5 class="alert-heading">Did you know?</h5>
            <p>Bootstrap makes responsive design easy!</p>
        </div>
    </div>
    <div class="col-md-4">
        <div class="d-grid gap-2">
            <button class="btn btn-primary">Primary Button</button>
            <button class="btn btn-secondary">Secondary Button</button>
        </div>
    </div>
</div>
{% endblock %}
```

With this code your homepage now looks like this:
<figure markdown="span">
![Homepage with bootstrap styling](assets/images/ch1_template_bootstrap_home.png)
</figure>

## Adding custom static files (CSS & JavaScript)

If you ever want to add additional styling instructions or client-side code that is not included in bootstrap, then you can link custom `.css` and `.js` files.

Create the following two files in new folder and place the code below into them. Your final folder structure should look like this. The new folders and files should be named and placed exactly like this:

```
my_flask_project/
├── venv/
└── my_flask_app/
    ├── .gitignore
    ├── app.py
    ├── templates/
    │   ├── base.html
    │   ├── home.html
    │   └── about.html
    └── static/
        ├── css/
        │   └── style.css
        └── js/
            └── script.js
```

```css title="static/css/style.css" linenums="1"
/* Custom styles */
body {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
}
.container {
    flex: 1;
}
footer {
    margin-top: auto;
}

/* Card hover effect */
.card {
    transition: transform 0.2s;
}
.card:hover {
    transform: translateY(-5px);
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}
```

```javascript title="static/js/script.js" linenums="1"
// Highlight active navigation link
document.addEventListener('DOMContentLoaded', function() {
    const currentLocation = window.location.pathname;
    const navLinks = document.querySelectorAll('.nav-link');
    navLinks.forEach(link => {
        if (link.getAttribute('href') === currentLocation) {
            link.classList.add('active');
        }
    });
});
```

Then, link the custom static files in your html templates. Using the following variables ensures correct paths regardless of where your app is deployed.

- `<link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">` in the `head` of your template.
- `<script src="{{ url_for('static', filename='js/script.js') }}"></script>` at the end of the `body` of the template.

You can include them in the specific child template if only that one needs it or in the base template if all pages need it. Note that you can only overwrite the default bootstrap css/js if the link to your custom file comes *after* (i.e., in any line below) the `{{ bootstrap.load_css() }}`, for example.

