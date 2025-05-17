Here's a simple React.js + MySQL project setup for Ubuntu 24.04. It’s a basic CRUD (Create, Read, Update, Delete) app to manage users. The stack includes:

    Frontend: React.js

    Backend: Node.js + Express

    Database: MySQL

🔧 Step-by-Step Guide
✅ 1. Prerequisites

Install the required software:

sudo apt update
sudo apt install nodejs npm mysql-server

Check versions:

node -v
npm -v
mysql --version

📁 Project Structure

myapp/
├── backend/
│   ├── server.js
│   └── db.js
├── frontend/
│   └── React app (via create-react-app)
└── README.md

🛠️ 2. MySQL Setup

Log in to MySQL and create a database and table:

sudo mysql -u root -p

CREATE DATABASE userdb;
USE userdb;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

🔙 3. Backend Setup (Node.js + Express)

Create a folder called backend:

mkdir -p myapp/backend && cd myapp/backend
npm init -y
npm install express mysql cors body-parser

db.js – MySQL connection:

const mysql = require('mysql');

const db = mysql.createConnection({
  host: 'localhost',
  user: 'root',
  password: '', // enter your MySQL password if set
  database: 'userdb'
});

db.connect(err => {
  if (err) throw err;
  console.log('MySQL Connected...');
});

module.exports = db;

server.js – API routes:

const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
const db = require('./db');

const app = express();
app.use(cors());
app.use(bodyParser.json());

// Get all users
app.get('/users', (req, res) => {
  db.query('SELECT * FROM users', (err, results) => {
    if (err) throw err;
    res.send(results);
  });
});

// Add a user
app.post('/users', (req, res) => {
  const { name, email } = req.body;
  db.query('INSERT INTO users (name, email) VALUES (?, ?)', [name, email], (err, result) => {
    if (err) throw err;
    res.send({ id: result.insertId, name, email });
  });
});

// Delete user
app.delete('/users/:id', (req, res) => {
  db.query('DELETE FROM users WHERE id = ?', [req.params.id], (err) => {
    if (err) throw err;
    res.send({ success: true });
  });
});

app.listen(3001, () => console.log('Server running on http://localhost:3001'));

Start backend:

node server.js

⚛️ 4. Frontend Setup (React)

Open a new terminal:

cd myapp
npx create-react-app frontend
cd frontend
npm install axios

Edit App.js in frontend/src/:

import React, { useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [users, setUsers] = useState([]);
  const [form, setForm] = useState({ name: '', email: '' });

  const fetchUsers = async () => {
    const res = await axios.get('http://localhost:3001/users');
    setUsers(res.data);
  };

  useEffect(() => {
    fetchUsers();
  }, []);

  const handleSubmit = async (e) => {
    e.preventDefault();
    await axios.post('http://localhost:3001/users', form);
    setForm({ name: '', email: '' });
    fetchUsers();
  };

  const deleteUser = async (id) => {
    await axios.delete(`http://localhost:3001/users/${id}`);
    fetchUsers();
  };

  return (
    <div>
      <h1>User Manager</h1>
      <form onSubmit={handleSubmit}>
        <input value={form.name} onChange={e => setForm({...form, name: e.target.value})} placeholder="Name" />
        <input value={form.email} onChange={e => setForm({...form, email: e.target.value})} placeholder="Email" />
        <button type="submit">Add</button>
      </form>

      <ul>
        {users.map(u => (
          <li key={u.id}>{u.name} ({u.email}) <button onClick={() => deleteUser(u.id)}>Delete</button></li>
        ))}
      </ul>
    </div>
  );
}

export default App;

Start frontend:

npm start

✅ Final Steps

    Make sure both backend (localhost:3001) and frontend (localhost:3000) are running.

    React will fetch and display users, and you can add/delete users.
