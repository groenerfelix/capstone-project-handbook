# Chapter 4: CRUD Operations with Forms


!!! danger "Add the simple app.py back to chapter 2, introduce flash messages, and let them play with that by passing the parameters"


!!! warning "This will not yet work because more changes are needed to the server and the templates!"
    In the next steps you will add new routes and templates to let people interface with the database.

!!! info "Note that we have imported `redirect, url_for, flash` from the flask library
    - `redirect` lets you send the client to another route (e.g., to the user profile page after adding a new user)
    - `url_for` gives you the url for a route (e.g., index); you have used these in the previous section already in the html templates.
    - `flash` is used to send pop-up notifications from the server to the client


## 4.2 Project Setup

Project Structure:

```
your_project/
├── app.py
└── templates/
    ├── index.html
    ├── add_user.html
    ├── edit_user.html
    └── view_user.html
```

Install required packages:

```
pip install flask flask-sqlalchemy pymysql
```

---

## 4.3 Creating the User Model

Complete app.py:

```python title="app.py"
from flask import Flask, render_template, request, redirect, url_for, flash
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:mysqlrootpassword@localhost:3306/flask_app'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)


with app.app_context():
    db.create_all()
```

---

## 4.4 Building HTML Forms

### templates/index.html - Main page with user list:

```html
<!DOCTYPE html>
<html>
<head>
<title>User Management</title>
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css
"
rel="stylesheet">
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
<a href="{{ url_for('view_user', id=user.id) }}"
class="btn btn-info btn-sm">View</a>
<a href="{{ url_for('edit_user', id=user.id) }}"
class="btn btn-warning btn-sm">Edit</a>
<a href="{{ url_for('delete_user', id=user.id) }}"
class="btn btn-danger btn-sm"
onclick="return confirm('Are you sure?')">Delete</a>
</td>
</tr>
{% endfor %}
</tbody>
</table>
</div>
</body>
</html>
```

### templates/add_user.html - Form to add new user:

```html
<!DOCTYPE html>
<html>
<head>
<title>Add User</title>
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css
"
rel="stylesheet">
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
<input type="text" class="form-control" id="username"
name="username" required>
</div>
<div class="mb-3">
<label for="email" class="form-label">Email</label>
<input type="email" class="form-control" id="email"
name="email" required>
</div>
<button type="submit" class="btn btn-primary">Add User</button>
<a href="{{ url_for('index') }}" class="btn btn-secondary">Cancel</a>
</form>
</div>
</body>
</html>
```

---

## 4.5 Implementing Routes

### READ - Display all users

```python
@app.route('/')
def index():
users = User.query.all()
return render_template('index.html', users=users)
```

### CREATE - Add new user

```python
@app.route('/add', methods=['GET', 'POST'])
def add_user():
if request.method == 'POST':
username = request.form['username']
email = request.form['email']
if not username or not email:
flash('Please fill in all fields', 'error')
return redirect(url_for('add_user'))
try:
new_user = User(username=username, email=email)
db.session.add(new_user)
db.session.commit()
flash('User added successfully!', 'success')
return redirect(url_for('index'))
except Exception as e:
flash(f'Error adding user: {str(e)}', 'error')
return redirect(url_for('add_user'))
return render_template('add_user.html')
```

### READ - View single user

```python
@app.route('/view/<int:id>')
def view_user(id):
user = User.query.get_or_404(id)
return render_template('view_user.html', user=user)
```

### UPDATE - Edit user

```python
@app.route('/edit/<int:id>', methods=['GET', 'POST'])
def edit_user(id):
user = User.query.get_or_404(id)
if request.method == 'POST':
username = request.form['username']
email = request.form['email']
if not username or not email:
flash('Please fill in all fields', 'error')
return redirect(url_for('edit_user', id=id))
try:
user.username = username
user.email = email
db.session.commit()
flash('User updated successfully!', 'success')
return redirect(url_for('index'))
except Exception as e:
flash(f'Error updating user: {str(e)}', 'error')
return redirect(url_for('edit_user', id=id))
return render_template('edit_user.html', user=user)
```

### DELETE - Remove user

```python
@app.route('/delete/<int:id>')
def delete_user(id):
user = User.query.get_or_404(id)
try:
db.session.delete(user)
db.session.commit()
flash('User deleted successfully!', 'success')
except Exception as e:
flash(f'Error deleting user: {str(e)}', 'error')
return redirect(url_for('index'))
if __name__ == '__main__':
app.run(debug=True)
```

---

## 4.6 Flash Messages

Flash messages provide feedback to users about the results of their actions. They're stored in the session and displayed once on the next page load.

How to use flash messages:

```python
# In your route:
flash('Operation successful!', 'success') # Green alert
flash('Something went wrong', 'error') # Red alert
flash('Please check your input', 'warning') # Yellow alert
flash('FYI: This is info', 'info') # Blue alert
```

Display in template:

```html
{% with messages = get_flashed_messages(with_categories=true) %}
{% if messages %}
{% for category, message in messages %}
<div class="alert alert-{{ category }}">{{ message }}</div>
{% endfor %}
{% endif %}
{% endwith %}
```
