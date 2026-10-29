# Module 17: Common Interview Questions (Rapid Fire)

All questions with concise, interview-ready answers.

---

## Node.js Core

### Q: What is Node.js?
> A JavaScript runtime built on Chrome's V8 engine. It allows running JavaScript on the server side. It uses a single-threaded, event-driven, non-blocking I/O model, making it efficient for building scalable network applications.

### Q: What is the Event Loop?
> The mechanism that makes non-blocking I/O possible in single-threaded Node.js. It has phases: Timers (setTimeout/setInterval), Poll (I/O callbacks), Check (setImmediate), Close. It continuously checks for pending callbacks and executes them. `process.nextTick()` has highest priority (runs before event loop continues), then microtasks (Promises), then macrotasks.

### Q: What is `libuv`?
> A C library that provides Node.js with its event loop and thread pool. It handles async I/O operations across different operating systems. It maintains a thread pool (default 4 threads) for file system operations, DNS lookups, and other CPU-intensive tasks.

### Q: What are Worker Threads?
> Worker Threads allow running JavaScript in parallel threads, useful for CPU-intensive tasks (image processing, encryption). The main thread handles I/O while workers handle computation. Unlike `child_process`, workers share memory via `SharedArrayBuffer`. Introduced in Node.js 10.5.

### Q: What is clustering in Node.js?
> Clustering creates multiple child processes (workers) that share the same server port. Each worker is a separate Node.js instance running on a different CPU core. It maximizes multi-core CPU usage. The `cluster` module or PM2 handles this automatically.

```js
const cluster = require('cluster');
const os = require('os');

if (cluster.isMaster) {
  const numCPUs = os.cpus().length;
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();  // Create a worker for each CPU core
  }
} else {
  // Each worker runs the Express app
  app.listen(3000);
}
```

### Q: What is the `Buffer` class?
> Buffer is used to handle binary data (images, files, network packets) in Node.js. Since JavaScript was designed for strings, Buffer provides a way to work with raw binary data. Common methods: `Buffer.from('hello')`, `Buffer.alloc(10)`, `buf.toString()`.

### Q: Difference between `spawn` and `exec` in `child_process`?

| | `spawn` | `exec` |
|--|---------|--------|
| Returns | Stream (data in chunks) | Buffer (all data at once) |
| Best for | Long-running processes, large output | Short commands, small output |
| Shell | No shell by default | Runs in shell |
| Memory | Efficient (streams) | Buffers entire output |

### Q: What is the difference between `process.nextTick()` and `setImmediate()`?
> `process.nextTick()` executes before the event loop continues (highest priority async callback). `setImmediate()` executes in the Check phase of the event loop. `nextTick` can starve the event loop if recursive; `setImmediate` is safer. Priority: synchronous → nextTick → Promises → setTimeout/setImmediate.

### Q: How does garbage collection work in Node.js?
> V8 uses a generational garbage collector with two spaces: Young Generation (new objects, frequently collected using Scavenge algorithm) and Old Generation (long-lived objects, collected using Mark-Sweep-Compact). Objects that survive multiple young gen collections are promoted to old gen.

---

## Express.js

### Q: What is Express.js?
> A minimal, unopinionated web framework for Node.js that provides routing, middleware support, template engines, and a simplified API over the raw `http` module. It's the most popular Node.js framework and the "E" in MERN stack.

### Q: How is Express different from Node.js?
> Node.js is the runtime that executes JavaScript. Express is a framework built ON TOP of Node.js that simplifies building web servers. Node.js provides the raw `http` module; Express adds routing, middleware, body parsing, static file serving, and a cleaner API.

### Q: What is middleware chaining?
> Executing multiple middleware functions in sequence for a request. Each middleware calls `next()` to pass control to the next one. Order matters. Example: `app.use(express.json(), cors(), morgan(), authMiddleware)`. Each function runs in the order defined.

### Q: How do you handle 404 and 500 errors?

```js
// 404 — after all routes
app.use((req, res) => {
  res.status(404).json({ message: 'Route not found' });
});

// 500 — error handler (4 params)
app.use((err, req, res, next) => {
  res.status(err.statusCode || 500).json({
    message: err.message || 'Internal Server Error',
  });
});
```

### Q: How do you implement logging in Express?
> Use the `morgan` middleware: `app.use(morgan('dev'))` for development (concise colored output) or `morgan('combined')` for production (Apache combined format). For advanced logging, use `winston` with custom transports (console, file, cloud services).

---

## MERN Specific

### Q: How does the frontend (React) communicate with the backend (Express)?

```
React (client)  →  HTTP Requests  →  Express (server)  →  MongoDB
     ↑                                      ↓
     └────── JSON Response ←────────────────┘

React uses fetch() or axios to make HTTP requests to Express API endpoints.
Express processes the request, queries MongoDB, and returns JSON.
```

