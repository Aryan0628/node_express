# Module 2: Node.js Core Modules

Node.js comes with built-in modules — no need to install them. Just `require()` them.

---

## `fs` — File System ⭐

The `fs` module lets you work with files: read, write, delete, rename, etc.

### Async (Non-Blocking) — Always prefer this

```js
const fs = require('fs');

// READ a file
fs.readFile('example.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('Error reading file:', err.message);
    return;
  }
  console.log(data);
});

// WRITE to a file (creates or overwrites)
fs.writeFile('output.txt', 'Hello World!', (err) => {
  if (err) throw err;
  console.log('File written successfully');
});

// APPEND to a file
fs.appendFile('output.txt', '\nNew line added', (err) => {
  if (err) throw err;
  console.log('Content appended');
});

// DELETE a file
fs.unlink('output.txt', (err) => {
  if (err) throw err;
  console.log('File deleted');
});

// CHECK if file exists
fs.access('example.txt', fs.constants.F_OK, (err) => {
  console.log(err ? 'File does not exist' : 'File exists');
});

// CREATE a directory
fs.mkdir('newFolder', { recursive: true }, (err) => {
  if (err) throw err;
  console.log('Directory created');
});

// READ directory contents
fs.readdir('./', (err, files) => {
  if (err) throw err;
  console.log(files);  // ['app.js', 'package.json', ...]
});
```

### Sync (Blocking) — Use only for startup scripts, config loading

```js
const fs = require('fs');

const data = fs.readFileSync('example.txt', 'utf8');
console.log(data);

fs.writeFileSync('output.txt', 'Hello Sync!');
```

### Promise-based (Modern way)

```js
const fs = require('fs').promises;
// OR
const fs = require('fs/promises');  // Node 14+

async function readMyFile() {
  try {
    const data = await fs.readFile('example.txt', 'utf8');
    console.log(data);
  } catch (err) {
    console.error('Error:', err.message);
  }
}

readMyFile();
```

### 🔑 Interview Tip
> Always use **async** methods in production. Sync methods block the entire event loop.

---

## `path` — Path Utilities ⭐

The `path` module helps you work with file and directory paths **across operating systems**.

```js
const path = require('path');

// Join paths (handles slashes for you)
path.join('/users', 'aryan', 'documents', 'file.txt');
// → '/users/aryan/documents/file.txt'

path.join(__dirname, 'data', 'users.json');
// → '/absolute/path/to/your/project/data/users.json'

// Resolve to absolute path
path.resolve('file.txt');
// → '/absolute/path/to/current/dir/file.txt'

// Get file extension
path.extname('image.png');     // → '.png'
path.extname('app.test.js');   // → '.js'

// Get filename
path.basename('/path/to/file.txt');        // → 'file.txt'
path.basename('/path/to/file.txt', '.txt'); // → 'file'

// Get directory name
path.dirname('/path/to/file.txt');  // → '/path/to'

// Parse a path into components
path.parse('/home/aryan/app.js');
// {
//   root: '/',
//   dir: '/home/aryan',
//   base: 'app.js',
//   ext: '.js',
//   name: 'app'
// }
```

### `__dirname` and `__filename`

```js
console.log(__filename);  // Full path of current file
// → '/Users/aryan/project/app.js'

console.log(__dirname);   // Directory of current file
// → '/Users/aryan/project'

// Common usage: construct paths relative to current file
const dataPath = path.join(__dirname, 'data', 'users.json');
```

> ⚠️ `__dirname` and `__filename` are NOT available in ES Modules. Use `import.meta.url` instead.

---

## `http` — Creating an HTTP Server ⭐

This is how you create a web server **without Express**. Interviewers love asking this.

```js
const http = require('http');

const server = http.createServer((req, res) => {
  // req = request object (what the client sent)
  // res = response object (what we send back)
  
  console.log(req.method);  // 'GET', 'POST', etc.
  console.log(req.url);     // '/', '/about', '/api/users'
  
  // Set response headers
  res.setHeader('Content-Type', 'application/json');
  
  // Simple routing
  if (req.url === '/' && req.method === 'GET') {
    res.statusCode = 200;
    res.end(JSON.stringify({ message: 'Home Page' }));
  } else if (req.url === '/about' && req.method === 'GET') {
    res.statusCode = 200;
    res.end(JSON.stringify({ message: 'About Page' }));
  } else {
    res.statusCode = 404;
    res.end(JSON.stringify({ message: 'Not Found' }));
  }
});

const PORT = 3000;
server.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

### Why Express is better than raw `http`:
- Manual routing with `if/else` is painful
- No built-in body parsing
- No middleware system
- No template engine support
- No static file serving

---

## `events` — EventEmitter ⭐

Node.js is **event-driven**. Many built-in modules (http, fs, streams) use the `EventEmitter` class.

```js
const EventEmitter = require('events');

// Create an emitter instance
const emitter = new EventEmitter();

// Register an event listener (subscriber)
emitter.on('orderPlaced', (order) => {
  console.log(`Processing order: ${order.id}`);
});

emitter.on('orderPlaced', (order) => {
  console.log(`Sending email for order: ${order.id}`);
});

// Emit the event (publisher)
emitter.emit('orderPlaced', { id: 123, item: 'Laptop' });

// Output:
// Processing order: 123
// Sending email for order: 123
```

### Creating your own class with EventEmitter

```js
const EventEmitter = require('events');

class Logger extends EventEmitter {
  log(message) {
    console.log(message);
    this.emit('messageLogged', { message, timestamp: new Date() });
  }
}

