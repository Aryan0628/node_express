# Module 11: Authentication & Authorization

## Authentication vs Authorization ⭐

| | Authentication (AuthN) | Authorization (AuthZ) |
|--|----------------------|---------------------|
| **Question** | "Who are you?" | "What can you do?" |
| **Process** | Verify identity (login) | Check permissions |
| **When** | First | After authentication |
| **Example** | Login with email/password | Admin can delete users, regular users can't |
| **HTTP Code** | 401 Unauthorized | 403 Forbidden |

---

## Password Hashing with bcrypt ⭐

**NEVER store plain-text passwords!** Use bcrypt to hash them.

```bash
npm install bcryptjs
```

### How bcrypt works:
```
Plain password: "mypassword123"
        ↓
bcrypt.hash() with salt rounds
        ↓
Hashed: "$2a$10$XrY3J5K8LmN2pQ7rS9tUvOWxY1Z2aB3cD4eF5gH6iJ7kL8mN9oP0"
        ↑
   Cannot reverse this! (one-way hash)
```

### Salting:
```
Without salt: "password" → always same hash → attackers can use rainbow tables
With salt:    "password" + random_salt → unique hash every time → much safer
```

```js
const bcrypt = require('bcryptjs');

// HASH a password (during registration)
const hashPassword = async (plainPassword) => {
  const salt = await bcrypt.genSalt(10);  // 10 = salt rounds (cost factor)
  const hashed = await bcrypt.hash(plainPassword, salt);
  return hashed;
};

// Shorthand (auto-generates salt)
const hashed = await bcrypt.hash('mypassword', 10);

// COMPARE passwords (during login)
const isMatch = await bcrypt.compare('mypassword', hashedPassword);
// true if match, false if not
```

### In Mongoose Schema:
```js
const userSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true, select: false },
});

// Hash password before saving
userSchema.pre('save', async function(next) {
  // Only hash if password is new or modified
  if (!this.isModified('password')) return next();
  
  this.password = await bcrypt.hash(this.password, 10);
  next();
});

// Method to compare passwords
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};
```

---

## JSON Web Tokens (JWT) ⭐⭐ (Very Frequently Asked)

### What is JWT?

JWT is a compact, self-contained token used for securely transmitting information between parties. It's the most common way to handle authentication in MERN apps.

### JWT Structure:
```
xxxxx.yyyyy.zzzzz
  │      │      │
  │      │      └── Signature (verifies the token hasn't been tampered)
  │      └────────── Payload (data: user ID, role, expiry)
  └───────────────── Header (algorithm, token type)
```

### Encoded JWT:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJpZCI6IjEyMyIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTcwMH0.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### Decoded:
```json
// Header
{ "alg": "HS256", "typ": "JWT" }

// Payload
{ "id": "123", "role": "admin", "iat": 1700000000, "exp": 1700086400 }

// Signature
HMACSHA256(base64(header) + "." + base64(payload), SECRET_KEY)
```

```bash
npm install jsonwebtoken
```

```js
const jwt = require('jsonwebtoken');

// CREATE (sign) a token
const token = jwt.sign(
  { id: user._id, role: user.role },  // Payload (data)
  process.env.JWT_SECRET,              // Secret key
  { expiresIn: '7d' }                  // Options (expiry)
);
// expiresIn: '30s', '10m', '1h', '7d', '30d'

// VERIFY a token
try {
  const decoded = jwt.verify(token, process.env.JWT_SECRET);
  console.log(decoded);
  // { id: '64a1b2c3...', role: 'admin', iat: 1700000000, exp: 1700604800 }
} catch (err) {
  console.log('Invalid or expired token');
}
```

---

## Complete Auth Flow ⭐

### 1. User Model

```js
// models/User.js
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Name is required'],
    trim: true,
  },
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
  },
  password: {
    type: String,
    required: [true, 'Password is required'],
    minlength: 6,
    select: false,  // Don't return password in queries
  },
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user',
  },
}, { timestamps: true });

// Hash password before saving
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  this.password = await bcrypt.hash(this.password, 10);
  next();
});

// Compare password method
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};

module.exports = mongoose.model('User', userSchema);
```

### 2. Auth Controller

```js
// controllers/authController.js
const User = require('../models/User');
const jwt = require('jsonwebtoken');

// Helper: Generate token
const generateToken = (userId) => {
  return jwt.sign({ id: userId }, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRES_IN || '7d',
  });
};

// REGISTER
exports.register = async (req, res) => {
  try {
    const { name, email, password } = req.body;
    
    // Check if user exists
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(409).json({ message: 'Email already registered' });
    }
    
    // Create user (password is hashed by pre-save hook)
    const user = await User.create({ name, email, password });
    
    // Generate token
    const token = generateToken(user._id);
    
    res.status(201).json({
      success: true,
      token,
      data: {
        id: user._id,
        name: user.name,
        email: user.email,
        role: user.role,
      },
    });
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
};

// LOGIN
exports.login = async (req, res) => {
  try {
    const { email, password } = req.body;
    
    // Validate input
    if (!email || !password) {
      return res.status(400).json({ message: 'Email and password required' });
    }
    
    // Find user (include password since select: false)
    const user = await User.findOne({ email }).select('+password');
    
    if (!user) {
      return res.status(401).json({ message: 'Invalid credentials' });
    }
    
    // Check password
    const isMatch = await user.comparePassword(password);
    
    if (!isMatch) {
      return res.status(401).json({ message: 'Invalid credentials' });
    }
    
    // Generate token
    const token = generateToken(user._id);
    
    res.json({
      success: true,
      token,
      data: {
        id: user._id,
        name: user.name,
        email: user.email,
        role: user.role,
      },
    });
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
};

// GET CURRENT USER (protected route)
exports.getMe = async (req, res) => {
  // req.user is set by the auth middleware
  const user = await User.findById(req.user.id);
  res.json({ success: true, data: user });
};
```

