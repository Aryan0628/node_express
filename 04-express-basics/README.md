# Module 4: Express.js Basics

## What is Express.js?

Express is a **minimal, unopinionated web framework** for Node.js. It makes building web servers and APIs much easier than using the raw `http` module.

### Express vs Raw `http`:
| Feature | Raw `http` | Express |
|---------|-----------|---------|
| Routing | Manual `if/else` chains | `app.get()`, `app.post()`, etc. |
| Body parsing | Manual (read stream chunks) | `express.json()` middleware |
| Static files | Manual | `express.static()` |
| Middleware | None | Full middleware system |
| Code readability | Poor for large apps | Clean and organized |

---

## Setting Up

```bash
# Create project
mkdir my-express-app
cd my-express-app
npm init -y

# Install express
npm install express

# Install nodemon (auto-restart on file changes)
npm install nodemon --save-dev
```

Add a dev script to `package.json`:
```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  }
}
```

---

## Your First Express Server

```js
// server.js
const express = require('express');
const app = express();

const PORT = 3000;

// Define a route
app.get('/', (req, res) => {
  res.send('Hello World!');
});

// Start the server
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

```bash
# Run it
npm run dev    # with nodemon (auto-restart)
# OR
node server.js # without nodemon
```

### What's happening:
1. `express()` creates an Express application
2. `app.get('/', callback)` handles GET requests to `/`
3. `res.send()` sends a response to the client
4. `app.listen()` starts the server on port 3000

---

## HTTP Methods

```js
const express = require('express');
const app = express();

// Parse JSON request bodies
app.use(express.json());

// GET — Read/Retrieve data
app.get('/api/users', (req, res) => {
  res.json({ users: ['Aryan', 'Rahul', 'Priya'] });
});

// POST — Create new data
app.post('/api/users', (req, res) => {
  const { name, email } = req.body;
  res.status(201).json({ message: `User ${name} created` });
});

// PUT — Update (replace) entire resource
app.put('/api/users/:id', (req, res) => {
  const { id } = req.params;
  res.json({ message: `User ${id} fully updated` });
});

// PATCH — Update (partial) resource
app.patch('/api/users/:id', (req, res) => {
  const { id } = req.params;
  res.json({ message: `User ${id} partially updated` });
});

