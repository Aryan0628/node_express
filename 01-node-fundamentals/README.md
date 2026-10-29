# Module 1: Node.js Fundamentals

## What is Node.js?

Node.js is a **JavaScript runtime** built on Chrome's **V8 engine**. It lets you run JavaScript **outside the browser** — on servers, CLIs, etc.

> ⚠️ Node.js is NOT a language. It's NOT a framework. It's a **runtime environment**.

```
Browser → JS runs using V8 engine (Chrome), SpiderMonkey (Firefox)
Node.js → JS runs using V8 engine on your machine/server
```

### Why Node.js?
- **Same language** for frontend + backend (JavaScript)
- **Non-blocking I/O** — handles many requests without waiting
- **Huge ecosystem** — npm has millions of packages
- **Fast** — V8 compiles JS to machine code
- **Great for APIs** — especially REST APIs, real-time apps (chat, streaming)

---

## V8 Engine

V8 is Google's open-source JavaScript engine written in C++. It:
1. Takes your JavaScript code
2. Compiles it to **machine code** (not interpreted line by line)
3. Executes it directly on the CPU

```
Your JS Code → V8 Engine → Machine Code → CPU executes it
```

Node.js wraps V8 and adds extra features that browsers don't have:
- File system access (`fs`)
- Network access (`http`, `net`)
- OS interaction (`os`, `child_process`)

---

## Single-Threaded & Non-Blocking I/O

### Single-Threaded
Node.js uses **one main thread** to execute your JavaScript code. It does NOT create a new thread for each request (unlike Java/PHP).

### Non-Blocking I/O
When Node.js encounters an I/O operation (reading a file, database query, API call), it **doesn't wait**. It sends the operation to the system and moves on. When the result is ready, a **callback** is triggered.

```js
// NON-BLOCKING (Async) — Node.js way
const fs = require('fs');

console.log('1. Start');

fs.readFile('file.txt', 'utf8', (err, data) => {
  console.log('3. File read complete');  // This runs LATER
});

console.log('2. End');

// Output:
// 1. Start
// 2. End
// 3. File read complete
```

```js
// BLOCKING (Sync) — Avoid this in production
const fs = require('fs');

console.log('1. Start');

const data = fs.readFileSync('file.txt', 'utf8');  // WAITS here
console.log('2. File read complete');

console.log('3. End');

// Output:
// 1. Start
// 2. File read complete
// 3. End
```

### How does single-threaded handle thousands of requests?

```
Request 1 → "Read DB" → offloaded to system → moves on
Request 2 → "Read file" → offloaded to system → moves on
Request 3 → "Compute 2+2" → done instantly → responds

Meanwhile...
DB result ready → callback fires → responds to Request 1
File result ready → callback fires → responds to Request 2
```

Node.js uses **libuv** (a C library) which manages a **thread pool** (default 4 threads) for heavy I/O operations behind the scenes.

---

## Event Loop ⭐ (Most Asked in Interviews)

The Event Loop is the mechanism that allows Node.js to perform non-blocking I/O operations despite being single-threaded.

### How it works (simplified):

```
   ┌───────────────────────────┐
┌─>│         TIMERS            │  → setTimeout, setInterval callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       PENDING CALLBACKS   │  → I/O callbacks deferred to next loop
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       IDLE, PREPARE       │  → internal use only
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │          POLL             │  → retrieve new I/O events, execute callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │          CHECK            │  → setImmediate callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │      CLOSE CALLBACKS      │  → socket.on('close'), etc.
│  └─────────────┬─────────────┘
│                │
└────────────────┘  (loops back if there's more work)
```

### Key phases:
1. **Timers** — executes `setTimeout` and `setInterval` callbacks
2. **Poll** — retrieves new I/O events (file reads, network, etc.)
3. **Check** — executes `setImmediate()` callbacks

---

## `process.nextTick()` vs `setImmediate()` ⭐

This is a **classic interview question**.

```js
setTimeout(() => console.log('setTimeout'), 0);
setImmediate(() => console.log('setImmediate'));
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('Promise'));

console.log('Synchronous');

// Output:
// Synchronous
// nextTick
// Promise
// setTimeout (or setImmediate — order varies)
// setImmediate (or setTimeout — order varies)
```

### Priority order:
```
1. Synchronous code         → runs first, always
2. process.nextTick()       → runs BEFORE the event loop continues
3. Microtasks (Promises)    → runs after nextTick, before macrotasks
4. Macrotasks (setTimeout, setImmediate, I/O callbacks)
```

| Feature | `process.nextTick()` | `setImmediate()` |
|---------|---------------------|------------------|
| When it runs | **Before** the event loop continues | **After** the current poll phase |
| Priority | Higher | Lower |
| Use case | When you need something to run immediately after current operation | When you want to run after I/O events |
| Danger | Can starve the event loop if called recursively | Safer for recursive calls |

---

## The `global` Object

In the browser, the global object is `window`. In Node.js, it's `global`.

```js
// Browser
console.log(window);    // the global object
console.log(this);      // points to window (in global scope)

// Node.js
console.log(global);    // the global object
console.log(this);      // {} (empty object in a module, NOT global)
```

### Things available on `global`:
```js
console.log()       // global.console.log()
setTimeout()        // global.setTimeout()
setInterval()       // global.setInterval()
setImmediate()      // global.setImmediate()
Buffer              // global.Buffer
process             // global.process
```

