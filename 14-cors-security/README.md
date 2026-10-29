# Module 14: CORS & Security Basics

## What is CORS? ⭐

**CORS** = Cross-Origin Resource Sharing

### The Problem — Same-Origin Policy:
Browsers block requests from one origin to a different origin for security.

```
Origin = Protocol + Domain + Port

http://localhost:3000    →  origin 1 (React frontend)
http://localhost:5000    →  origin 2 (Express backend)

React (port 3000) tries to fetch from Express (port 5000)
→ BLOCKED by browser! ❌ "CORS error"
```

### The Solution:
The server must explicitly allow other origins to access its resources by setting CORS headers.

```bash
npm install cors
```

```js
const cors = require('cors');

// Allow ALL origins (good for development)
app.use(cors());

// Allow specific origins (production)
app.use(cors({
  origin: 'http://localhost:3000',  // Only your React app
}));

// Multiple origins
app.use(cors({
  origin: ['http://localhost:3000', 'https://myapp.com'],
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  credentials: true,  // Allow cookies
}));

// Dynamic origin
app.use(cors({
  origin: (origin, callback) => {
    const allowedOrigins = [
      'http://localhost:3000',
      'https://myapp.com',
    ];
    
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
}));
```

### What CORS headers look like:
```
Access-Control-Allow-Origin: http://localhost:3000
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
```

---

## Helmet — Security Headers

```bash
npm install helmet
```

```js
const helmet = require('helmet');

app.use(helmet());
// Sets 15+ security headers automatically:
// X-Content-Type-Options: nosniff
// X-Frame-Options: SAMEORIGIN
// X-XSS-Protection: 1; mode=block
// Strict-Transport-Security: max-age=...
// etc.
```

---

## Rate Limiting

Prevents abuse (brute force attacks, DDoS).

```bash
npm install express-rate-limit
```

```js
const rateLimit = require('express-rate-limit');

// General limiter
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                    // 100 requests per window
  message: {
    success: false,
    message: 'Too many requests, please try again after 15 minutes',
  },
  standardHeaders: true,
  legacyHeaders: false,
});

app.use(limiter);  // Apply to all routes

// Stricter limiter for auth routes
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,  // Only 5 login attempts per 15 min
  message: { message: 'Too many login attempts' },
});

app.use('/api/auth/login', authLimiter);
```

---

## Input Validation

### With express-validator:
```bash
npm install express-validator
```

```js
const { body, validationResult } = require('express-validator');

app.post('/api/users', [
  body('name').trim().notEmpty().withMessage('Name is required'),
  body('email').isEmail().normalizeEmail().withMessage('Valid email required'),
  body('password').isLength({ min: 6 }).withMessage('Password must be 6+ chars'),
  body('age').optional().isInt({ min: 0, max: 150 }),
], (req, res) => {
  const errors = validationResult(req);
  
  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      errors: errors.array(),
    });
  }
  
  // Input is valid, proceed...
  res.json({ message: 'User created' });
});
```

---

## Common Security Vulnerabilities

### XSS (Cross-Site Scripting)
Attacker injects malicious script into your app.

```js
// ❌ Vulnerable: rendering user input as HTML
res.send(`<h1>Hello ${req.query.name}</h1>`);
// URL: /greet?name=<script>steal_cookies()</script>

// ✅ Prevent: sanitize input, use template engines that auto-escape
```

### NoSQL Injection
```js
// ❌ Vulnerable
app.post('/login', async (req, res) => {
  const user = await User.findOne({
    email: req.body.email,
    password: req.body.password,
  });
  // Attacker sends: { "email": {"$gt": ""}, "password": {"$gt": ""} }
  // This matches ALL users!
});

// ✅ Prevent: validate types, use mongoose (it casts types)
npm install express-mongo-sanitize
```

```js
const mongoSanitize = require('express-mongo-sanitize');
app.use(mongoSanitize());  // Removes $ and . from req.body/params/query
```

### CSRF (Cross-Site Request Forgery)
Tricks a logged-in user into making unintended requests.

Prevention: use `sameSite: 'strict'` on cookies, CSRF tokens.

---

## Security Middleware Checklist

```js
const express = require('express');
const helmet = require('helmet');
const cors = require('cors');
const rateLimit = require('express-rate-limit');
const mongoSanitize = require('express-mongo-sanitize');

const app = express();

// 1. Security headers
app.use(helmet());

// 2. CORS
app.use(cors({ origin: process.env.CLIENT_URL, credentials: true }));

// 3. Rate limiting
app.use(rateLimit({ windowMs: 15 * 60 * 1000, max: 100 }));

// 4. Body parsing with size limit
app.use(express.json({ limit: '10kb' }));  // Prevent large payload attacks

// 5. Data sanitization
app.use(mongoSanitize());

// 6. Your routes...
```

---

## 🎯 Interview Answers

### Q: What is CORS and how do you handle it?
> CORS (Cross-Origin Resource Sharing) is a browser security mechanism that blocks requests between different origins. In MERN, React (port 3000) calling Express (port 5000) triggers CORS errors. To fix it, install the `cors` package and use `app.use(cors())` in Express. For production, specify allowed origins: `cors({ origin: 'https://myapp.com' })`. CORS headers tell the browser which origins, methods, and headers are allowed.

### Q: How do you prevent common security vulnerabilities in Express?
> Use `helmet` for security headers, `cors` for controlled cross-origin access, `express-rate-limit` to prevent brute force and DDoS, `express-mongo-sanitize` to prevent NoSQL injection, limit request body size with `express.json({ limit: '10kb' })`, validate all input with `express-validator`, and hash passwords with `bcrypt`. Never expose stack traces or internal errors in production.

### Q: What is rate limiting and why is it important?
> Rate limiting restricts the number of requests a client can make in a time window. It prevents brute force attacks (trying thousands of passwords), DDoS attacks (overwhelming the server), and API abuse. Using `express-rate-limit`, you can allow e.g., 100 requests per 15 minutes globally, and only 5 login attempts per 15 minutes for auth routes.

### Q: What is XSS? How do you prevent it?
> XSS (Cross-Site Scripting) is when an attacker injects malicious JavaScript into your web page, which then runs in other users' browsers. It can steal cookies, session tokens, or redirect users. Prevent it by sanitizing user input, using `helmet` for XSS protection headers, storing tokens in httpOnly cookies (inaccessible to JS), and using template engines that auto-escape HTML.

---

[⬅️ Module 13: Env Config](../13-env-config/README.md) | [Next: Async Patterns ➡️](../15-async-patterns/README.md)
