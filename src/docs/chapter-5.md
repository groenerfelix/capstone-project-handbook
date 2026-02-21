# Chapter7: AWS RDS Database Setup

!!! danger "TODO: work in progress"


## 7.1 Creating an RDS Instance

Amazon RDS (Relational Database Service) provides a managed MySQL database in the cloud. This is essential for deploying your Flask application to production.

1. Log in to the AWS Management Console  
2. Navigate to RDS service  
3. Click Create Database  
4. Select Standard Create  
5. Choose MySQL as the engine  
6. Select the latest MySQL version  
7. Choose Free Tier template (if eligible)  

Configure Instance Settings:

Setting | Value | Description  
--- | --- | ---  
DB instance identifier | my-rds-instance | Unique name for your database  
Master username | admin | Database administrator username  
Master password | [strong password] | Save this securely!  
Initial database name | flask_app | Name of your first database  
Public access | Yes | Required for external connections  

---

## 7.2 Configuring Security Groups

Security groups act as a firewall, controlling which IP addresses can access your database.

1. Navigate to EC2 > Security Groups  
2. Find the security group associated with your RDS instance  
3. Click Edit inbound rules  
4. Add a new rule with the following settings:

Setting | Value  
--- | ---  
Type | MySQL/Aurora  
Port Range | 3306  
Source | Your IP (or 0.0.0.0/0 for development only)  

■■ Warning: Using 0.0.0.0/0 allows access from anywhere. Use this only for development. For production, restrict to specific IP addresses.

---

## 7.3 Connecting with MySQL Workbench

### Retrieve Connection Details:

1. In the RDS Dashboard, select your instance  
2. Copy the Endpoint (e.g., my-rds-instance.xyz.us-east-1.rds.amazonaws.com)  
3. Note the port (default: 3306)  

### Connect with MySQL Workbench:

1. Open MySQL Workbench  
2. Click Database > Connect to Database  
3. Enter the connection details:  
   - Hostname: Your RDS endpoint  
   - Port: 3306  
   - Username: admin  
   - Password: Your RDS password  
4. Click Test Connection to verify  
5. If successful, click OK to connect  

---

## 7.4 Integrating with Flask

Update your Flask application to connect to the RDS database instead of localhost:

config.py:

```python
import os
DB_USERNAME = "admin"
DB_PASSWORD = "your_rds_password"
DB_HOST = "your-rds-endpoint.region.rds.amazonaws.com"
DB_PORT = "3306"
DB_NAME = "flask_app"
SQLALCHEMY_DATABASE_URI = \
f"mysql+pymysql://{DB_USERNAME}:{DB_PASSWORD}@{DB_HOST}:{DB_PORT}/{DB_NAME}"
SQLALCHEMY_TRACK_MODIFICATIONS = False
```

app.py:

```python
from flask import Flask, jsonify
from flask_sqlalchemy import SQLAlchemy
from config import SQLALCHEMY_DATABASE_URI
app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = SQLALCHEMY_DATABASE_URI
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)
class User(db.Model):
id = db.Column(db.Integer, primary_key=True)
name = db.Column(db.String(50), nullable=False)
@app.route('/')
def index():
return jsonify({'message': 'Connected to RDS MySQL!'})
if __name__ == '__main__':
app.run(debug=True)
```

✓ Tip: For production, store sensitive credentials in environment variables, not in your code.