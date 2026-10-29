# 🚀 Node.js & Express — Interview Prep Roadmap (MERN Stack)

> **Goal:** Revise Node.js & Express from scratch, focusing on concepts frequently asked in SDE Intern interviews.
> **How to use:** Go topic by topic. Each topic has a status tracker. We'll create code examples for each as we learn.

---

## 📊 Progress Tracker

| # | Module | Status |
|---|--------|--------|
| 1 | Node.js Fundamentals | ⬜ Not Started |
| 2 | Node.js Core Modules | ⬜ Not Started |
| 3 | npm & package.json | ⬜ Not Started |
| 4 | Express.js Basics | ⬜ Not Started |
| 5 | Routing in Express | ⬜ Not Started |
| 6 | Middleware | ⬜ Not Started |
| 7 | Request & Response Deep Dive | ⬜ Not Started |
| 8 | REST API Design | ⬜ Not Started |
| 9 | Error Handling | ⬜ Not Started |
| 10 | MongoDB + Mongoose (with Express) | ⬜ Not Started |
| 11 | Authentication & Authorization | ⬜ Not Started |
| 12 | File Upload & Static Files | ⬜ Not Started |
| 13 | Environment Variables & Config | ⬜ Not Started |
| 14 | CORS & Security Basics | ⬜ Not Started |
| 15 | Async Patterns in Node.js | ⬜ Not Started |
| 16 | MVC Architecture | ⬜ Not Started |
| 17 | Common Interview Questions | ⬜ Not Started |

---

## Module 1: Node.js Fundamentals

### Topics to Cover
- [ ] What is Node.js? (Runtime, not a framework/language)
- [ ] V8 Engine — how JS runs outside the browser
- [ ] Event-Driven Architecture
- [ ] Single-threaded & Non-blocking I/O model
- [ ] Event Loop — phases (timers, poll, check, etc.)
- [ ] `global` object vs `window` in browser
- [ ] `process` object (`process.env`, `process.argv`, `process.exit()`)
- [ ] `setTimeout`, `setInterval`, `setImmediate`, `process.nextTick` — order of execution
- [ ] CommonJS (`require/module.exports`) vs ES Modules (`import/export`)

### 🎯 Interview Questions
1. What is Node.js and why is it used?
2. Explain the Event Loop in Node.js.
3. What is the difference between `process.nextTick()` and `setImmediate()`?
4. Is Node.js single-threaded? How does it handle concurrent requests?
5. Difference between CommonJS and ES Modules?

---

## Module 2: Node.js Core Modules

### Topics to Cover
- [ ] `fs` — File System (readFile, writeFile, sync vs async)
- [ ] `path` — `path.join()`, `path.resolve()`, `__dirname`, `__filename`
- [ ] `http` — Creating a basic HTTP server without Express
- [ ] `os` — System info
- [ ] `events` — EventEmitter class, custom events
- [ ] `stream` — Readable, Writable, Transform streams (basics)
- [ ] `url` — URL parsing
- [ ] `crypto` — Hashing passwords (bcrypt concept)

### 🎯 Interview Questions
1. How do you create an HTTP server in Node.js without Express?
2. What are Streams in Node.js? Why are they useful?
3. What is the EventEmitter class?
4. Difference between `fs.readFile` and `fs.readFileSync`?

---

## Module 3: npm & package.json

### Topics to Cover
- [ ] What is npm? (Node Package Manager)
- [ ] `npm init`, `npm init -y`
- [ ] `dependencies` vs `devDependencies`
- [ ] `npm install`, `npm install --save-dev`, `npm install -g`
- [ ] `package.json` vs `package-lock.json`
- [ ] Semantic Versioning (`^`, `~`, exact)
- [ ] npm scripts (`start`, `dev`, `test`, custom scripts)
- [ ] `npx` — what it does
- [ ] `.gitignore` — always ignore `node_modules`

### 🎯 Interview Questions
1. What is the difference between `dependencies` and `devDependencies`?
2. What is `package-lock.json` and why is it important?
3. What does `^` and `~` mean in version numbers?
4. What is `npx`?

---

