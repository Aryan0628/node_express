# Module 6: Middleware ⭐ (Most Important Express Concept)

## What is Middleware?

Middleware functions are functions that have access to the **request object** (`req`), **response object** (`res`), and the **next function** (`next`) in the request-response cycle.

Think of middleware as **checkpoints** a request passes through before reaching the route handler.

```
Client Request → Middleware 1 → Middleware 2 → Route Handler → Response
                   (logging)     (auth check)    (business logic)
```

### Every middleware can:
1. **Execute code** (do something)
2. **Modify** `req` and `res` objects
3. **End the request-response cycle** (send a response)
4. **Call `next()`** to pass control to the next middleware

```js
// Basic middleware structure
function myMiddleware(req, res, next) {
  // Do something...
  console.log('Middleware executed');
  
  next();  // Pass control to the next middleware/route handler
}
```

---

## `next()` — The Key to Middleware ⭐

```js
app.use((req, res, next) => {
  console.log('Middleware 1');
  next();  // ✅ Passes to next middleware
});

app.use((req, res, next) => {
  console.log('Middleware 2');
  next();  // ✅ Passes to route handler
});

app.get('/', (req, res) => {
  res.send('Home');  // Request ends here
});
```

### What happens if you DON'T call `next()`?

```js
app.use((req, res, next) => {
  console.log('Middleware runs');
  // ❌ No next() and no res.send()
  // Request HANGS! Client waits forever until timeout.
});

app.get('/', (req, res) => {
  res.send('This NEVER runs');  // Never reached!
});
```

> **Rule:** Every middleware must either call `next()` or send a response. Otherwise the request hangs.

---

## Types of Middleware

### 1. Application-Level Middleware — `app.use()`

Applied to **every request** (or specific paths).

```js
// Runs on EVERY request
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
  next();
});

// Runs only on requests to /api/*
app.use('/api', (req, res, next) => {
  console.log('API request received');
  next();
});
```

### 2. Router-Level Middleware

Applied to a specific router.

```js
const router = express.Router();

// Runs for all routes in this router
router.use((req, res, next) => {
  console.log('User router middleware');
  next();
});

router.get('/', (req, res) => {
  res.json({ users: [] });
});

app.use('/api/users', router);
```

### 3. Built-in Middleware

Express comes with these:

```js
// Parse JSON request bodies
app.use(express.json());
// Without this, req.body is undefined for JSON requests!

// Parse URL-encoded form data
app.use(express.urlencoded({ extended: true }));
// For form submissions: name=Aryan&email=aryan@test.com

// Serve static files (images, CSS, JS)
app.use(express.static('public'));
// Files in /public folder are accessible directly:
// /public/style.css → http://localhost:3000/style.css
```

### 4. Third-Party Middleware

```bash
npm install cors morgan helmet cookie-parser
```

```js
const cors = require('cors');
const morgan = require('morgan');
const helmet = require('helmet');
const cookieParser = require('cookie-parser');

app.use(cors());                // Enable CORS
app.use(morgan('dev'));         // HTTP request logging
app.use(helmet());             // Security headers
app.use(cookieParser());       // Parse cookies

// morgan output example:
// GET /api/users 200 5.432 ms - 52
```

### 5. Custom Middleware

```js
// Logger middleware
const logger = (req, res, next) => {
  const timestamp = new Date().toISOString();
  console.log(`[${timestamp}] ${req.method} ${req.url}`);
  next();
};

// Auth middleware
const authenticate = (req, res, next) => {
  const token = req.headers.authorization;
  
  if (!token) {
    return res.status(401).json({ message: 'No token provided' });
    // Note: no next() call — request ends here
  }
  
  try {
    // Verify token (we'll learn JWT later)
    const decoded = jwt.verify(token.split(' ')[1], 'secret');
    req.user = decoded;  // Attach user data to request
    next();              // Proceed to route handler
  } catch (err) {
    return res.status(401).json({ message: 'Invalid token' });
  }
};

// Use globally
app.use(logger);

// Use on specific routes
app.get('/api/profile', authenticate, (req, res) => {
  // Only runs if authenticate calls next()
  res.json({ user: req.user });
});

// Use on all routes under a path
app.use('/api/admin', authenticate, adminRoutes);
```

### 6. Error-Handling Middleware ⭐

Error middleware has **4 parameters** — that's how Express identifies it.

```js
// Regular middleware: (req, res, next) — 3 params
// Error middleware: (err, req, res, next) — 4 params!

// Triggering an error
app.get('/api/error', (req, res, next) => {
  const error = new Error('Something went wrong');
  error.statusCode = 500;
  next(error);  // Pass error to error-handling middleware
});

// Error-handling middleware (MUST have 4 parameters)
app.use((err, req, res, next) => {
  console.error(err.stack);
  
  const statusCode = err.statusCode || 500;
  res.status(statusCode).json({
    success: false,
    message: err.message || 'Internal Server Error',
  });
});
```

> **Must be defined AFTER all routes and other middleware!**

---

## Middleware Order Matters! ⭐

Express executes middleware **in the order they are defined**.

