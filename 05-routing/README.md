# Module 5: Routing in Express

## What is Routing?

Routing determines **how your app responds to a client request** at a specific URL (path) and HTTP method.

```
Route = HTTP Method + URL Path + Handler Function
```

---

## Basic Routing

```js
const express = require('express');
const app = express();

// app.METHOD(PATH, HANDLER)
app.get('/', (req, res) => {
  res.send('Home Page');
});

app.get('/about', (req, res) => {
  res.send('About Page');
});

app.post('/contact', (req, res) => {
  res.send('Form submitted');
});
```

---

## Route Parameters — `req.params` ⭐

Route parameters are named **URL segments** used to capture values at specific positions.

```js
// :id is a route parameter
app.get('/api/users/:id', (req, res) => {
  console.log(req.params);    // { id: '42' }
  console.log(req.params.id); // '42' (always a string!)
  
  res.json({ userId: req.params.id });
});

// Multiple parameters
app.get('/api/users/:userId/posts/:postId', (req, res) => {
  console.log(req.params);
  // { userId: '5', postId: '12' }
  
  res.json({
    user: req.params.userId,
    post: req.params.postId,
  });
});

// Example requests:
// GET /api/users/42          → { id: '42' }
// GET /api/users/42/posts/7  → { userId: '42', postId: '7' }
```

> ⚠️ **`req.params` values are always strings!** Convert to number with `parseInt()` or `Number()` when needed.

---

## Query Strings — `req.query` ⭐

Query strings are **key-value pairs** in the URL after `?`. Used for filtering, searching, pagination.

```js
app.get('/api/products', (req, res) => {
  console.log(req.query);
  // For URL: /api/products?category=electronics&sort=price&page=2
  // { category: 'electronics', sort: 'price', page: '2' }
  
  const { category, sort, page } = req.query;
  
  res.json({
    category: category || 'all',
    sort: sort || 'default',
    page: parseInt(page) || 1,
  });
});

// /api/search?q=node+js&limit=10
app.get('/api/search', (req, res) => {
  const { q, limit } = req.query;
  res.json({ searchTerm: q, limit: parseInt(limit) || 10 });
});
```

---

## `req.params` vs `req.query` ⭐ (Interview Favorite)

```
URL: /api/users/42?role=admin&active=true
       └──────┘    └──────────────────┘
       req.params       req.query
       { id: '42' }     { role: 'admin', active: 'true' }
```