## Module 4: Express.js Basics

### Topics to Cover
- [ ] What is Express? (Minimal, unopinionated web framework)
- [ ] Installing Express — `npm install express`
- [ ] Creating a basic Express server
- [ ] `app.listen()` — starting the server
- [ ] `app.get()`, `app.post()`, `app.put()`, `app.patch()`, `app.delete()`
- [ ] Sending responses — `res.send()`, `res.json()`, `res.status()`
- [ ] `nodemon` for auto-restart during development

### 🎯 Interview Questions
1. What is Express.js and why use it over plain `http` module?
2. How do you create a basic Express server?
3. What is the difference between `res.send()` and `res.json()`?

---

## Module 5: Routing in Express

### Topics to Cover
- [ ] Basic routing — `app.get('/path', handler)`
- [ ] Route parameters — `req.params` (e.g., `/users/:id`)
- [ ] Query strings — `req.query` (e.g., `/search?q=node`)
- [ ] `express.Router()` — modular routing
- [ ] Route chaining — `app.route('/path').get().post().put()`
- [ ] Nested routes / Sub-routers
- [ ] Wildcard routes and 404 handling

### 🎯 Interview Questions
1. What is the difference between `req.params` and `req.query`?
2. How do you create modular routes using `express.Router()`?
3. How do you handle 404 (Not Found) in Express?

---

## Module 6: Middleware

### Topics to Cover
- [ ] What is Middleware? (Functions with access to `req`, `res`, `next`)
- [ ] How middleware works — the request-response cycle
- [ ] `next()` function — passing control to next middleware
- [ ] Application-level middleware — `app.use()`
- [ ] Router-level middleware
- [ ] Built-in middleware — `express.json()`, `express.urlencoded()`, `express.static()`
- [ ] Third-party middleware — `cors`, `morgan`, `helmet`, `cookie-parser`
- [ ] Custom middleware — logging, auth checks
- [ ] Error-handling middleware — `(err, req, res, next)`
- [ ] Order of middleware matters!

### 🎯 Interview Questions
1. What is middleware in Express? Give examples.
2. What is `next()` and what happens if you don't call it?
3. What is the difference between `app.use()` and `app.get()`?
4. How does error-handling middleware differ from regular middleware?
5. Why does the order of middleware matter?

---

## Module 7: Request & Response Deep Dive

### Topics to Cover
- [ ] `req.body` — parsing JSON and form data
- [ ] `req.params` — route parameters
- [ ] `req.query` — query string parameters
- [ ] `req.headers` — accessing request headers
- [ ] `req.cookies` — reading cookies (with cookie-parser)
- [ ] `req.method`, `req.url`, `req.path`
- [ ] `res.status()` — setting HTTP status codes
- [ ] `res.json()` — sending JSON response
- [ ] `res.send()` — sending generic response
- [ ] `res.redirect()` — redirecting
- [ ] `res.cookie()` — setting cookies
- [ ] HTTP Status Codes — 200, 201, 204, 400, 401, 403, 404, 500

### 🎯 Interview Questions
1. How do you access the request body in Express?
2. Why do we need `express.json()` middleware?
3. Name 5 HTTP status codes and when to use them.

---

## Module 8: REST API Design

### Topics to Cover
- [ ] What is REST? (Representational State Transfer)
- [ ] REST principles — stateless, uniform interface, resource-based
- [ ] HTTP Methods mapping — CRUD operations
  - `GET` → Read
  - `POST` → Create
  - `PUT` → Full Update
  - `PATCH` → Partial Update
  - `DELETE` → Delete
- [ ] Resource naming conventions (`/api/users`, `/api/users/:id`)
- [ ] API versioning (`/api/v1/users`)
- [ ] Pagination, filtering, sorting in APIs
- [ ] Response format — consistent JSON structure
- [ ] Build a complete CRUD API for a resource (e.g., Users, Todos)

### 🎯 Interview Questions
1. What is a RESTful API?
2. Difference between PUT and PATCH?
3. How do you design pagination in a REST API?
4. What makes an API RESTful?

---

## Module 9: Error Handling

