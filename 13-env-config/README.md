# Module 13: Environment Variables & Config

## Why Environment Variables?

Different environments (development, staging, production) need different configs:
- **Development:** `localhost` DB, debug logging
- **Production:** Cloud DB, minimal logging, HTTPS

You also have **secrets** (DB passwords, API keys, JWT secrets) that should NEVER be in your code.

---

## dotenv Package

```bash
npm install dotenv
```

### Create `.env` file:
```env
# .env (root of project)
NODE_ENV=development
PORT=3000

# Database
MONGODB_URI=mongodb://localhost:27017/myapp

# JWT
JWT_SECRET=my_super_secret_key_change_in_production
JWT_EXPIRES_IN=7d

# Email (example)
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your@email.com
EMAIL_PASS=yourpassword

# API Keys
STRIPE_API_KEY=sk_test_abc123
```

### Load in your app:
```js
// server.js — MUST be at the very top, before any other imports
require('dotenv').config();

// Now you can access them anywhere:
console.log(process.env.PORT);          // '3000'
console.log(process.env.MONGODB_URI);   // 'mongodb://...'
console.log(process.env.JWT_SECRET);    // 'my_super_secret...'
console.log(process.env.NODE_ENV);      // 'development'
```

### Using with config file:
```js
// config/config.js
require('dotenv').config();

module.exports = {
  port: process.env.PORT || 3000,
  mongoUri: process.env.MONGODB_URI,
  jwtSecret: process.env.JWT_SECRET,
  jwtExpiresIn: process.env.JWT_EXPIRES_IN || '7d',
  nodeEnv: process.env.NODE_ENV || 'development',
};
```

```js
// Usage anywhere
const config = require('./config/config');
app.listen(config.port);
mongoose.connect(config.mongoUri);
```

---

## Important Rules

### ⚠️ NEVER commit `.env` to git!

```gitignore
# .gitignore
.env
.env.local
.env.production
```

### Create a `.env.example` file (DO commit this):
```env
# .env.example — shows what variables are needed
NODE_ENV=development
PORT=3000
MONGODB_URI=mongodb://localhost:27017/myapp
JWT_SECRET=your_jwt_secret_here
JWT_EXPIRES_IN=7d
```

### All `process.env` values are STRINGS:
```js
process.env.PORT         // '3000' (string, not number!)
parseInt(process.env.PORT)  // 3000 (number)

process.env.DEBUG        // 'true' (string, not boolean!)
process.env.DEBUG === 'true'  // true (boolean comparison)
```

---

## 🎯 Interview Answers

### Q: Why do we use environment variables?
> Environment variables let us store configuration that changes between environments (development, production) and sensitive data (API keys, database passwords, JWT secrets) outside the codebase. This follows the twelve-factor app methodology, keeps secrets secure, and makes the app configurable without code changes.

### Q: What is the `dotenv` package?
> `dotenv` loads variables from a `.env` file into `process.env`. You call `require('dotenv').config()` at the top of your entry file, and it makes all variables in `.env` accessible via `process.env.VARIABLE_NAME`. The `.env` file should never be committed to git — instead, commit a `.env.example` showing required variables without actual values.

---

[⬅️ Module 12: File Upload](../12-file-upload/README.md) | [Next: CORS & Security ➡️](../14-cors-security/README.md)
