# Module 15: Async Patterns in Node.js

## Evolution of Async in JavaScript

```
Callbacks (old) → Promises (ES6) → async/await (ES8, modern)
```

---

## 1. Callbacks

A function passed as an argument, called when the async operation completes.

```js
// Node.js style: callback(error, result)
const fs = require('fs');

fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('Error:', err);
    return;
  }
  console.log(data);
});
```

### Callback Hell 😱

When you need to do multiple async operations in sequence:

```js
// ❌ CALLBACK HELL — nested, unreadable, unmaintainable
getUser(userId, (err, user) => {
  if (err) return handleError(err);
  
  getOrders(user.id, (err, orders) => {
    if (err) return handleError(err);
    
    getOrderDetails(orders[0].id, (err, details) => {
      if (err) return handleError(err);
      
      getShippingInfo(details.shippingId, (err, shipping) => {
        if (err) return handleError(err);
        
        console.log(shipping);
        // 4 levels deep... and it gets worse
      });
    });
  });
});
```

---

## 2. Promises

A Promise represents a value that will be available in the future.

### States:
```
Pending   → Operation in progress
Fulfilled → Operation completed successfully (resolved)
Rejected  → Operation failed (rejected)
```

```js
// Creating a Promise
const myPromise = new Promise((resolve, reject) => {
  const success = true;
  
  setTimeout(() => {
    if (success) {
      resolve({ data: 'Hello!' });  // Success
    } else {
      reject(new Error('Something failed'));  // Failure
    }
  }, 1000);
});

// Using a Promise
myPromise
  .then((result) => {
    console.log(result);  // { data: 'Hello!' }
    return 'next value';
  })
  .then((value) => {
    console.log(value);   // 'next value'
  })
  .catch((err) => {
    console.error(err);   // Only if rejected
  })
  .finally(() => {
    console.log('Always runs');  // Cleanup code
  });
```

### Fixing Callback Hell with Promises:
```js
// ✅ Promise chain — flat and readable
getUser(userId)
  .then(user => getOrders(user.id))
  .then(orders => getOrderDetails(orders[0].id))
  .then(details => getShippingInfo(details.shippingId))
  .then(shipping => console.log(shipping))
  .catch(err => console.error(err));  // One error handler for all!
```

---

## 3. async/await ⭐ (What You Should Use)

Syntactic sugar over Promises. Makes async code look synchronous.

```js
// ✅ async/await — clean, readable, modern
async function getShipping(userId) {
  try {
    const user = await getUser(userId);
    const orders = await getOrders(user.id);
    const details = await getOrderDetails(orders[0].id);
    const shipping = await getShippingInfo(details.shippingId);
    
    console.log(shipping);
    return shipping;
  } catch (err) {
    console.error('Error:', err);
  }
}

getShipping(123);
```

### Rules:
1. `await` can ONLY be used inside an `async` function
2. `async` function always returns a Promise
3. `await` pauses execution until the Promise resolves

```js
// async function always returns a Promise
async function greet() {
  return 'Hello';  // Same as: return Promise.resolve('Hello')
}

greet().then(msg => console.log(msg));  // 'Hello'

// await pauses until Promise resolves
async function getData() {
  console.log('1. Before');
  const data = await fetchData();  // Pauses here
  console.log('2. After');         // Runs after fetchData resolves
  return data;
}
```

### Error Handling with async/await:
```js
// Method 1: try-catch
async function getUser(id) {
  try {
    const user = await User.findById(id);
    if (!user) throw new Error('User not found');
    return user;
  } catch (err) {
    console.error(err);
    throw err;  // Re-throw if needed
  }
}

// Method 2: .catch() on the function call
getUser(123).catch(err => console.error(err));

// Method 3: Wrapper (used in Express — see Module 9)
const catchAsync = fn => (req, res, next) => fn(req, res, next).catch(next);
```

---

## Promise Methods ⭐

### `Promise.all()` — Run in parallel, fail if ANY fails

```js
// All promises run simultaneously
const [users, products, orders] = await Promise.all([
  User.find(),        // These 3 queries run at the SAME TIME
  Product.find(),     // Not one after another
  Order.find(),
]);

// If ANY one fails, the entire Promise.all rejects
// Use when all results are needed
```

