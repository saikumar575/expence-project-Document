# 3 Tier Architecture

![alt text](3-tier-expense.svg)

# Expense Application

## Project Overview

Expense is a three-tier web application designed to manage expense transactions. The application consists of:

* Frontend Layer (Nginx Web Server)
* Backend Layer (Node.js Application)
* Database Layer (MySQL Server)

The frontend provides the user interface, the backend processes business logic and API requests, and MySQL stores transaction data.

---

# Architecture

## Three-Tier Architecture

```text
+-------------------+
|     End User      |
+---------+---------+
          |
          v
+-------------------+
|    Frontend       |
|      Nginx        |
|   Port : 80       |
+---------+---------+
          |
          | Reverse Proxy (/api)
          v
+-------------------+
|     Backend       |
|      Node.js      |
|   Port : 8080     |
+---------+---------+
          |
          |
          v
+-------------------+
|     Database      |
|      MySQL        |
|   Port : 3306     |
+-------------------+
```
# 3 Tier Architecture

![alt text](3-tier-expense.svg)

---

# Technology Stack

| Component       | Technology                  |
| --------------- | --------------------------- |
| Frontend        | HTML, CSS, JavaScript       |
| Web Server      | Nginx                       |
| Backend         | Node.js (v20+)              |
| Database        | MySQL 8.0                   |
| OS              | RHEL / Rocky Linux / CentOS |
| Service Manager | Systemd                     |

---

# Prerequisites

* Linux Server (RHEL 9 / Rocky Linux 9)
* Root or Sudo Access
* Internet Connectivity
* Open Ports:

  * 80 (HTTP)
  * 8080 (Backend API)
  * 3306 (MySQL)

---

# MySQL Database Setup

## Install MySQL Server

```bash
dnf install mysql-server -y
```

### Explanation

Installs MySQL Server package.

---

## Enable MySQL Service

```bash
systemctl enable mysqld
```

### Explanation

Configures MySQL service to start automatically during server boot.

---

## Start MySQL Service

```bash
systemctl start mysqld
```

### Explanation

Starts MySQL service immediately.

---

## Secure MySQL Root User

```bash
mysql_secure_installation --set-root-pass ExpenseApp@1
```

### Explanation

Sets root password and secures MySQL installation.

---

## Verify MySQL Connection

```bash
mysql -uroot -pExpenseApp@1
```

Check databases:

```sql
show databases;
```

---

# Backend Setup

## Install Node.js 20

Disable default NodeJS module:

```bash
dnf module disable nodejs -y
```

Enable NodeJS 20:

```bash
dnf module enable nodejs:20 -y
```

Install NodeJS:

```bash
dnf install nodejs -y
```

### Explanation

Installs Node.js version 20 required by the application.

---

## Create Application User

```bash
useradd expense
```

### Explanation

Creates dedicated application user for security purposes.

---

## Create Application Directory

```bash
mkdir /app
```

### Explanation

Stores application code in a standard location.

---

## Download Backend Code

```bash
curl -o /tmp/backend.zip https://expense-builds.s3.us-east-1.amazonaws.com/expense-backend-v2.zip
```

```bash
cd /app
```

```bash
unzip /tmp/backend.zip
```

### Explanation

Downloads and extracts backend source code.

---

## Install Dependencies

```bash
cd /app
npm install
```

### Explanation

Downloads all required Node.js packages.

---

## Configure Systemd Service

Create backend service file:

```bash
vim /etc/systemd/system/backend.service
```

Paste:

```ini
[Unit]
Description=Backend Service

[Service]
User=expense
Environment=DB_HOST="<MYSQL-SERVER-IP>"
ExecStart=/bin/node /app/index.js
SyslogIdentifier=backend

[Install]
WantedBy=multi-user.target
```

### Explanation

Creates a managed service for backend application.

---

## Reload Systemd

```bash
systemctl daemon-reload
```

---

## Start Backend Service

```bash
systemctl start backend
```

Enable at boot:

```bash
systemctl enable backend
```

---

## Install MySQL Client

```bash
dnf install mysql -y
```

### Explanation

Required to load schema into MySQL.

---

## Load Database Schema

```bash
mysql -h <MYSQL-SERVER-IP> -uroot -pExpenseApp@1 < /app/schema/backend.sql
```

### Explanation

Creates database, tables, and required structures.

---

## Restart Backend

```bash
systemctl restart backend
```

---

# Frontend Setup

## Install Nginx

```bash
dnf install nginx -y
```

### Explanation

Installs Nginx web server.

---

## Enable Nginx

```bash
systemctl enable nginx
```

---

## Start Nginx

```bash
systemctl start nginx
```

---

## Remove Default Website

```bash
rm -rf /usr/share/nginx/html/*
```

### Explanation

Deletes default Nginx landing page.

---

## Download Frontend Code

```bash
curl -o /tmp/frontend.zip https://expense-builds.s3.us-east-1.amazonaws.com/expense-frontend-v2.zip
```

---

## Extract Frontend Files

```bash
cd /usr/share/nginx/html
```

```bash
unzip /tmp/frontend.zip
```

### Explanation

Deploys frontend application files.

---

## Configure Reverse Proxy

Create configuration file:

```bash
vim /etc/nginx/default.d/expense.conf
```

Paste:

```nginx
proxy_http_version 1.1;

location /api/ {
    proxy_pass http://<BACKEND-IP>:8080/;
}

location /health {
    stub_status on;
    access_log off;
}
```

### Explanation

Routes API requests from frontend to backend service.

---

## Restart Nginx

```bash
systemctl restart nginx
```

---

# Verification

## Verify Frontend

Open browser:

```text
http://<FRONTEND-IP>
```

Expected Result:

* Expense application page loads successfully.

---

## Verify Backend

Check service:

```bash
systemctl status backend
```

---

## Verify Database

Connect:

```bash
mysql -uroot -pExpenseApp@1
```

List databases:

```sql
show databases;
```

Use database:

```sql
use transactions;
```

Show tables:

```sql
show tables;
```

View records:

```sql
select * from transactions;
```

---

# Project Directory Structure

```text
Expense/
│
├── Frontend (Nginx)
│   └── Static Web Files
│
├── Backend (Node.js)
│   ├── index.js
│   ├── package.json
│   └── schema/
│
└── Database
    └── MySQL
```

---

# Deployment Flow

```text
User
 ↓
Nginx Frontend
 ↓
Reverse Proxy (/api)
 ↓
Node.js Backend
 ↓
MySQL Database
```

---

# Author

Project Name: Expense

Architecture: Three-Tier Architecture

Technologies:

* Nginx
* Node.js
* MySQL
* Linux
* Systemd
