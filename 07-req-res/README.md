# Module 7: Request & Response Deep Dive

## The Request Object (`req`)

The `req` object represents the HTTP request from the client.

### `req.body` — Request Body ⭐

```js
// ⚠️ You MUST add express.json() middleware first!
app.use(express.json());

app.post('/api/users', (req, res) => {
  console.log(req.body);
  // { name: 'Aryan', email: 'aryan@test.com' }
  
  const { name, email } = req.body;
  res.status(201).json({ name, email });
});
```

**Without `express.json()`:**
```js
app.post('/api/users', (req, res) => {
  console.log(req.body);  // undefined!
});
```

### For URL-encoded form data:
```js
app.use(express.urlencoded({ extended: true }));

// HTML form: <form method="POST">
//   <input name="username" value="aryan">
//   <input name="password" value="123">
// </form>

app.post('/login', (req, res) => {
  console.log(req.body);
  // { username: 'aryan', password: '123' }
});
```

### `extended: true` vs `extended: false`:
| Option | Library Used | Supports |
|--------|-------------|----------|
| `extended: true` | `qs` library | Nested objects, arrays |
| `extended: false` | `querystring` module | Simple key-value pairs only |

---

### `req.params` — Route Parameters

```js
// URL: GET /api/users/42
app.get('/api/users/:id', (req, res) => {
  req.params.id;  // '42' (string)
});

// URL: GET /api/users/42/posts/7
app.get('/api/users/:userId/posts/:postId', (req, res) => {
  req.params.userId;  // '42'
  req.params.postId;  // '7'
});
```

---

### `req.query` — Query String Parameters

```js
// URL: GET /api/products?category=phones&price_min=100&sort=price
app.get('/api/products', (req, res) => {
  req.query.category;   // 'phones'
  req.query.price_min;  // '100' (string!)
  req.query.sort;       // 'price'
  
  // Practical usage with defaults
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const sort = req.query.sort || 'createdAt';
});
```

---

### `req.headers` — Request Headers

```js
app.get('/api/data', (req, res) => {
  console.log(req.headers);
  // {
  //   'host': 'localhost:3000',
  //   'user-agent': 'Mozilla/5.0...',
  //   'content-type': 'application/json',
  //   'authorization': 'Bearer eyJhbGc...',
  //   'accept': 'application/json'
  // }
  
  // Access specific headers (always lowercase)
  const authHeader = req.headers.authorization;
  const contentType = req.headers['content-type'];
  
  // Shortcut for common headers
  const contentType2 = req.get('Content-Type');
});
```

---

### Other Useful `req` Properties

```js
app.use((req, res, next) => {
  console.log(req.method);       // 'GET', 'POST', etc.
  console.log(req.url);          // '/api/users?page=2'
  console.log(req.path);         // '/api/users' (without query string)
  console.log(req.originalUrl);  // '/api/users?page=2' (full URL)
  console.log(req.protocol);     // 'http' or 'https'
  console.log(req.hostname);     // 'localhost'
  console.log(req.ip);           // '127.0.0.1'
  console.log(req.secure);       // true if HTTPS
  console.log(req.fresh);        // true if response is still "fresh" in cache
  
  next();
});
```

---

## The Response Object (`res`)

The `res` object represents the HTTP response that Express sends to the client.

### Setting Status Codes

```js
// Chain status with response
res.status(200).json({ message: 'OK' });
res.status(201).json({ message: 'Created' });
res.status(400).json({ message: 'Bad Request' });
res.status(404).json({ message: 'Not Found' });
res.status(500).json({ message: 'Server Error' });

// Send status code only (with default status text)
res.sendStatus(200);  // Sends 'OK'
res.sendStatus(204);  // Sends '' (No Content)
res.sendStatus(404);  // Sends 'Not Found'
```

---

### Sending Responses

```js
// JSON response (most common for APIs)
res.json({ name: 'Aryan', age: 22 });

// Plain text / HTML
res.send('Hello World');
res.send('<h1>Hello</h1>');

// Send a file
res.sendFile(path.join(__dirname, 'public', 'index.html'));

// Download a file
res.download('./files/report.pdf');
res.download('./files/report.pdf', 'custom-filename.pdf');

// Redirect
res.redirect('/login');
res.redirect(301, '/new-url');  // Permanent redirect
res.redirect('back');           // Go back to previous page
```

---

### Setting Headers

```js
// Set a single header
res.set('Content-Type', 'application/json');
res.set('X-Custom-Header', 'my-value');

// Set multiple headers
res.set({
  'Content-Type': 'text/plain',
  'X-Request-Id': '12345',
  'Cache-Control': 'no-cache',
});

// Shorthand
res.type('json');  // Sets Content-Type to application/json
res.type('html');  // Sets Content-Type to text/html
```