> In Node.js, each file is a **module** with its own scope. Variables declared in a file are NOT added to `global`.

---

## The `process` Object ⭐

`process` is a global object that provides info and control over the current Node.js process.

```js
// Environment variables
console.log(process.env.NODE_ENV);    // 'development' or 'production'
console.log(process.env.PORT);        // e.g., '3000'

// Command line arguments
// Run: node app.js hello world
console.log(process.argv);
// [
//   '/usr/local/bin/node',   → path to node
//   '/path/to/app.js',       → path to your script
//   'hello',                 → your arguments
//   'world'
// ]

// Current working directory
console.log(process.cwd());    // '/Users/aryan/project'

// Exit the process
process.exit(0);    // 0 = success
process.exit(1);    // 1 = failure/error

// Memory usage
console.log(process.memoryUsage());
// { rss: 30000000, heapTotal: 6000000, heapUsed: 4000000, external: 8000 }

// Platform
console.log(process.platform);    // 'darwin' (macOS), 'linux', 'win32'

// Node.js version
console.log(process.version);     // 'v20.10.0'
```

---

## CommonJS vs ES Modules ⭐

### CommonJS (CJS) — Default in Node.js
```js
// exporting — math.js
const add = (a, b) => a + b;
const subtract = (a, b) => a - b;

module.exports = { add, subtract };
// OR
module.exports.add = add;
// OR
exports.add = add;  // shorthand (but don't reassign exports directly)

// importing — app.js
const { add, subtract } = require('./math');
// OR
const math = require('./math');
math.add(2, 3);
```

### ES Modules (ESM) — Modern JavaScript
```js
// To use ES Modules in Node.js, either:
// 1. Use .mjs extension, OR
// 2. Add "type": "module" in package.json

// exporting — math.mjs (or math.js with "type": "module")
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;
// OR
export default function add(a, b) { return a + b; }

// importing — app.mjs
import { add, subtract } from './math.mjs';
// OR
import add from './math.mjs';  // for default export
```

### Key Differences:

| Feature | CommonJS (`require`) | ES Modules (`import`) |
|---------|---------------------|----------------------|
| Loading | **Synchronous** | **Asynchronous** |
| When loaded | **Runtime** (dynamic) | **Parse time** (static) |
| Syntax | `require()` / `module.exports` | `import` / `export` |
| Can be conditional? | ✅ Yes (`if (x) require(...)`) | ❌ No (top-level only) |
| Tree shaking | ❌ No | ✅ Yes (bundlers can remove unused) |
| `this` in module | `module.exports` | `undefined` |
| Default in Node.js | ✅ Yes | Need `"type": "module"` or `.mjs` |
| Used in browsers | ❌ No | ✅ Yes (with `<script type="module">`) |

### ⚠️ Common Gotcha
```js
// This DOES NOT work as expected:
exports = { add, subtract };  // ❌ This breaks the reference!

// This works:
module.exports = { add, subtract };  // ✅ Correct
exports.add = add;                    // ✅ Also correct
```

Why? Because `exports` is just a **reference** to `module.exports`. Reassigning `exports` breaks that reference.

---

## 🎯 Interview Answers (Ready to Use)

### Q: What is Node.js and why is it used?
> Node.js is a JavaScript runtime built on Chrome's V8 engine that allows JavaScript to run outside the browser — primarily on servers. It uses a single-threaded, event-driven, non-blocking I/O model, which makes it lightweight and efficient for building scalable network applications like REST APIs and real-time apps. It's used because it allows developers to use JavaScript for both frontend and backend, has a massive npm ecosystem, and handles concurrent connections efficiently.

### Q: Explain the Event Loop in Node.js.
> The Event Loop is the core mechanism that allows Node.js to perform non-blocking operations despite being single-threaded. It continuously checks for pending callbacks and executes them. It has several phases: Timers (setTimeout/setInterval), Poll (I/O callbacks), Check (setImmediate), and Close callbacks. When an async operation completes, its callback is queued and the event loop picks it up in the appropriate phase.

### Q: Is Node.js single-threaded? How does it handle concurrent requests?
> Yes, the main JavaScript execution is single-threaded. However, Node.js uses libuv under the hood, which maintains a thread pool (default 4 threads) for CPU-intensive and blocking I/O operations. When a request involves I/O (like reading a file or querying a database), Node offloads it to the thread pool, and the main thread continues handling other requests. When the I/O completes, a callback is triggered on the main thread.

### Q: Difference between `process.nextTick()` and `setImmediate()`?
> `process.nextTick()` executes its callback **before** the event loop continues to the next phase — it has the highest priority among async callbacks. `setImmediate()` executes its callback in the **check phase** of the event loop, after the poll phase. `nextTick` is faster but can starve the event loop if used recursively, while `setImmediate` is safer for recursive operations.

### Q: Difference between CommonJS and ES Modules?
> CommonJS uses `require()` and `module.exports`, loads modules synchronously at runtime, and is the default in Node.js. ES Modules use `import/export`, load asynchronously at parse time, support tree-shaking, and are the standard in modern JavaScript and browsers. In Node.js, you need to either use `.mjs` extension or set `"type": "module"` in package.json to use ES Modules.

---

[⬅️ Back to Roadmap](../LEARNING.md) | [Next: Core Modules ➡️](../02-core-modules/README.md)