### Topics to Cover
- [ ] try-catch blocks in async/await
- [ ] Express error-handling middleware `(err, req, res, next)`
- [ ] Centralized error handling
- [ ] Custom Error classes (e.g., `AppError`, `NotFoundError`)
- [ ] Handling async errors — `express-async-errors` or wrapper function
- [ ] Operational errors vs Programming errors
- [ ] Unhandled Promise Rejections — `process.on('unhandledRejection')`
- [ ] Uncaught Exceptions — `process.on('uncaughtException')`

### 🎯 Interview Questions
1. How do you handle errors in Express?
2. What is centralized error handling?
3. How do you handle async errors in Express?

---

## Module 10: MongoDB + Mongoose (with Express)

### Topics to Cover
- [ ] What is MongoDB? (NoSQL, Document-based)
- [ ] MongoDB vs SQL databases — when to use what
- [ ] Connecting to MongoDB — `mongoose.connect()`
- [ ] Mongoose Schema & Model
- [ ] Schema types — String, Number, Boolean, Date, Array, ObjectId
- [ ] Schema validation — `required`, `unique`, `minlength`, `enum`, `default`
- [ ] Custom validators
- [ ] CRUD operations with Mongoose:
  - `Model.create()` / `new Model().save()`
  - `Model.find()`, `Model.findById()`, `Model.findOne()`
  - `Model.findByIdAndUpdate()`, `Model.updateOne()`
  - `Model.findByIdAndDelete()`, `Model.deleteOne()`
- [ ] Mongoose Middleware (pre/post hooks) — `pre('save')`, etc.
- [ ] `populate()` — referencing other documents
- [ ] Virtuals — computed fields
- [ ] Indexing basics
- [ ] Aggregation Pipeline basics (`$match`, `$group`, `$sort`, `$project`)

### 🎯 Interview Questions
1. What is Mongoose and why use it with MongoDB?
2. What is a Schema in Mongoose?
3. Explain `populate()` in Mongoose.
4. How do you add validation in Mongoose?
5. What are pre/post hooks in Mongoose?
6. SQL vs NoSQL — when to use which?

---

## Module 11: Authentication & Authorization

### Topics to Cover
- [ ] Authentication vs Authorization
- [ ] Password hashing — `bcrypt` / `bcryptjs`
- [ ] JSON Web Tokens (JWT)
  - What is JWT? (Header, Payload, Signature)
  - `jwt.sign()` — creating tokens
  - `jwt.verify()` — verifying tokens
  - Token expiration
- [ ] Auth flow: Register → Login → Get Token → Use Token
- [ ] Protecting routes — auth middleware
- [ ] Role-based access control (admin, user)
- [ ] Storing tokens — localStorage vs httpOnly cookies
- [ ] Session-based vs Token-based authentication
- [ ] Refresh tokens (concept)

### 🎯 Interview Questions
1. What is JWT and how does it work?
2. Difference between authentication and authorization?
3. How do you protect routes in Express?
4. Session-based vs Token-based auth — pros and cons?
5. Why should you hash passwords? How does bcrypt work?

---

## Module 12: File Upload & Static Files

### Topics to Cover
- [ ] Serving static files — `express.static()`
- [ ] File uploads with `multer`
  - Single file upload
  - Multiple file uploads
  - File filtering (only images, etc.)
  - Storage options (disk, memory)
- [ ] Serving uploaded files

### 🎯 Interview Questions
1. How do you serve static files in Express?
2. How do you handle file uploads in Express?

---

## Module 13: Environment Variables & Config

### Topics to Cover
- [ ] Why use environment variables?
- [ ] `dotenv` package — `.env` file
- [ ] `process.env.VARIABLE_NAME`
- [ ] Different configs for dev/prod/test
- [ ] Never commit `.env` to git!
- [ ] Common env variables: `PORT`, `MONGODB_URI`, `JWT_SECRET`, `NODE_ENV`

### 🎯 Interview Questions
1. Why do we use environment variables?
2. What is the `dotenv` package?
3. What should you never put in your source code?

---

## Module 14: CORS & Security Basics

