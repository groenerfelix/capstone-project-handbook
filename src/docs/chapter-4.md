# Chapter 4: Authentication and Access Control

Flask-Login handles user session management, making it easy to implement login/logout functionality and restrict access to certain pages. 
Combined with role-based access control (RBAC), you can create applications where different users have different permissions.

You need to install a couple of additional packages:

```
pip install flask flask-login flask-sqlalchemy flask-bcrypt
```

## User model with roles

The new user model looks as follows. We are inheriting from the UserMixin class of the Flask-Login package. We are storing the password instead of the email (more about security later), and a role (e.g., "user" or "admin").

```python
class User(UserMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(250), unique=True, nullable=False)
    password = db.Column(db.String(250), nullable=False)
    role = db.Column(db.String(50), default="user", nullable=False) # "user" or "admin"
```

!!! warning "If you are using the same MySQL database for this chapter as for the one before, you will need to delete the User table!"
    - In MySQL Workbench run the script `DROP TABLE user;`.
    - Confirm in the left panel that the table is gone.
    - (Re-)start your Flask app to create the new user table.


## Flask-Login

The following code is required for Flask-Login to work. The imported functions (e.g., `login_user`) are handling and hiding most of the compicated procedures so that you don't have to deal with them.

```python
from flask_login import LoginManager, UserMixin, login_user, logout_user, login_required, current_user

@login_manager.user_loader # (1)!
def load_user(user_id):
    return User.query.get(int(user_id))

@app.route('/login', methods=["GET", "POST"])
def login():
    if request.method == "POST":
        user = User.query.filter_by(
        username=request.form.get("username")
        ).first()
        if user and user.password == request.form.get("password"): # (2)!
            login_user(user) # (3)!
            return redirect(url_for("home"))
    return render_template("login.html")

@app.route('/logout')
@login_required # (4)!
def logout():
    logout_user()
    return redirect(url_for("login"))
```

1. This tells Flask-Login where to find a specific User object in our system.
2. Here we are checking whether the password entered in the browser is the same as the one stored in the database.
3. We call the `login_user` function imported from the `flask_login` package and pass the `UserMixin` object.
4. `@login_required` is a decorator that sends an error page to the browser if the user is not logged in.

With this code we only get to the homepage after successfully logging in. Note that this snippet alone won't work. We still need all the other server code:


```python title="app.py" linenums="1"
from flask import Flask, render_template, request, url_for, redirect
from flask_sqlalchemy import SQLAlchemy
from flask_login import LoginManager, UserMixin, login_user, logout_user, login_required, current_user
app = Flask(__name__)

app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:mysqlrootpassword@localhost:3306/flask_app'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
app.config['SECRET_KEY'] = 'your-secret-key-here'

db = SQLAlchemy(app)

login_manager = LoginManager()
login_manager.init_app(app)
# User model with role-based access control
class User(UserMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(250), unique=True, nullable=False)
    password = db.Column(db.String(250), nullable=False)
    role = db.Column(db.String(50), default="user", nullable=False) # "user" or "admin"

with app.app_context():
    db.create_all()

# User loader - required by Flask-Login
@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))

@app.route('/register', methods=["GET", "POST"])
def register():
    if request.method == "POST":
        user = User(
        username=request.form.get("username"),
        password=request.form.get("password"),
        role="user" # Default role
        )
        db.session.add(user)
        db.session.commit()
        return redirect(url_for("login"))
    return render_template("sign_up.html")

@app.route('/login', methods=["GET", "POST"])
def login():
    if request.method == "POST":
        user = User.query.filter_by(
        username=request.form.get("username")
        ).first()
        if user and user.password == request.form.get("password"):
            login_user(user)
        return redirect(url_for("home"))
    return render_template("login.html")

@app.route('/logout')
@login_required
def logout():
    logout_user()
    return redirect(url_for("login"))

@app.route('/')
def home():
    return render_template("home.html")
```


## New HTML templates