// DELETE — Remove data
app.delete('/api/users/:id', (req, res) => {
  const { id } = req.params;
  res.json({ message: `User ${id} deleted` });
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

### HTTP Methods ↔ CRUD Mapping:
```
GET    → READ    (fetch data)
POST   → CREATE  (submit new data)
PUT    → UPDATE  (replace entire resource)
PATCH  → UPDATE  (modify part of resource)
DELETE → DELETE  (remove data)
```

---

## Sending Responses

```js
// Send plain text
res.send('Hello World');

// Send JSON (most common for APIs)
res.json({ message: 'Success', data: users });

// Send with status code
res.status(201).json({ message: 'Created successfully' });

// Send status only (no body)
res.sendStatus(204);    // 204 No Content

// Redirect
res.redirect('/login');
res.redirect(301, '/new-page');  // permanent redirect

// Send a file
res.sendFile('/path/to/file.html');

// Download a file
res.download('/path/to/report.pdf');
```

### `res.send()` vs `res.json()` ⭐

```js
// res.send() — can send strings, objects, Buffers, arrays
res.send('Hello');                // Content-Type: text/html
res.send({ name: 'Aryan' });     // Content-Type: application/json
res.send(Buffer.from('hello'));   // Content-Type: application/octet-stream

// res.json() — ALWAYS sends JSON
res.json({ name: 'Aryan' });     // Content-Type: application/json
res.json(null);                   // sends "null"
res.json([1, 2, 3]);             // sends "[1,2,3]"
```

> **Best practice:** Use `res.json()` for API responses. It's explicit and always sets the correct content type.

---

## HTTP Status Codes You Must Know ⭐

```
2xx — SUCCESS
  200  OK                   → Request succeeded (default for GET)
  201  Created              → Resource created (use after POST)
  204  No Content           → Success but no body (use after DELETE)

3xx — REDIRECTION
  301  Moved Permanently    → Resource URL changed permanently
  302  Found                → Temporary redirect
  304  Not Modified         → Cached version is still valid

4xx — CLIENT ERROR
  400  Bad Request          → Invalid input/validation failed
  401  Unauthorized         → Not authenticated (no/invalid token)
  403  Forbidden            → Authenticated but not authorized (no permission)
  404  Not Found            → Resource doesn't exist
  409  Conflict             → Duplicate resource (e.g., email already exists)
  422  Unprocessable Entity → Validation error
  429  Too Many Requests    → Rate limit exceeded

5xx — SERVER ERROR
  500  Internal Server Error → Something broke on the server
  502  Bad Gateway           → Invalid response from upstream server
  503  Service Unavailable   → Server overloaded or in maintenance
```

---

## nodemon — Auto-Restart on Changes

Without nodemon, you have to manually stop and restart the server every time you change code.

```bash
# Install
npm install nodemon -D

# Use
npx nodemon server.js
# OR add to scripts:
# "dev": "nodemon server.js"
# Then: npm run dev
```

### nodemon.json (optional config):
```json
{
  "watch": ["src"],
  "ext": "js,json",
  "ignore": ["node_modules", "test"],
  "delay": "1000"
}
```

---

## Complete Example — Basic Express API

```js
const express = require('express');
const app = express();

// Middleware to parse JSON bodies
app.use(express.json());

// In-memory data store (temporary)
let todos = [
  { id: 1, title: 'Learn Node.js', completed: false },
  { id: 2, title: 'Learn Express', completed: false },
  { id: 3, title: 'Build an API', completed: false },
];

// GET all todos
app.get('/api/todos', (req, res) => {
  res.json({
    success: true,
    count: todos.length,
    data: todos,
  });
});

// GET single todo
app.get('/api/todos/:id', (req, res) => {
  const todo = todos.find(t => t.id === parseInt(req.params.id));
  
  if (!todo) {
    return res.status(404).json({
      success: false,
      message: 'Todo not found',
    });
  }
  
  res.json({ success: true, data: todo });
});

// POST create todo
app.post('/api/todos', (req, res) => {
  const { title } = req.body;
  
  if (!title) {
    return res.status(400).json({
      success: false,
      message: 'Title is required',
    });
  }
  
  const newTodo = {
    id: todos.length + 1,
    title,
    completed: false,
  };
  
  todos.push(newTodo);
  
  res.status(201).json({
    success: true,
    data: newTodo,
  });
});

// PUT update todo
app.put('/api/todos/:id', (req, res) => {
  const todo = todos.find(t => t.id === parseInt(req.params.id));
  
  if (!todo) {
    return res.status(404).json({
      success: false,
      message: 'Todo not found',
    });
  }
  
  todo.title = req.body.title || todo.title;
  todo.completed = req.body.completed ?? todo.completed;
  
  res.json({ success: true, data: todo });
});

// DELETE todo
app.delete('/api/todos/:id', (req, res) => {
  const index = todos.findIndex(t => t.id === parseInt(req.params.id));
  
  if (index === -1) {
    return res.status(404).json({
      success: false,
      message: 'Todo not found',
    });
  }
  
  todos.splice(index, 1);
  
  res.status(200).json({
    success: true,
    message: 'Todo deleted',
  });
});

// Start server
const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

---

## 🎯 Interview Answers

### Q: What is Express.js and why use it over plain `http` module?
> Express.js is a minimal, fast, and unopinionated web framework for Node.js. While you can create servers with the built-in `http` module, Express provides a robust routing system, middleware support, built-in body parsing, static file serving, and a cleaner API. It reduces boilerplate code significantly — for example, routing in raw `http` requires manual `if/else` chains, while Express provides declarative methods like `app.get()`, `app.post()`.

### Q: How do you create a basic Express server?
> Import express, call `express()` to create an app, define routes using `app.get()`, `app.post()`, etc., and start the server with `app.listen(port)`. You also typically add `express.json()` middleware to parse JSON request bodies.

### Q: What is the difference between `res.send()` and `res.json()`?
> `res.send()` can send various types of responses — strings, objects, Buffers, arrays. It auto-detects the content type. `res.json()` specifically sends JSON responses and always sets the `Content-Type` to `application/json`. For APIs, `res.json()` is preferred because it's explicit, and it also converts non-objects like `null` to valid JSON.

---

[⬅️ Module 3: npm](../03-npm-basics/README.md) | [Next: Routing ➡️](../05-routing/README.md)
