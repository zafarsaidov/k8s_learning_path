🧪 Practice Project: Flask + PostgreSQL App in Kubernetes

🧠 Learning Objectives
* Build and push Docker image of a Flask app
* Deploy a Flask app with PostgreSQL backend on Kubernetes
* Use environment variables for credentials
* Configure Deployment, Service, and Ingress
* Apply NetworkPolicy to restrict PostgreSQL access


## Flask App Code

# app.py
```python
from flask import Flask, request, jsonify
import psycopg2
import os

app = Flask(__name__)

DB_HOST = os.environ['DB_HOST']
DB_PORT = os.environ['DB_PORT']
DB_NAME = os.environ['DB_NAME']
DB_USER = os.environ['DB_USER']
DB_PASSWORD = os.environ['DB_PASSWORD']

def get_conn():
    return psycopg2.connect(
        host=DB_HOST, port=DB_PORT,
        dbname=DB_NAME, user=DB_USER,
        password=DB_PASSWORD
    )

@app.route('/add', methods=['POST'])
def add():
    data = request.json
    name = data.get('name')
    conn = get_conn()
    cur = conn.cursor()
    cur.execute("INSERT INTO users (name) VALUES (%s);", (name,))
    conn.commit()
    cur.close()
    conn.close()
    return jsonify({'status': 'added'}), 201

@app.route('/users', methods=['GET'])
def users():
    conn = get_conn()
    cur = conn.cursor()
    cur.execute("SELECT * FROM users;")
    rows = cur.fetchall()
    cur.close()
    conn.close()
    return jsonify(rows)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
```

# requirements.txt
```txt
Flask==2.3.2
psycopg2-binary==2.9.6
```



## SQL Queries for DB

```sql
CREATE USER user WITH PASSWORD 'password';

CREATE DATABASE usersdb OWNER user;

\c usersdb

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);
```



## 🧪 Optional: Verify

```sql
SELECT * FROM users;

INSERT INTO users (name) VALUES ('test');

SELECT * FROM users;
```