You will need to create templates for `home`, `login`, and `sign_up`. The latter two are very similar to the `add_user` page that you already know, so you could use it as a starting point.

!!! info "For passwords use `<input type="password">`"

On the homepage, we want to display different content depending on the user role and authentication:

- If the user is logged in with role "user", show "This is visible only to regular users."
- If the user is logged in with role "admin", show "This is visible only to admins."
- If the user is not logged in at all, show "You are not logged in."


```html title="templates/home.html" linenums="1"
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Home</title>
</head>
<body>
    <nav>
        <ul>
            <li><a href="/login">Login</a></li>
            <li><a href="/register">Create account</a></li>
            {% if current_user.is_authenticated %}
            <li><a href="/logout">Logout</a></li>
            {% endif %}
        </ul>
    </nav>
    {% if current_user.is_authenticated %}
        <h1>Welcome, {{ current_user.username }}!</h1>
        {% if current_user.role == "admin" %}
            <h2>Admin Dashboard</h2>
            <p>This is visible only to admins: Manage users, settings, etc.</p>
            {% elif current_user.role == "user" %}
            <h2>User Dashboard</h2>
            <p>This is visible only to regular users.</p>
        {% endif %}
    {% else %}
    <h1>You are not logged in.</h1>
    {% endif %}
</body>
</html>
```

- `{% if current_user.is_authenticated %}` only shows the following content if the user is logged in.
- `{% if current_user.role == "admin" %}` only shows the following content if the user has the role "admin".



## Password hashing

!!! warning "Never store passwords as plain text in your database!"
    You will hash them into a string that can still be compared to verify a password, but not be reversed to reveal the password. Look up "hashing algorithms" if you want to learn more.

You will use Bcrypt from the Flask-Bcrypt package to hash passwords. You will need to do that everytime a password is entered (i.e., when signing up and signing in). 

At the top of your `app.py` import and initialize Bcrypt:

```python
from flask_bcrypt import Bcrypt
bcrypt = Bcrypt(app)
```

In the registration route, hash the new password and only store the hashed value:

```python
@app.route('/register', methods=["GET", "POST"])
def register():
    if request.method == "POST":
        hashed_password = bcrypt.generate_password_hash( # (1)!
            request.form.get("password")
        ).decode('utf-8')
        user = User(
            username=request.form.get("username"),
            password=hashed_password, # (2)!
            role="user"
        )
        db.session.add(user)
        db.session.commit()
        return redirect(url_for("login"))
    return render_template("sign_up.html")
```

1. Create the hash value for the password.
2. Create the User object with the hash value as the password.

In the login route, verify the password:

```python
@app.route('/login', methods=["GET", "POST"])
def login():
    if request.method == "POST":
        user = User.query.filter_by(
            username=request.form.get("username")
        ).first()
        if user and bcrypt.check_password_hash( # (1)!
            user.password, request.form.get("password")
        ):
            login_user(user)
            return redirect(url_for("home"))
    return render_template("login.html")
```

1. Compare the hashed password from the User object using the `check_password_hash` function instead of simple string comparison.


## Custom decorators

Just like `@login_required` you can create your own `@`-decorator functions that limit access to a page depending on custom conditions. You will need to import the `wrap` decorator from the pre-installed `functools` package. Use this wrapper to crate a function within a function like this:

```python
from functools import wraps

def decroator_function(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        # Your code here
        return f(*args, **kwargs)
    return decorated_function
```

The following example creates an `@admin_required` decorator that only allows logged-in users with the role "admin" to access the page:

```python
def admin_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if not current_user.is_authenticated or current_user.role != "admin":
            return redirect(url_for("home"))
        return f(*args, **kwargs)
    return decorated_function
```

You can also add parameters to the decorator function. The following example creates a decorator that only allows users with one of a list of roles to access the page like `@role_required("admin", "manager")`

```python
def role_required(*roles):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            if not current_user.is_authenticated or current_user.role not in roles:
                return redirect(url_for("home"))
            return f(*args, **kwargs)
        return decorated_function
    return decorator
```