### Topics to Cover
- [ ] What is CORS? (Cross-Origin Resource Sharing)
- [ ] Why CORS errors happen (Same-Origin Policy)
- [ ] `cors` package — enabling CORS in Express
- [ ] `helmet` — security headers
- [ ] Rate limiting — `express-rate-limit`
- [ ] Data sanitization — preventing NoSQL injection
- [ ] Input validation with `joi` or `express-validator`
- [ ] XSS, CSRF basics (conceptual)
- [ ] HTTPS basics

### 🎯 Interview Questions
1. What is CORS and how do you handle it?
2. How do you prevent common security vulnerabilities in Express?
3. What is rate limiting and why is it important?
4. What is XSS? How do you prevent it?

---

## Module 15: Async Patterns in Node.js

### Topics to Cover
- [ ] Callbacks — callback hell
- [ ] Promises — `.then()`, `.catch()`, `.finally()`
- [ ] `async/await` — modern approach
- [ ] `Promise.all()` — parallel execution
- [ ] `Promise.allSettled()` — all results regardless of failure
- [ ] `Promise.race()` — first to resolve/reject
- [ ] Error handling in async code

### 🎯 Interview Questions
1. What is callback hell and how do you avoid it?
2. Explain `async/await` in Node.js.
3. What is the difference between `Promise.all()` and `Promise.allSettled()`?
4. How does error handling work with `async/await`?

---

## Module 16: MVC Architecture

### Topics to Cover
- [ ] What is MVC? (Model-View-Controller)
- [ ] Project structure for Express apps:
  ```
  project/
  ├── controllers/    # Business logic
  ├── models/         # Mongoose schemas
  ├── routes/         # Route definitions
  ├── middleware/     # Custom middleware
  ├── config/         # DB connection, env config
  ├── utils/          # Helper functions
  ├── app.js          # Express app setup
  └── server.js       # Server entry point
  ```
- [ ] Separation of concerns
- [ ] Controller functions
- [ ] Service layer (optional)
- [ ] Why separate `app.js` and `server.js`?

### 🎯 Interview Questions
1. What is MVC architecture?
2. How do you structure an Express project?
3. Why is separation of concerns important?

---

## Module 17: Common Interview Questions (Rapid Fire)

### Node.js
- [ ] What is the Event Loop?
- [ ] What is `libuv`?
- [ ] What are Worker Threads?
- [ ] What is clustering in Node.js?
- [ ] How does garbage collection work in Node.js?
- [ ] What is the `Buffer` class?
- [ ] Difference between `spawn` and `exec` in `child_process`?

### Express.js
- [ ] What is Express.js?
- [ ] How is Express different from Node.js?
- [ ] What is middleware chaining?
- [ ] How do you handle 404 and 500 errors?
- [ ] How do you implement logging in Express?

### MERN Specific
- [ ] How does the frontend (React) communicate with the backend (Express)?
- [ ] What is a proxy in development? (`proxy` in package.json)
- [ ] How do you deploy a MERN app?
- [ ] What is `axios` vs `fetch`?
- [ ] How do you handle CORS between React and Express?

---

## 📁 Folder Structure for Our Learning

We'll create code examples for each module:

```
node_express/
├── LEARNING.md              ← You are here
├── 01-node-fundamentals/
├── 02-core-modules/
├── 03-npm-basics/
├── 04-express-basics/
├── 05-routing/
├── 06-middleware/
├── 07-req-res/
├── 08-rest-api/
├── 09-error-handling/
├── 10-mongodb-mongoose/
├── 11-auth/
├── 12-file-upload/
├── 13-env-config/
├── 14-cors-security/
├── 15-async-patterns/
├── 16-mvc-architecture/
└── 17-interview-questions/
```

---

## 🎯 How to Use This Roadmap

1. **Go module by module** — tell me which module you want to start with
2. I'll create **code examples + explanations** in the respective folder
3. Each module will have a `README.md` with theory + code
4. We'll mark topics as ✅ as we complete them
5. Practice the interview questions after each module

---

> **Ready?** Tell me which module to start with, or just say **"start from Module 1"** and we'll begin! 🚀