---

### Working with Cookies

```bash
npm install cookie-parser
```

```js
const cookieParser = require('cookie-parser');
app.use(cookieParser());

// SET a cookie
app.get('/set-cookie', (req, res) => {
  res.cookie('username', 'aryan', {
    maxAge: 24 * 60 * 60 * 1000,  // 1 day in ms
    httpOnly: true,                 // Not accessible via JS
    secure: true,                   // Only sent over HTTPS
    sameSite: 'strict',            // CSRF protection
  });
  res.json({ message: 'Cookie set' });
});

// READ cookies
app.get('/get-cookie', (req, res) => {
  console.log(req.cookies);           // { username: 'aryan' }
  console.log(req.cookies.username);  // 'aryan'
  res.json({ cookies: req.cookies });
});

// DELETE a cookie
app.get('/clear-cookie', (req, res) => {
  res.clearCookie('username');
  res.json({ message: 'Cookie cleared' });
});
```

---

## HTTP Status Codes Cheat Sheet ⭐

### When to use each:

```js
// ✅ SUCCESS
res.status(200).json(data);                          // GET succeeded
res.status(201).json(newResource);                   // POST created resource
res.status(204).send();                              // DELETE succeeded (no body)

// ❌ CLIENT ERRORS
res.status(400).json({ message: 'Invalid input' });  // Validation failed
res.status(401).json({ message: 'Please login' });   // Not authenticated
res.status(403).json({ message: 'Not allowed' });    // Authenticated but forbidden
res.status(404).json({ message: 'Not found' });      // Resource doesn't exist
res.status(409).json({ message: 'Already exists' }); // Duplicate (e.g., email taken)
res.status(429).json({ message: 'Too many requests' }); // Rate limited

// 💥 SERVER ERRORS
res.status(500).json({ message: 'Server error' });   // Something broke
```

### Memory trick:
```
2xx = 😊 Everything is fine
3xx = 🔀 Go somewhere else (redirect)
4xx = 🤦 Client messed up
5xx = 💀 Server messed up
```

### 401 vs 403 (Common Interview Question):
```
401 Unauthorized = "Who are you?" → Not logged in / invalid token
403 Forbidden    = "I know who you are, but you can't do this" → No permission
```

---

## Consistent API Response Format

Always use a consistent structure:

```js
// ✅ GOOD — Consistent structure
// Success response
{
  "success": true,
  "data": { ... },
  "message": "Users fetched successfully"
}

// Error response
{
  "success": false,
  "message": "User not found",
  "error": "NOT_FOUND"
}

// List response with pagination
{
  "success": true,
  "count": 25,
  "totalPages": 3,
  "currentPage": 1,
  "data": [ ... ]
}
```

### Helper function for consistent responses:

```js
const sendSuccess = (res, data, message = 'Success', statusCode = 200) => {
  res.status(statusCode).json({
    success: true,
    message,
    data,
  });
};

const sendError = (res, message = 'Error', statusCode = 500) => {
  res.status(statusCode).json({
    success: false,
    message,
  });
};

// Usage
app.get('/api/users', (req, res) => {
  const users = [{ name: 'Aryan' }];
  sendSuccess(res, users, 'Users fetched');
});

app.get('/api/users/:id', (req, res) => {
  const user = null;
  if (!user) {
    return sendError(res, 'User not found', 404);
  }
  sendSuccess(res, user);
});
```

---

## 🎯 Interview Answers

### Q: How do you access the request body in Express?
> Use `req.body` to access the parsed request body. But first, you must add the `express.json()` middleware with `app.use(express.json())` for JSON bodies, or `express.urlencoded({ extended: true })` for form data. Without these middleware, `req.body` will be `undefined`.

### Q: Why do we need `express.json()` middleware?
> By default, Express does not parse the request body — `req.body` is `undefined`. `express.json()` is a built-in middleware that parses incoming requests with JSON payloads. It reads the raw body stream, parses the JSON, and sets the result on `req.body`. Similarly, `express.urlencoded()` parses URL-encoded form data.

### Q: Name 5 HTTP status codes and when to use them.
> **200 OK** — successful GET request. **201 Created** — resource successfully created after POST. **400 Bad Request** — client sent invalid data (validation failure). **401 Unauthorized** — authentication required or token invalid. **404 Not Found** — requested resource doesn't exist. **500 Internal Server Error** — unexpected server-side failure.

---

[⬅️ Module 6: Middleware](../06-middleware/README.md) | [Next: REST API Design ➡️](../08-rest-api/README.md)