| Feature | `req.params` | `req.query` |
|---------|-------------|-------------|
| **Where** | In the URL path (`/users/:id`) | After `?` in URL (`?key=value`) |
| **Purpose** | Identify a specific resource | Filter, sort, paginate |
| **Required** | Yes (route won't match without it) | No (optional) |
| **Example URL** | `/users/42` | `/users?role=admin` |
| **Defined by** | Route definition (`:id`) | Client sends it |

### When to use which:
```
req.params → "Give me user WITH THIS ID"
  GET /api/users/:id         → get specific user
  DELETE /api/users/:id      → delete specific user

req.query → "Give me users MATCHING THESE FILTERS"
  GET /api/users?role=admin  → filter by role
  GET /api/users?page=2      → pagination
  GET /api/users?sort=name   → sorting
```

---

## `express.Router()` — Modular Routing ⭐

As your app grows, putting all routes in one file becomes messy. `Router` lets you split routes into separate files.

### Without Router (messy for big apps):
```js
// server.js — everything in one file
app.get('/api/users', ...);
app.post('/api/users', ...);
app.get('/api/users/:id', ...);
app.get('/api/products', ...);
app.post('/api/products', ...);
app.get('/api/orders', ...);
// ... 50 more routes
```

### With Router (clean, modular):

```js
// routes/userRoutes.js
const express = require('express');
const router = express.Router();

// Note: paths are relative to where the router is mounted
router.get('/', (req, res) => {
  res.json({ message: 'Get all users' });
});

router.get('/:id', (req, res) => {
  res.json({ message: `Get user ${req.params.id}` });
});

router.post('/', (req, res) => {
  res.status(201).json({ message: 'User created' });
});

router.put('/:id', (req, res) => {
  res.json({ message: `User ${req.params.id} updated` });
});

router.delete('/:id', (req, res) => {
  res.json({ message: `User ${req.params.id} deleted` });
});

module.exports = router;
```

```js
// routes/productRoutes.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json({ message: 'Get all products' });
});

router.get('/:id', (req, res) => {
  res.json({ message: `Get product ${req.params.id}` });
});

module.exports = router;
```

```js
// server.js — clean and organized
const express = require('express');
const app = express();

const userRoutes = require('./routes/userRoutes');
const productRoutes = require('./routes/productRoutes');

app.use(express.json());

// Mount routers at specific paths
app.use('/api/users', userRoutes);       // All user routes start with /api/users
app.use('/api/products', productRoutes); // All product routes start with /api/products

app.listen(3000, () => console.log('Server running on port 3000'));
```

### How it maps:
```
router.get('/')     + mounted at '/api/users'  → GET /api/users
router.get('/:id')  + mounted at '/api/users'  → GET /api/users/:id
router.post('/')    + mounted at '/api/users'  → POST /api/users
```

---

## Route Chaining — `app.route()`

When multiple HTTP methods share the same path:

```js
// Instead of this:
app.get('/api/books', getBooks);
app.post('/api/books', createBook);

// You can chain:
app.route('/api/books')
  .get((req, res) => {
    res.json({ message: 'Get all books' });
  })
  .post((req, res) => {
    res.status(201).json({ message: 'Book created' });
  });

app.route('/api/books/:id')
  .get((req, res) => {
    res.json({ message: `Get book ${req.params.id}` });
  })
  .put((req, res) => {
    res.json({ message: `Book ${req.params.id} updated` });
  })
  .delete((req, res) => {
    res.json({ message: `Book ${req.params.id} deleted` });
  });
```

---

## 404 Handling — Catch-All Route

Place this **after all other routes**:

```js
// All your routes above...
app.get('/api/users', ...);
app.get('/api/products', ...);

// 404 — No route matched (MUST be LAST)
app.use((req, res) => {
  res.status(404).json({
    success: false,
    message: `Route ${req.originalUrl} not found`,
  });
});

// OR with a wildcard
app.all('*', (req, res) => {
  res.status(404).json({
    success: false,
    message: `Cannot ${req.method} ${req.originalUrl}`,
  });
});
```

> `app.all('*')` catches ANY method on ANY unmatched path.

---

## Route Order Matters!

Express matches routes **top to bottom** and stops at the first match.

```js
// ❌ WRONG ORDER — '/api/users/profile' will never be reached
app.get('/api/users/:id', (req, res) => {
  // This catches '/api/users/profile' too!
  // req.params.id would be 'profile'
  res.json({ id: req.params.id });
});

app.get('/api/users/profile', (req, res) => {
  // Never reached!
  res.json({ message: 'User profile' });
});

// ✅ CORRECT ORDER — specific routes first
app.get('/api/users/profile', (req, res) => {
  res.json({ message: 'User profile' });
});

app.get('/api/users/:id', (req, res) => {
  res.json({ id: req.params.id });
});
```

> **Rule:** Put **specific routes before parameterized routes**.

---

## 🎯 Interview Answers

### Q: What is the difference between `req.params` and `req.query`?
> `req.params` captures named route parameters defined in the URL path (like `/users/:id`), used to identify specific resources. `req.query` captures query string parameters after `?` in the URL (like `?page=2&sort=name`), used for filtering, sorting, and pagination. Params are required for the route to match, while query strings are optional.

### Q: How do you create modular routes using `express.Router()`?
> `express.Router()` creates a mini-app that handles routes for a specific resource. You define routes in separate files using `router.get()`, `router.post()`, etc., export the router, and mount it in the main app using `app.use('/prefix', router)`. This keeps code organized — e.g., all user routes in `userRoutes.js`, product routes in `productRoutes.js`.

### Q: How do you handle 404 (Not Found) in Express?
> Add a catch-all middleware or route at the very end of all route definitions using `app.use()` or `app.all('*')` that sends a 404 status with an error message. Since Express matches routes top-to-bottom, if no previous route matched, this final handler catches the request.

---

[⬅️ Module 4: Express Basics](../04-express-basics/README.md) | [Next: Middleware ➡️](../06-middleware/README.md)