### `Promise.allSettled()` — Run in parallel, get ALL results

```js
const results = await Promise.allSettled([
  fetchUserData(),     // might fail
  fetchOrderHistory(), // might fail
  fetchNotifications(),// might fail
]);

// Returns ALL results regardless of success/failure
results.forEach((result) => {
  if (result.status === 'fulfilled') {
    console.log('Success:', result.value);
  } else {
    console.log('Failed:', result.reason);  // The error
  }
});

// Use when you want results from everything, even if some fail
```

### `Promise.race()` — First to finish wins

```js
// Returns the result of the FIRST promise to resolve or reject
const result = await Promise.race([
  fetch('https://api1.com/data'),  // Whoever responds first wins
  fetch('https://api2.com/data'),
]);

// Use case: timeout
const fetchWithTimeout = (url, timeout = 5000) => {
  return Promise.race([
    fetch(url),
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('Request timed out')), timeout)
    ),
  ]);
};
```

### `Promise.any()` — First SUCCESS wins

```js
// Returns the first promise to RESOLVE (ignores rejections)
const result = await Promise.any([
  fetch('https://api1.com/data'),  // fails
  fetch('https://api2.com/data'),  // succeeds → this wins
  fetch('https://api3.com/data'),  // succeeds but slower
]);

// Only rejects if ALL promises reject
```

### Comparison:

| Method | Resolves when | Rejects when |
|--------|--------------|-------------|
| `Promise.all()` | ALL resolve | ANY rejects |
| `Promise.allSettled()` | ALL settle (resolve or reject) | Never |
| `Promise.race()` | First to settle (resolve OR reject) | First to reject |
| `Promise.any()` | First to resolve | ALL reject |

---

## Practical Examples

### Sequential vs Parallel execution:

```js
// ❌ SEQUENTIAL — slow (one after another)
const users = await User.find();        // 200ms
const products = await Product.find();  // 300ms
const orders = await Order.find();      // 250ms
// Total: 750ms

// ✅ PARALLEL — fast (all at once)
const [users, products, orders] = await Promise.all([
  User.find(),        // 200ms ─┐
  Product.find(),     // 300ms ─┼── All run simultaneously
  Order.find(),       // 250ms ─┘
]);
// Total: 300ms (slowest one)
```

### When to use sequential:
```js
// When each step depends on the previous one
const user = await User.findById(userId);
const orders = await Order.find({ userId: user._id });  // needs user._id
const details = await OrderDetail.findById(orders[0].detailId);  // needs order
```

---

## 🎯 Interview Answers

### Q: What is callback hell and how do you avoid it?
> Callback hell is deeply nested callbacks that make code unreadable and hard to maintain — it forms a "pyramid of doom." You avoid it by using Promises (`.then()` chains) or `async/await` (makes async code look synchronous). `async/await` is the modern preferred approach, combined with `try-catch` for error handling.

### Q: Explain `async/await` in Node.js.
> `async/await` is syntactic sugar over Promises. An `async` function always returns a Promise. The `await` keyword pauses execution until the Promise resolves, making asynchronous code look and behave like synchronous code. It's more readable than `.then()` chains. Errors are handled with `try-catch` blocks. `await` can only be used inside an `async` function.

### Q: What is the difference between `Promise.all()` and `Promise.allSettled()`?
> `Promise.all()` runs multiple promises in parallel and resolves when ALL succeed — but if ANY one rejects, the entire `Promise.all` rejects immediately. `Promise.allSettled()` also runs promises in parallel but always waits for ALL to complete (whether they resolved or rejected) and returns the result of each. Use `all()` when you need everything to succeed, and `allSettled()` when you want all results regardless of failures.

### Q: How does error handling work with `async/await`?
> Use `try-catch` blocks. Wrap `await` calls in `try` and handle errors in `catch`. In Express, instead of writing `try-catch` in every route, use a `catchAsync` wrapper function that catches rejected promises and passes them to the error-handling middleware via `next(error)`. Alternatively, the `express-async-errors` package automatically catches async errors.

---

[⬅️ Module 14: CORS & Security](../14-cors-security/README.md) | [Next: MVC Architecture ➡️](../16-mvc-architecture/README.md)
