# Chapter 2: MySQL Database Integration

As the database for this project you will you MySQL. This chapter will show you how to set up your local development server, how to access it in Flask, and basic operations.

## Installing MySQL

### Windows Installation

1. Go to [dev.mysql.com/downloads](https://dev.mysql.com/downloads)  
2. Download 'MySQL Installer for Windows' (mysql-installer-community)  
3. Run the installer and select 'Custom' installation  
4. Select 'MySQL Server' and 'MySQL Workbench' from the products list
5. Follow the wizard to complete installation  
6. Set a root password and remember it! 

### Mac Installation

1. Go to [dev.mysql.com/downloads](https://dev.mysql.com/downloads)  
2. Select macOS and download the DMG archive  
3. Double-click the downloaded DMG file  
4. Run the installer package (`.pkg`)
5. Follow the installation wizard  
6. **IMPORTANT:** Save the temporary root password shown at the end!  

!!! warning "If you lose your MySQL root password, you'll need to reset it through a recovery process."

!!! info "What are MySQL Server and Workbench?"
    - The server is what holds your databases, tables, entries.
    - The workbench is a program that we will use to look at the tables and verify that our flask app successfully stored data there.  

## Creating the database in MySQL Workbench

Open the MySQL Workbench program and connect to your server (local instance) with the password you just created.
<figure markdown="span">
![](assets/images/ch2_workbench_create_schema_1.png)
</figure>

Then, on the left, switch to "Schemas", right click, and create a new schema.
<figure markdown="span">
![](assets/images/ch2_workbench_create_schema_2.png)
</figure>

Name the database "flask_app".
<figure markdown="span">
![](assets/images/ch2_workbench_create_schema_3.png)
</figure>

MySQL Workbench will create the simple SQL script that creates the new database (schema). Execute that script and see your new database show up in the left panel.
<figure markdown="span">
![](assets/images/ch2_workbench_create_schema_4.png)
</figure>

You won't add or manipulate tables or entries through workbench. The next sections will show you how to do that in python.


## Configuring Flask-SQLAlchemy

Flask-SQLAlchemy is an ORM (Object-Relational Mapper) that lets you interact with databases using Python objects instead of raw SQL queries.
Install the required packages:

```
pip install flask flask-sqlalchemy pymysql
```

You will later connect your flask app to the database with a code snippet similar to this:

```python
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:mysqlrootpassword@localhost:3306/flask_app'
```

In the above code, `app` is the flask app instance. `.config['X'] = Y` sets a config variable X to a value Y. The imported SQLAlchemy library accesses this config to retrieve the database URI.

You will need to update the exact value for your own project. The *database connection string* is constructed as follows:

```
mysql+pymysql://username:password@hostname:port/database_name
```

- `mysql` is the database "dialect"
- `pymysql` is the database driver
- `root` is the username
- `password` is your root password
- `hostname` and `port` are your URL (localhost during development)
- `database_name` will be whatever we set as the project name


## Creating database models

You won't create or manipulate any tables in the MySQL workbench. Instead, you will create tables and columns through the flask app. To create a table, we have to create a data model as a python class. Save the following code as your `app.py` and make sure you understand every single line of it:

```python title="app.py" linenums="1"
from flask import Flask, render_template
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:mysqlrootpassword@localhost:3306/flask_app'

db = SQLAlchemy(app)

class User(db.Model): # (1)!
    id = db.Column(db.Integer, primary_key=True) # (2)!
    username = db.Column(db.String(80), unique=True, nullable=False) # (3)!
    email = db.Column(db.String(120), unique=True, nullable=False)

with app.app_context():
    db.create_all() # (4)!
```

1. The user model inherits from the `db.Model` superclass, telling the database to create a table for it
2. This creates a column with the datatype integer and sets it as the primary key
3. This creates a column with the datatype string that has to be unique, cannot be null (i.e., left out), and has a max length of 80 characters
4. This creates all the tables when the app starts

### SQLAlchemy column options

SQLAlchemy supports these common datatypes for columns:

Column Type | Description | Example  
--- | --- | ---  
db.Integer | Whole numbers | id = db.Column(db.Integer)  
db.String(n) | Text with max length n | name = db.Column(db.String(80))  
db.Text | Long text (no length limit) | bio = db.Column(db.Text)  
db.Float | Decimal numbers | price = db.Column(db.Float)  
db.Boolean | True/False values | active = db.Column(db.Boolean)  
db.DateTime | Date and time | created = db.Column(db.DateTime)  

You might need these common parameters for your project:

- `primary_key=True` - Makes this column the primary key  
- `unique=True` - Values must be unique across all rows  
- `nullable=False` - Column cannot be empty (required field)  
- `default=value` - Sets a default value if none provided


### Flask Shell Operations

You can interact with your database directly from the Flask shell. This is useful for testing and debugging. You basically write python code line by line into your terminal.

1. Start the flask shell with `flask shell` instead of `flask run` to start your app. This allows you to continue working in the terminal.
2. Create a new user

    ```bash
    new_user = User(username='john', email='john@example.com') # (1)!
    db.session.add(new_user) # (2)!
    db.session.commit() # (3)!
    ```

    1. We create a new User object
    2. We add this user to the database but haven't committed the change yet
    3. We save the change to the database

3. Query all users

    ```bash
    users = User.query.all()
    for user in users:
        print(user.username, user.email)
    ```
    Instead of `.all()` users you could query with filters (e.g., `.filter_by(username='john').first()`)

4. Exit the shell with `exit()`


### Verifying changes in MySQL Workbench

You can inspect the database in MySQL Workbench. To create queries, click on "New Query Tab" and type in your query. To execute it, click the flash icon. You will see the results at the bottom. Start by selecting the correct database with `USE flask_app;`



<figure markdown="span">
  ![Alt](assets/images/ch2_workbench_check_highlighted.png)
</figure>

- To view all tables in your database write `SHOW TABLES;`
- To see the structure of a table called `user` write `DESCRIBE user;`
- To see all entries in a table called `user` write `SELECT * FROM user;`
- To count how many entries are in the table `user` write `SELECT COUNT(*) FROM user;`

You can always come back to this program whenever you are unsure about the state of a specific table or the success of an operation.

## Basic database operations (CRUD)

The four fundamental database operations as often called by their acronym "CRUD": create, read, update, delete. Below you will find implementations for each of these operations as different routes in the flask application.

General explanations:

- routes containing strings like `<int:user_id>` or `<string:email>` allow you to add parameters to your route. The format is `<datatype:variable_name>`.
- `db.session` is the object that manages all the changes you make to the database. The changes are only committed when you run `db.session.commit()` and can be reverted with `db.session.rollback()` if an error occurred.
- We end every route with `return redirect(url_for('index'))` to bring visitors back to the home page, even though they clicked on a link (e.g., to `/add_user/`)

??? info "In the real world you wouldn't use GET requests for all of these"
    The examples below all use the HTTP method "GET" which is simple but unsafe. In real products you would use POST, PATCH, and DELETE. But in this project we want to keep it simple.

### CREATE - Add a new user

```python
@app.route('/add_user/<string:username>/<string:email>') # (1)!
def add_user(username, email):
    new_user = User(username=username, email=email) # (2)!
    try:
        db.session.add(new_user) # (3)!
        db.session.commit() # (4)!
    except Exception as e:
        db.session.rollback() # (5)!
    return redirect(url_for('index')) # (6)!
```

1. This endpoint requires a username and an email to be passed
2. We create a new User object
3. We add this user to the database but haven't committed the change yet
4. We commit the change but only if there were no exceptions in the line before (e.g., username already exists)
5. If an exception occurred, then we rollback the change (adding the user)
6. Regardless of the outcome, we redirect the browser to the landing page.


### READ - Get a specific user

```python
@app.route('/read_user/<int:user_id>') # (1)!
def read_user(user_id):
    user = User.query.get_or_404(user_id) # (2)!
    return f"User: {user.username}, Email: {user.email}" # (3)!
```

1. This endpoint requires a integer user id to be passed
2. If the user is not found, we are returning a HTTP error code 404 "Not Found"
3. If the user is found, we return the string similar to the hello-world example, not a full HTML template.


### UPDATE - Modify a user

```python
@app.route('/update_user/<int:user_id>/<string:username>/<string:email>') # (1)!
def update_user(user_id, username, email):
    user = User.query.get_or_404(user_id)
    user.username = username # (2)!
    user.email = email
    try:
        db.session.commit()
    except Exception as e:
        db.session.rollback()
    return redirect(url_for('index'))
```

1. For this endpoint, we need the user id of the entry that we want to change and the new username and email.
2. Here we are updating the values in the User object without committing the changes yet


### DELETE - Remove a user

```python
@app.route('/delete_user/<int:user_id>')
def delete_user(user_id):
    user = User.query.get_or_404(user_id)
    try:
        db.session.delete(user)
        db.session.commit()
    except Exception as e:
        db.session.rollback()
    return redirect(url_for('index'))
```