### 3. Auth Middleware (Protecting Routes)

```js
// middleware/auth.js
const jwt = require('jsonwebtoken');
const User = require('../models/User');

// Protect routes — user must be logged in
const protect = async (req, res, next) => {
  try {
    let token;
    
    // Check for token in Authorization header
    if (req.headers.authorization && 
        req.headers.authorization.startsWith('Bearer')) {
      token = req.headers.authorization.split(' ')[1];
      // "Bearer eyJhbGci..." → "eyJhbGci..."
    }
    
    if (!token) {
      return res.status(401).json({ message: 'Not authorized, no token' });
    }
    
    // Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // Attach user to request
    req.user = await User.findById(decoded.id);
    
    if (!req.user) {
      return res.status(401).json({ message: 'User no longer exists' });
    }
    
    next();
  } catch (err) {
    return res.status(401).json({ message: 'Not authorized, token invalid' });
  }
};

// Authorize by role
const authorize = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        message: `Role '${req.user.role}' is not authorized for this action`,
      });
    }
    next();
  };
};

module.exports = { protect, authorize };
```

### 4. Routes

```js
// routes/authRoutes.js
const express = require('express');
const router = express.Router();
const { register, login, getMe } = require('../controllers/authController');
const { protect, authorize } = require('../middleware/auth');

router.post('/register', register);
router.post('/login', login);
router.get('/me', protect, getMe);

module.exports = router;
```

```js
// routes/userRoutes.js
const express = require('express');
const router = express.Router();
const { protect, authorize } = require('../middleware/auth');

// All routes below require authentication
router.use(protect);

// Only admins can delete users
router.delete('/:id', authorize('admin'), async (req, res) => {
  await User.findByIdAndDelete(req.params.id);
  res.json({ message: 'User deleted' });
});

module.exports = router;
```

### 5. Using in App

```js
// app.js
const authRoutes = require('./routes/authRoutes');
const userRoutes = require('./routes/userRoutes');

app.use('/api/auth', authRoutes);
app.use('/api/users', userRoutes);
```

---

## Auth Flow Diagram

```
REGISTER:
Client → POST /api/auth/register { name, email, password }
Server → Hash password → Save user → Generate JWT → Return { token }

LOGIN:
Client → POST /api/auth/login { email, password }
Server → Find user → Compare passwords → Generate JWT → Return { token }

PROTECTED REQUEST:
Client → GET /api/users (Authorization: Bearer <token>)
Server → Extract token → jwt.verify() → Find user → Attach to req → Route handler
```

---

## Token Storage ⭐

| Method | Pros | Cons |
|--------|------|------|
| **localStorage** | Easy to use, persists | Vulnerable to XSS attacks |
| **sessionStorage** | Cleared on tab close | Vulnerable to XSS, lost on tab close |
| **httpOnly Cookie** | Not accessible by JS (XSS safe) | Needs CSRF protection |

### Best practice: httpOnly cookies

```js
// Setting token in cookie (server-side)
res.cookie('token', token, {
  httpOnly: true,      // JS can't access it
  secure: process.env.NODE_ENV === 'production',  // HTTPS only in prod
  sameSite: 'strict',  // CSRF protection
  maxAge: 7 * 24 * 60 * 60 * 1000,  // 7 days
});
```

---

## Session-based vs Token-based Auth ⭐

| | Session-based | Token-based (JWT) |
|--|--------------|------------------|
| **Storage** | Server stores session in memory/DB | Client stores token |
| **Stateful?** | Yes (server tracks sessions) | No (stateless) |
| **Scalability** | Harder (sessions tied to server) | Easier (any server can verify) |
| **Mobile friendly** | Not ideal | ✅ Works great |
| **Used in** | Traditional web apps | SPAs, mobile apps, MERN |

---

## 🎯 Interview Answers

### Q: What is JWT and how does it work?
> JWT (JSON Web Token) is a compact token format for securely transmitting information. It has three parts: Header (algorithm), Payload (data like user ID and expiry), and Signature (ensures the token hasn't been tampered with). The server creates a JWT on login using a secret key, sends it to the client, and the client includes it in the Authorization header for subsequent requests. The server verifies the token's signature and extracts user info without querying the database.

### Q: How do you protect routes in Express?
> Create an authentication middleware that extracts the JWT from the `Authorization: Bearer <token>` header, verifies it using `jwt.verify()`, finds the user from the decoded payload, and attaches the user to `req.user`. If verification fails, return a 401 status. Apply this middleware to protected routes using `app.use(protect)` or as a route-level middleware.

### Q: Session-based vs Token-based auth?
> Session-based auth stores session data on the server and sends a session ID cookie to the client — it's stateful. Token-based (JWT) auth sends a self-contained token to the client — the server doesn't store anything, making it stateless. JWT is preferred in MERN because it scales better (any server can verify the token), works well with mobile apps, and supports microservices. Session-based is simpler but harder to scale across multiple servers.

### Q: Why should you hash passwords? How does bcrypt work?
> Storing plain-text passwords means if the database is breached, all user passwords are exposed. Hashing is one-way — you can't reverse it. bcrypt adds a **salt** (random string) before hashing, so even identical passwords produce different hashes, defeating rainbow table attacks. The salt rounds parameter (usually 10-12) controls the computational cost, making brute-force attacks slow.

---

[⬅️ Module 10: MongoDB](../10-mongodb-mongoose/README.md) | [Next: File Upload ➡️](../12-file-upload/README.md)