```js
// ✅ CORRECT ORDER
app.use(express.json());           // 1. Parse body first
app.use(morgan('dev'));            // 2. Log requests
app.use(cors());                   // 3. Handle CORS
app.use('/api/users', userRoutes); // 4. Routes
app.use(errorHandler);             // 5. Error handler LAST

// ❌ WRONG ORDER
app.use('/api/users', userRoutes); // Routes first
app.use(express.json());           // Body parser AFTER routes
// → req.body will be undefined in routes!
```

### Common order:
```
1. express.json()          → Parse request bodies
2. express.urlencoded()    → Parse form data
3. cors()                  → Handle CORS
4. morgan()                → Logging
5. helmet()                → Security headers
6. express.static()        → Serve static files
7. Custom middleware        → Auth, logging, etc.
8. Routes                  → Your API routes
9. 404 handler             → Catch unmatched routes
10. Error handler          → Catch all errors
```

---

## `app.use()` vs `app.get()` ⭐

```js
// app.use() — matches ALL HTTP methods, partial path matching
app.use('/api', handler);
// Matches: GET /api, POST /api, GET /api/users, GET /api/anything
// → /api is a PREFIX match

// app.get() — matches ONLY GET method, exact path matching
app.get('/api', handler);
// Matches: GET /api ONLY
// Does NOT match: POST /api, GET /api/users
```

| Feature | `app.use()` | `app.get()` |
|---------|------------|------------|
| HTTP methods | ALL methods | GET only |
| Path matching | **Prefix** match | **Exact** match |
| Use case | Middleware, mounting routers | Route handlers |
| `app.use('/api')` | matches `/api/anything` | matches only `/api` |

---

## Multiple Middleware Per Route

You can stack middleware on a single route:

```js
const isLoggedIn = (req, res, next) => {
  if (!req.headers.authorization) {
    return res.status(401).json({ message: 'Login required' });
  }
  next();
};

const isAdmin = (req, res, next) => {
  if (req.user.role !== 'admin') {
    return res.status(403).json({ message: 'Admin access required' });
  }
  next();
};

// Route with multiple middleware
app.delete('/api/users/:id', isLoggedIn, isAdmin, (req, res) => {
  // Only runs if BOTH isLoggedIn and isAdmin call next()
  res.json({ message: 'User deleted' });
});

// Flow: isLoggedIn → isAdmin → route handler
```

---

## Real-World Middleware Example

```js
const express = require('express');
const app = express();

// 1. Body parsing
app.use(express.json());

// 2. Custom request logger
app.use((req, res, next) => {
  req.requestTime = Date.now();
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next();
});

// 3. API Key validation (for specific routes)
const validateApiKey = (req, res, next) => {
  const apiKey = req.headers['x-api-key'];
  
  if (apiKey !== 'my-secret-api-key') {
    return res.status(401).json({ message: 'Invalid API key' });
  }
  
  next();
};

// 4. Response time tracker
app.use((req, res, next) => {
  const start = Date.now();
  
  // Override res.json to add timing info
  const originalJson = res.json.bind(res);
  res.json = (data) => {
    data.responseTime = `${Date.now() - start}ms`;
    return originalJson(data);
  };
  
  next();
});

// 5. Public route (no API key needed)
app.get('/api/public', (req, res) => {
  res.json({ message: 'Public data' });
});

// 6. Protected route (API key required)
app.get('/api/private', validateApiKey, (req, res) => {
  res.json({ message: 'Secret data' });
});

// 7. 404 Handler
app.use((req, res) => {
  res.status(404).json({ message: 'Route not found' });
});

// 8. Error Handler (MUST have 4 params)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ message: 'Something broke!' });
});

app.listen(3000);
```

---

## 🎯 Interview Answers

### Q: What is middleware in Express? Give examples.
> Middleware functions are functions that execute during the request-response cycle, with access to `req`, `res`, and `next`. They can execute code, modify request/response objects, end the cycle, or call `next()` to pass control. Examples: `express.json()` for body parsing, `cors()` for cross-origin requests, `morgan()` for logging, and custom middleware for authentication.

### Q: What is `next()` and what happens if you don't call it?
> `next()` is a function that passes control to the next middleware in the stack. If you don't call `next()` and don't send a response, the request will hang indefinitely — the client keeps waiting until it times out. Every middleware must either call `next()` to continue the chain or send a response to end it.

### Q: What is the difference between `app.use()` and `app.get()`?
> `app.use()` matches ALL HTTP methods and does prefix matching on paths — `app.use('/api')` matches `/api/users`, `/api/anything`. It's used for middleware and mounting routers. `app.get()` matches only GET requests and does exact path matching — it's used for defining specific route handlers.

### Q: How does error-handling middleware differ from regular middleware?
> Error-handling middleware has **4 parameters** `(err, req, res, next)` instead of 3. Express identifies it as an error handler because of the 4 parameters. It's triggered when `next(error)` is called or when an error is thrown. It should be defined after all routes and other middleware.

### Q: Why does the order of middleware matter?
> Express executes middleware in the order they are defined. If you put body parsing after your routes, `req.body` will be undefined. If you put the error handler before your routes, it won't catch route errors. The general order should be: body parsing → logging → security → routes → 404 handler → error handler.

---

[⬅️ Module 5: Routing](../05-routing/README.md) | [Next: Request & Response ➡️](../07-req-res/README.md)