```js
// React frontend
const response = await fetch('http://localhost:5000/api/users');
const data = await response.json();

// OR with axios
const { data } = await axios.get('http://localhost:5000/api/users');
```

### Q: What is a proxy in development?

In development, React (port 3000) and Express (port 5000) run on different ports → CORS issues.

```json
// package.json (React app)
{
  "proxy": "http://localhost:5000"
}
```

Now React's `fetch('/api/users')` automatically proxies to `http://localhost:5000/api/users`. No CORS issues in development!

### Q: How do you deploy a MERN app?

**Common approaches:**
1. **Same server:** Build React (`npm run build`), serve static files from Express
   ```js
   app.use(express.static(path.join(__dirname, 'client/build')));
   app.get('*', (req, res) => {
     res.sendFile(path.join(__dirname, 'client/build', 'index.html'));
   });
   ```
2. **Separate servers:** Deploy React on Vercel/Netlify, Express on Render/Railway
3. **Docker:** Containerize both and deploy to AWS/GCP

**Platforms:** Render, Railway, Fly.io, AWS, DigitalOcean, Heroku (paid)

### Q: `axios` vs `fetch`?

| Feature | `fetch` (built-in) | `axios` (npm package) |
|---------|--------------------|-----------------------|
| Installation | Built into browsers & Node 18+ | `npm install axios` |
| JSON parsing | Manual (`.json()`) | Automatic (`data` property) |
| Error handling | Only rejects on network errors | Rejects on 4xx/5xx too |
| Interceptors | ❌ No | ✅ Yes (request/response) |
| Request cancel | AbortController | Built-in cancel token |
| Progress | ❌ No | ✅ Yes |
| Browser support | Modern browsers | All browsers |

```js
// fetch — need to check status and parse JSON manually
const res = await fetch('/api/users');
if (!res.ok) throw new Error('Failed');
const data = await res.json();

// axios — cleaner
const { data } = await axios.get('/api/users');
// Auto throws on error, auto parses JSON
```

### Q: How do you handle CORS between React and Express?

**Development:**
```json
// React package.json
{ "proxy": "http://localhost:5000" }
```

**Production:**
```js
// Express
const cors = require('cors');
app.use(cors({
  origin: 'https://myreactapp.com',
  credentials: true,
}));
```

---

## Bonus: Quick Concept Checks

### What is `req.body`?
> The parsed request body. Requires `express.json()` middleware. Used in POST/PUT/PATCH requests.

### What is the difference between `==` and `===`?
> `==` compares with type coercion (`'5' == 5` is true). `===` strict equality without coercion (`'5' === 5` is false). Always use `===`.

### What is `JSON.stringify()` and `JSON.parse()`?
> `JSON.stringify()` converts a JS object to a JSON string. `JSON.parse()` converts a JSON string back to a JS object. Used for storage and data transfer.

### What does `module.exports` do?
> Exports values (functions, objects, classes) from a file so other files can `require()` them. Each file in Node.js is a module with its own scope.

### What is idempotent?
> An operation that produces the same result no matter how many times you call it. GET, PUT, DELETE are idempotent. POST is NOT (creates a new resource each time).

### What is the difference between `var`, `let`, and `const`?
> `var` — function-scoped, hoisted, can be re-declared. `let` — block-scoped, not hoisted, can be reassigned. `const` — block-scoped, not hoisted, cannot be reassigned (but object properties can change). Use `const` by default, `let` when reassignment needed, never `var`.

### What is the spread operator (`...`)?
```js
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5];  // [1, 2, 3, 4, 5]

const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1, c: 3 };  // { a: 1, b: 2, c: 3 }
```

### What is destructuring?
```js
const { name, email } = req.body;
const [first, second] = myArray;
const { data: users } = await axios.get('/api/users');  // rename
```

---

## 📋 Interview Prep Checklist

Before your interview, make sure you can confidently explain:

- [ ] Event Loop and how Node.js handles concurrency
- [ ] Middleware concept and order of execution
- [ ] REST API principles and proper HTTP method usage
- [ ] JWT authentication flow (register → login → token → protected routes)
- [ ] MongoDB/Mongoose basics (Schema, CRUD, populate)
- [ ] Error handling strategy (AppError + catchAsync + centralized handler)
- [ ] CORS — what it is and how to fix it
- [ ] `async/await` and `Promise.all()`
- [ ] MVC project structure
- [ ] PUT vs PATCH
- [ ] `req.params` vs `req.query`
- [ ] `dependencies` vs `devDependencies`
- [ ] How React communicates with Express

**Good luck with your interview! 🚀💪**

---

[⬅️ Module 16: MVC Architecture](../16-mvc-architecture/README.md) | [🏠 Back to Roadmap](../LEARNING.md)
