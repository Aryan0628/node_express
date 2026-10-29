# Module 9: Error Handling

## Why Proper Error Handling Matters

Without error handling, your server crashes or sends unhelpful error messages. In production, unhandled errors can:
- Expose sensitive information (stack traces, DB queries)
- Crash the entire server
- Leave clients hanging with no response

---

## Basic try-catch with async/await

```js
app.get('/api/users/:id', async (req, res) => {
  try {
    const user = await User.findById(req.params.id);
    
    if (!user) {
      return res.status(404).json({ message: 'User not found' });
    }
    
    res.json({ success: true, data: user });
  } catch (err) {
    console.error(err);
    res.status(500).json({ message: 'Server error' });
  }
});
```

> Problem: Writing try-catch in **every single route** is repetitive.

---

## Custom Error Class ⭐

Create a reusable error class:

```js
// utils/AppError.js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith('4') ? 'fail' : 'error';
    this.isOperational = true;  // Expected errors we can handle
    
    Error.captureStackTrace(this, this.constructor);
  }
}

module.exports = AppError;
```

### Usage:
```js
const AppError = require('./utils/AppError');

app.get('/api/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    
    if (!user) {
      throw new AppError('User not found', 404);
      // OR: return next(new AppError('User not found', 404));
    }
    
    res.json({ success: true, data: user });
  } catch (err) {
    next(err);  // Pass error to error-handling middleware
  }
});
```

---

## Centralized Error-Handling Middleware ⭐

Instead of handling errors in every route, create ONE error handler:

```js
// middleware/errorHandler.js
const errorHandler = (err, req, res, next) => {
  // Default values
  err.statusCode = err.statusCode || 500;
  err.message = err.message || 'Internal Server Error';
  
  // Log error (in development)
  if (process.env.NODE_ENV === 'development') {
    console.error('ERROR 💥:', err);
    
    return res.status(err.statusCode).json({
      success: false,
      message: err.message,
      stack: err.stack,         // Show stack trace in development
      error: err,
    });
  }
  
  // Production — don't leak error details
  // Operational errors: send message to client
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      success: false,
      message: err.message,
    });
  }
  
  // Programming errors: send generic message
  console.error('ERROR 💥:', err);
  res.status(500).json({
    success: false,
    message: 'Something went wrong',
  });
};

module.exports = errorHandler;
```

### Setup in app:
```js
const errorHandler = require('./middleware/errorHandler');

// All routes above...
app.use('/api/users', userRoutes);

// Error handler (MUST be LAST middleware)
app.use(errorHandler);
```

---

## Handling Async Errors (No more try-catch everywhere!) ⭐

### Method 1: Wrapper function (catchAsync)

```js
// utils/catchAsync.js
const catchAsync = (fn) => {
  return (req, res, next) => {
    fn(req, res, next).catch(next);
    // .catch(next) automatically passes errors to error middleware
  };
};

module.exports = catchAsync;
```

### Usage — Clean routes with NO try-catch:

```js
const catchAsync = require('./utils/catchAsync');
const AppError = require('./utils/AppError');

// ❌ BEFORE — try-catch in every route
app.get('/api/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) return next(new AppError('Not found', 404));
    res.json(user);
  } catch (err) {
    next(err);
  }
});

// ✅ AFTER — clean with catchAsync
app.get('/api/users/:id', catchAsync(async (req, res, next) => {
  const user = await User.findById(req.params.id);
  if (!user) throw new AppError('User not found', 404);
  res.json({ success: true, data: user });
}));

// If any await rejects, catchAsync catches it and passes to error middleware!
```

### Method 2: express-async-errors package

```bash
npm install express-async-errors
```

```js
// Just require it at the top — that's it!
require('express-async-errors');
const express = require('express');
const app = express();

// Now async errors are automatically caught!
app.get('/api/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);  // If this throws, it's caught
  if (!user) throw new AppError('Not found', 404);   // This too
  res.json(user);
});

// Error handler still needed
app.use(errorHandler);
```

---

## Operational vs Programming Errors