const logger = new Logger();

logger.on('messageLogged', (data) => {
  console.log('Event received:', data);
});

logger.log('Hello World');
```

### Useful EventEmitter methods:
```js
emitter.on('event', callback);       // Listen for event
emitter.once('event', callback);     // Listen only ONCE
emitter.off('event', callback);      // Remove specific listener
emitter.removeAllListeners('event'); // Remove all listeners for event
emitter.listenerCount('event');      // Count listeners
```

---

## `os` — Operating System Info

```js
const os = require('os');

console.log(os.platform());    // 'darwin', 'linux', 'win32'
console.log(os.arch());        // 'x64', 'arm64'
console.log(os.cpus().length); // Number of CPU cores
console.log(os.freemem());     // Free memory in bytes
console.log(os.totalmem());    // Total memory in bytes
console.log(os.homedir());     // Home directory path
console.log(os.hostname());    // Computer name
console.log(os.uptime());      // System uptime in seconds
```

---

## `stream` — Streams (Concept)

Streams let you process data **piece by piece** instead of loading everything into memory. Essential for large files.

### Types of Streams:
1. **Readable** — reading data (e.g., `fs.createReadStream`)
2. **Writable** — writing data (e.g., `fs.createWriteStream`)
3. **Duplex** — both read and write (e.g., TCP socket)
4. **Transform** — modify data as it passes through (e.g., compression)

### Example: Reading a large file with streams

```js
const fs = require('fs');

// ❌ BAD for large files (loads entire file into memory)
fs.readFile('huge-file.txt', 'utf8', (err, data) => {
  console.log(data);  // Could be 2GB in memory!
});

// ✅ GOOD — streams process it in chunks
const readStream = fs.createReadStream('huge-file.txt', 'utf8');

readStream.on('data', (chunk) => {
  console.log('Received chunk:', chunk.length, 'characters');
});

readStream.on('end', () => {
  console.log('Done reading file');
});

readStream.on('error', (err) => {
  console.error('Error:', err.message);
});
```

### Piping streams (connecting output to input)

```js
const fs = require('fs');

const readStream = fs.createReadStream('input.txt');
const writeStream = fs.createWriteStream('output.txt');

// Pipe: read from input.txt → write to output.txt
readStream.pipe(writeStream);

// Real-world: sending a file as HTTP response
const http = require('http');
const server = http.createServer((req, res) => {
  const stream = fs.createReadStream('big-video.mp4');
  stream.pipe(res);  // Stream directly to client without loading in memory
});
```

---

## `url` — URL Parsing

```js
const url = require('url');

const myUrl = new URL('https://example.com:8080/api/users?page=2&limit=10#section1');

console.log(myUrl.hostname);     // 'example.com'
console.log(myUrl.port);         // '8080'
console.log(myUrl.pathname);     // '/api/users'
console.log(myUrl.search);       // '?page=2&limit=10'
console.log(myUrl.hash);         // '#section1'
console.log(myUrl.searchParams.get('page'));   // '2'
console.log(myUrl.searchParams.get('limit'));  // '10'
```

---

## `crypto` — Hashing & Encryption

```js
const crypto = require('crypto');

// Create a hash (one-way — cannot reverse)
const hash = crypto.createHash('sha256').update('myPassword123').digest('hex');
console.log(hash);
// → 'a1b2c3d4...' (64 character hex string)

// Generate random bytes (for tokens, secrets)
const token = crypto.randomBytes(32).toString('hex');
console.log(token);
// → 'f4a8b2c1...' (64 character random string)

// UUID generation (Node 19+)
const uuid = crypto.randomUUID();
console.log(uuid);
// → '550e8400-e29b-41d4-a716-446655440000'
```

> In real apps, use **bcrypt** for password hashing (it includes salting). `crypto` is more for general-purpose hashing and token generation.

---

## 🎯 Interview Answers

### Q: How do you create an HTTP server in Node.js without Express?
> Use the built-in `http` module. Call `http.createServer()` which takes a callback with `req` (request) and `res` (response) objects. Inside the callback, you can check `req.url` and `req.method` for routing, set response headers with `res.setHeader()`, and send responses with `res.end()`. Finally, call `server.listen(port)` to start listening.

### Q: What are Streams in Node.js? Why are they useful?
> Streams are objects that let you read or write data continuously in chunks, rather than loading the entire data into memory at once. They're useful for processing large files (like videos or logs) because they keep memory usage low. There are 4 types: Readable, Writable, Duplex, and Transform. Streams can be connected using `.pipe()` to chain operations.

### Q: What is the EventEmitter class?
> EventEmitter is a class in the `events` module that implements the Observer pattern. Objects can emit named events and other objects can listen for those events using `.on()`. Many core Node.js modules inherit from EventEmitter — like HTTP servers (emit 'request' events), streams (emit 'data' events), etc. You can create custom classes that extend EventEmitter to build event-driven architectures.

### Q: Difference between `fs.readFile` and `fs.readFileSync`?
> `fs.readFile` is **asynchronous** — it reads the file in the background and calls a callback when done, without blocking the event loop. `fs.readFileSync` is **synchronous** — it blocks the entire event loop until the file is fully read. Always use async methods in production to keep the server responsive. Sync methods are only acceptable during startup (like reading config files).

---

[⬅️ Module 1: Fundamentals](../01-node-fundamentals/README.md) | [Next: npm & package.json ➡️](../03-npm-basics/README.md)
