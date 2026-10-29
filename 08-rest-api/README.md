# Module 8: REST API Design

## What is REST?

**REST** = **RE**presentational **S**tate **T**ransfer. It's a set of rules (architecture style) for building web APIs.

A RESTful API uses HTTP methods to perform CRUD operations on **resources** (data entities like users, products, orders).

---

## REST Principles

1. **Client-Server** — Frontend and backend are separate
2. **Stateless** — Each request contains all info needed; server doesn't store session state
3. **Uniform Interface** — Consistent URL patterns and HTTP methods
4. **Resource-Based** — Everything is a resource identified by a URL
5. **Use HTTP Methods** — GET, POST, PUT, PATCH, DELETE for actions

---

## HTTP Methods ↔ CRUD Mapping ⭐

| HTTP Method | CRUD | Action | Example |
|------------|------|--------|---------|
| `GET` | Read | Fetch data | Get all users, get user by ID |
| `POST` | Create | Submit new data | Create a new user |
| `PUT` | Update | Replace entire resource | Update all user fields |
| `PATCH` | Update | Modify part of resource | Update only user's name |
| `DELETE` | Delete | Remove data | Delete a user |

### PUT vs PATCH ⭐ (Interview Favorite)

```js
// Current user in database:
// { id: 1, name: 'Aryan', email: 'aryan@test.com', age: 22 }

// PUT — Replace ENTIRE resource (send ALL fields)
// PUT /api/users/1
// Body: { name: 'Aryan Gupta', email: 'new@test.com', age: 23 }
// → All fields are required. Missing fields may be set to null/default.

// PATCH — Modify PART of resource (send ONLY changed fields)
// PATCH /api/users/1
// Body: { name: 'Aryan Gupta' }
// → Only name changes. email and age remain as they were.
```

| | PUT | PATCH |
|--|-----|-------|
| **Sends** | Entire resource (all fields) | Only changed fields |
| **Idempotent** | ✅ Yes (same result every time) | ✅ Usually yes |
| **Missing fields** | Set to null/default | Left unchanged |
| **Use when** | Replacing the whole thing | Updating one or two fields |

---

## Resource Naming Conventions ⭐

### Rules:
1. Use **nouns** (not verbs) — resources are things, not actions
2. Use **plural** nouns
3. Use **lowercase** and **hyphens** for multi-word

```
✅ GOOD (RESTful)                    ❌ BAD (Not RESTful)
GET  /api/users                      GET  /api/getUsers
GET  /api/users/42                   GET  /api/getUser?id=42
POST /api/users                      POST /api/createUser
PUT  /api/users/42                   POST /api/updateUser
DELETE /api/users/42                 POST /api/deleteUser

GET  /api/blog-posts                 GET  /api/blogPosts
GET  /api/users/42/orders            GET  /api/getUserOrders
```

### Nested Resources:
```
GET    /api/users/42/posts           → All posts by user 42
GET    /api/users/42/posts/7         → Post 7 by user 42
POST   /api/users/42/posts           → Create post for user 42
DELETE /api/users/42/posts/7         → Delete post 7 by user 42
```

### API Versioning:
```
/api/v1/users
/api/v2/users
```

---

## Complete CRUD API Example

