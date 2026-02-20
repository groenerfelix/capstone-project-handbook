# Chapter 3: CRUD Operations with Forms and Requests

## Project setup

Now, you will make dedicated pages for the CRUD operations. Create a file structure that looks like this:

```
your_project/
├── app.py
└── templates/
    ├── index.html
    ├── add_user.html
    ├── edit_user.html
    └── view_user.html
```

Install the required packages:

```
pip install flask flask-sqlalchemy pymysql
```


## Updates to the server code

You will need to make some small changes to the server code of the previous chapter. Primarily, we are no longer passing information as URL parameters (e.g., `/add_user/<str:username>/`). Instead, we now use **POST requests** and transmit the new variables in the request body. Note that we have to import `request` from the Flask package for this.

!!! warning "We are still accepting GET requests"
    - `GET` (e.g., just typing in the URL) will render and return the template
    - `POST` (i.e., form action) will make database changes
    - In real-world systems you would likely use more specific HTTP methods like `PATCH`, `PUT`, and `DELETE` but we are keeping it simple for this class.

```python title="app.py" linenums="1"
from flask import Flask, render_template, redirect, url_for, flash, request
from flask_sqlalchemy import SQLAlchemy
from flask_bootstrap import Bootstrap5

app = Flask(__name__)
bootstrap = Bootstrap5(app)

app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:mysqlrootpassword@localhost:3306/flask_app'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
app.config['SECRET_KEY'] = 'your-secret-key-here'

db = SQLAlchemy(app)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)

with app.app_context():
    db.create_all()

@app.route('/')
def index():
    users = User.query.all() 
    return render_template('index.html', users=users)

# CREATE - Add new user
@app.route('/add', methods=['GET', 'POST']) # (1)!
def add_user():
    if request.method == 'POST': # (2)!
        username = request.form['username'] # (3)!
        email = request.form['email']
        if not username or not email:  # (4)!
            flash('Please fill in all fields', 'error')
            return redirect(url_for('add_user'))
        try:
            new_user = User(username=username, email=email)
            db.session.add(new_user)
            db.session.commit()
            flash('User added successfully!', 'success')
            return redirect(url_for('index')) # (5)!
        except Exception as e:
            flash(f'Error adding user: {str(e)}', 'error')
            return redirect(url_for('add_user')) # (6)!

    return render_template('add_user.html') # (7)!

# READ - View an individual user
@app.route('/view')
def view_user():
    # TODO: Implement view user functionality
    return redirect(url_for('index'))

# UPDATE - Edit an individual user
@app.route('/update')
def update_user():
    # TODO: Implement update user functionality
    return redirect(url_for('index'))

# DELETE - Delete a user
@app.route('/delete')
def delete_user():
    # TODO: Implement delete user functionality
    return redirect(url_for('index'))
```

Make sure you understand every line of the `add_user()` function. Then, implement the remaining three of the four CRUD functions (i.e., replace the `# TODO` comments and placeholder return values with your code).


## Form pages

Copy the following code snippets into their respective `.html` files. They only introduce HTML tables and forms as the input and output interfaces.


```html title="templates/add_user.html" linenums="1"
<!DOCTYPE html>
<html>
<head>
    <title>Add User</title>
    {{ bootstrap.load_css() }}
</head>
<body>
    <div class="container mt-5">
        <h1>Add New User</h1>
        {% with messages = get_flashed_messages(with_categories=true) %}
        {% if messages %}
        {% for category, message in messages %}
        <div class="alert alert-{{ category }}">{{ message }}</div>
        {% endfor %}
        {% endif %}
        {% endwith %}
        <form method="POST">
            <div class="mb-3">
                <label for="username" class="form-label">Username</label>
                <input type="text" class="form-control" id="username" name="username" required>
            </div>
            <div class="mb-3">
                <label for="email" class="form-label">Email</label>
                <input type="email" class="form-control" id="email" name="email" required>
            </div>
            <button type="submit" class="btn btn-primary">Add User</button>
            <a href="{{ url_for('index') }}" class="btn btn-secondary">Cancel</a>
        </form>
    </div>
    {{ bootstrap.load_js() }}
</body>
</html>
```

- `<form method="POST">` creates a native HTML form element that sends all fields within it to the same URL (in this case `/add`) but with the method `POST` instead of the regular `GET`.
- `<input>` elements contain the variables. The variable names are set by the `name` attribute. You should add `type` and `required` attributes.
- `<button type="submit">` then makes the POST API request, transmitting the filled out form fields. 


<figure markdown="span">
![](assets/images/ch3_add_page.png)
</figure>


```html title="templates/index.html" linenums="1"
<!DOCTYPE html>
<html>
<head>
    <title>User Management</title>
    {{ bootstrap.load_css() }}
</head>
<body>
    <div class="container mt-5">
        <h1>User Management</h1>
        <!-- Flash messages -->
        {% with messages = get_flashed_messages(with_categories=true) %}
            {% if messages %}
                {% for category, message in messages %}
                <div class="alert alert-{{ category }}">{{ message }}</div>
                {% endfor %}
            {% endif %}
        {% endwith %}
        <a href="{{ url_for('add_user') }}" class="btn btn-primary mb-3">Add New User</a>
        <table class="table">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Username</th>
                    <th>Email</th>
                    <th>Actions</th>
                </tr>
            </thead>
            <tbody>
                {% for user in users %}
                <tr>
                    <td>{{ user.id }}</td>
                    <td>{{ user.username }}</td>
                    <td>{{ user.email }}</td>
                    <td>
                        <a href="{{ url_for('view_user', id=user.id) }}" class="btn btn-info btn-sm">View</a>
                        <a href="{{ url_for('update_user', id=user.id) }}" class="btn btn-warning btn-sm">Edit</a>
                        <a href="{{ url_for('delete_user', id=user.id) }}" class="btn btn-danger btn-sm"
                            onclick="return confirm('Are you sure?')">Delete</a>
                    </td>
                </tr>
                {% endfor %}
            </tbody>
        </table>
    </div>
    {{ bootstrap.load_js() }}
</body>
</html>
```

<figure markdown="span">
![](assets/images/ch3_index_page.png)
</figure>

!!! warning "Test your understanding of how to create HTML forms by making the missing one to view an individual user and the one to edit an individual user! Make sure they work by adding, editing, and deleting a couple of users."