| | Operational Errors | Programming Errors |
|--|-------------------|-------------------|
| **What** | Expected, can happen in production | Bugs in code |
| **Examples** | Invalid input, DB down, auth failed | TypeError, undefined variable |
| **Handle** | Send meaningful error to client | Fix the code, log & restart |
| **isOperational** | `true` | `false` |

```js
// Operational — expected
throw new AppError('Email already exists', 409);
throw new AppError('Invalid password', 401);
throw new AppError('Product not found', 404);

// Programming — bugs (unexpected)
// undefined.toString()  → TypeError
// JSON.parse('invalid') → SyntaxError
// These should crash the app (in production, use a process manager to restart)
```

---

## Handling Specific Mongoose/MongoDB Errors

```js
const errorHandler = (err, req, res, next) => {
  let error = { ...err, message: err.message };
  
  // Mongoose bad ObjectId (CastError)
  // e.g., GET /api/users/invalidId
  if (err.name === 'CastError') {
    error = new AppError(`Invalid ${err.path}: ${err.value}`, 400);
  }
  
  // Mongoose duplicate key (11000)
  // e.g., POST /api/users with existing email
  if (err.code === 11000) {
    const field = Object.keys(err.keyValue)[0];
    error = new AppError(`Duplicate value for field: ${field}`, 409);
  }
  
  // Mongoose validation error
  // e.g., POST /api/users without required fields
  if (err.name === 'ValidationError') {
    const messages = Object.values(err.errors).map(e => e.message);
    error = new AppError(`Validation failed: ${messages.join(', ')}`, 400);
  }
  
  // JWT errors
  if (err.name === 'JsonWebTokenError') {
    error = new AppError('Invalid token. Please log in again.', 401);
  }
  if (err.name === 'TokenExpiredError') {
    error = new AppError('Token expired. Please log in again.', 401);
  }
  
  res.status(error.statusCode || 500).json({
    success: false,
    message: error.message || 'Server Error',
  });
};
```

---

## Global Process Error Handlers

Catch errors that slip through:

```js
// server.js

// Unhandled Promise Rejections (e.g., DB connection fails)
process.on('unhandledRejection', (err) => {
  console.error('UNHANDLED REJECTION 💥:', err.message);
  // Graceful shutdown
  server.close(() => {
    process.exit(1);
  });
});

// Uncaught Exceptions (synchronous code bugs)
process.on('uncaughtException', (err) => {
  console.error('UNCAUGHT EXCEPTION 💥:', err.message);
  process.exit(1);  // Must exit — app is in undefined state
});
```

---

## Complete Error Handling Setup

```
project/
├── utils/
│   ├── AppError.js         # Custom error class
│   └── catchAsync.js       # Async error wrapper
├── middleware/
│   └── errorHandler.js     # Centralized error handler
├── routes/
│   └── userRoutes.js       # Clean routes (no try-catch)
└── server.js               # Global error handlers
```

---

## 🎯 Interview Answers

### Q: How do you handle errors in Express?
> I use a centralized error-handling approach. I create a custom `AppError` class for known errors, wrap async route handlers with a `catchAsync` function to avoid repetitive try-catch blocks, and define a centralized error-handling middleware with 4 parameters `(err, req, res, next)` at the end of all routes. This middleware formats the error response differently for development (with stack trace) and production (clean message only).

### Q: What is centralized error handling?
> Instead of handling errors individually in every route with try-catch, you define a single error-handling middleware `(err, req, res, next)` that catches all errors. Routes pass errors using `next(error)`, and the centralized handler formats the response, logs the error, and sends an appropriate status code. This keeps routes clean and ensures consistent error responses.

### Q: How do you handle async errors in Express?
> Three approaches: (1) Wrap every route handler in try-catch and call `next(err)` in the catch block — works but verbose. (2) Create a `catchAsync` wrapper function that takes an async function and returns a new function that catches any promise rejection and passes it to `next()`. (3) Use the `express-async-errors` package which monkey-patches Express to automatically catch async errors. All approaches ultimately pass errors to the centralized error-handling middleware.

---

[⬅️ Module 8: REST API](../08-rest-api/README.md) | [Next: MongoDB + Mongoose ➡️](../10-mongodb-mongoose/README.md)