```js
const express = require('express');
const app = express();

app.use(express.json());

// In-memory database (will use MongoDB later)
let users = [
  { id: 1, name: 'Aryan', email: 'aryan@test.com', role: 'admin' },
  { id: 2, name: 'Rahul', email: 'rahul@test.com', role: 'user' },
  { id: 3, name: 'Priya', email: 'priya@test.com', role: 'user' },
];
let nextId = 4;

// GET /api/users — Get all users (with filtering, sorting, pagination)
app.get('/api/users', (req, res) => {
  let result = [...users];
  
  // Filtering
  if (req.query.role) {
    result = result.filter(u => u.role === req.query.role);
  }
  
  // Searching
  if (req.query.search) {
    const search = req.query.search.toLowerCase();
    result = result.filter(u => 
      u.name.toLowerCase().includes(search) || 
      u.email.toLowerCase().includes(search)
    );
  }
  
  // Sorting
  if (req.query.sort) {
    const sortField = req.query.sort;
    const order = req.query.order === 'desc' ? -1 : 1;
    result.sort((a, b) => {
      if (a[sortField] < b[sortField]) return -1 * order;
      if (a[sortField] > b[sortField]) return 1 * order;
      return 0;
    });
  }
  
  // Pagination
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const startIndex = (page - 1) * limit;
  const endIndex = page * limit;
  const paginatedResult = result.slice(startIndex, endIndex);
  
  res.json({
    success: true,
    count: paginatedResult.length,
    total: result.length,
    page,
    totalPages: Math.ceil(result.length / limit),
    data: paginatedResult,
  });
});

// GET /api/users/:id — Get single user
app.get('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  
  if (!user) {
    return res.status(404).json({
      success: false,
      message: 'User not found',
    });
  }
  
  res.json({ success: true, data: user });
});

// POST /api/users — Create new user
app.post('/api/users', (req, res) => {
  const { name, email, role } = req.body;
  
  // Validation
  if (!name || !email) {
    return res.status(400).json({
      success: false,
      message: 'Name and email are required',
    });
  }
  
  // Check duplicate email
  if (users.find(u => u.email === email)) {
    return res.status(409).json({
      success: false,
      message: 'Email already exists',
    });
  }
  
  const newUser = {
    id: nextId++,
    name,
    email,
    role: role || 'user',
  };
  
  users.push(newUser);
  
  res.status(201).json({
    success: true,
    message: 'User created successfully',
    data: newUser,
  });
});

// PUT /api/users/:id — Full update
app.put('/api/users/:id', (req, res) => {
  const index = users.findIndex(u => u.id === parseInt(req.params.id));
  
  if (index === -1) {
    return res.status(404).json({
      success: false,
      message: 'User not found',
    });
  }
  
  const { name, email, role } = req.body;
  
  // PUT requires ALL fields
  if (!name || !email) {
    return res.status(400).json({
      success: false,
      message: 'All fields are required for PUT',
    });
  }
  
  users[index] = {
    id: users[index].id,
    name,
    email,
    role: role || 'user',
  };
  
  res.json({ success: true, data: users[index] });
});

// PATCH /api/users/:id — Partial update
app.patch('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  
  if (!user) {
    return res.status(404).json({
      success: false,
      message: 'User not found',
    });
  }
  
  // PATCH — only update provided fields
  if (req.body.name) user.name = req.body.name;
  if (req.body.email) user.email = req.body.email;
  if (req.body.role) user.role = req.body.role;
  
  res.json({ success: true, data: user });
});

// DELETE /api/users/:id — Delete user
app.delete('/api/users/:id', (req, res) => {
  const index = users.findIndex(u => u.id === parseInt(req.params.id));
  
  if (index === -1) {
    return res.status(404).json({
      success: false,
      message: 'User not found',
    });
  }
  
  users.splice(index, 1);
  
  res.json({
    success: true,
    message: 'User deleted successfully',
  });
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

### Testing with curl:
```bash
# Get all users
curl http://localhost:3000/api/users

# Get users filtered by role
curl http://localhost:3000/api/users?role=admin

# Get users with pagination
curl "http://localhost:3000/api/users?page=1&limit=2"

# Get single user
curl http://localhost:3000/api/users/1

# Create user
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Amit","email":"amit@test.com"}'

# Full update
curl -X PUT http://localhost:3000/api/users/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"Aryan G","email":"aryan@new.com","role":"admin"}'

# Partial update
curl -X PATCH http://localhost:3000/api/users/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"Aryan Gupta"}'

# Delete user
curl -X DELETE http://localhost:3000/api/users/1
```

---

## Pagination Design ⭐

```js
// Query: GET /api/users?page=2&limit=10

const page = parseInt(req.query.page) || 1;
const limit = parseInt(req.query.limit) || 10;
const skip = (page - 1) * limit;

// With MongoDB/Mongoose:
const users = await User.find()
  .skip(skip)
  .limit(limit)
  .sort({ createdAt: -1 });

const total = await User.countDocuments();

res.json({
  success: true,
  data: users,
  pagination: {
    currentPage: page,
    totalPages: Math.ceil(total / limit),
    totalItems: total,
    itemsPerPage: limit,
    hasNextPage: page < Math.ceil(total / limit),
    hasPrevPage: page > 1,
  },
});
```

---

## 🎯 Interview Answers

### Q: What is a RESTful API?
> A RESTful API is a web API that follows REST architecture principles. It uses standard HTTP methods (GET, POST, PUT, PATCH, DELETE) to perform CRUD operations on resources identified by URLs. It's stateless (each request is independent), uses a uniform interface with consistent URL patterns (plural nouns like `/api/users`), and returns appropriate HTTP status codes.

### Q: Difference between PUT and PATCH?
> PUT replaces the entire resource — you must send all fields, and any missing fields may be set to null or defaults. PATCH modifies only the specified fields — you send only what changed, and everything else stays the same. For example, to change just a user's name, PATCH sends `{ name: "new name" }`, while PUT would require `{ name: "new name", email: "...", role: "..." }`.

### Q: How do you design pagination in a REST API?
> Use query parameters `page` and `limit` — e.g., `GET /api/users?page=2&limit=10`. Calculate `skip = (page - 1) * limit` to offset the database query. Return pagination metadata in the response: current page, total pages, total items, and whether there are next/previous pages. With Mongoose, use `.skip()` and `.limit()` methods.

### Q: What makes an API RESTful?
> An API is RESTful when it follows these principles: uses HTTP methods correctly (GET for reading, POST for creating, etc.), resources are identified by URLs with nouns (not verbs), is stateless (no server-side sessions), uses proper status codes, and has a uniform, predictable interface. For example, `/api/users` not `/api/getUsers`.

---

[⬅️ Module 7: Req & Res](../07-req-res/README.md) | [Next: Error Handling ➡️](../09-error-handling/README.md)
