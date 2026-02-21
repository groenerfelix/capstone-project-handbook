# Chapter8: Deploying to AWS EC2

!!! danger "TODO: work in progress"

## 8.1 Launching an EC2 Instance

Amazon EC2 (Elastic Compute Cloud) provides virtual servers to host your Flask application. This chapter walks you through deploying your application to the cloud.

1. Log in to the AWS Management Console  
2. Navigate to EC2 Dashboard  
3. Click Launch Instance  
4. Select Ubuntu Server 20.04 LTS (or newer)  
5. Choose t2.micro instance type (Free Tier eligible)  
6. Configure security group with the following rules:

Type | Port | Source | Purpose  
--- | --- | --- | ---  
SSH | 22 | Your IP | Terminal access  
HTTP | 80 | Anywhere | Web traffic  
Custom TCP | 5000 | Anywhere | Flask development server  

---

## 8.2 Connecting via SSH

### Option 1: AWS Console (Easiest)

1. In EC2 Dashboard, select your instance  
2. Click Connect  
3. Choose EC2 Instance Connect tab  
4. Click Connect to open a browser-based terminal  

### Option 2: Terminal/Command Prompt

```
# Make your key file readable only by you (Mac/Linux)
chmod 400 your-key.pem
```

```
# Connect to your instance
ssh -i "your-key.pem" ubuntu@your-ec2-public-dns
```

---

## 8.3 Server Setup and Dependencies

Once connected, run these commands:

```
# Update system packages
sudo apt update
sudo apt upgrade -y
```

```
# Install Python, pip, and Git
sudo apt install python3 python3-pip git -y
```

```
# Verify installation
python3 --version
pip3 --version
git --version
```

---

## 8.4 Deploying Your Application

```
# Navigate to home directory
cd /home/ubuntu
```

```
# Clone your repository
git clone https://github.com/your-username/your-flask-repo.git
cd your-flask-repo
```

```
# Install dependencies
pip3 install -r requirements.txt
```

Sample requirements.txt:

```
flask
flask-sqlalchemy
pymysql
flask-login
flask-bcrypt
```

---

## 8.5 Running in Production

Running the Flask Development Server:

```
# Run Flask accessible from any IP
flask run --host=0.0.0.0 --port=5000
```

```
# Or using Python directly
python3 app.py
```

Access your application:

Open your browser and navigate to: http://your-ec2-public-dns:5000

Running in Background with Screen:

By default, the Flask server stops when you close your terminal. Use 'screen' to keep it running:

```
# Install screen
sudo apt install screen
```

```
# Start a new screen session
screen
```

```
# Run your Flask app
flask run --host=0.0.0.0 --port=5000
```

Detach from screen: Press Ctrl+A, then D  

Later, reattach to see output:

```
screen -r
```

Useful Commands:

Command | Description  
--- | ---  
screen | Start a new screen session  
Ctrl+A, then D | Detach from current screen  
screen -r | Reattach to a screen session  
screen -ls | List all screen sessions  
exit | Close current screen session  

✓ Congratulations! Your Flask application is now running in the cloud. For production use, consider using a WSGI server like Gunicorn and a reverse proxy like Nginx.