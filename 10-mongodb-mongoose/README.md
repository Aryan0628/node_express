# Module 10: MongoDB + Mongoose

## What is MongoDB?

MongoDB is a **NoSQL, document-based database**. Instead of tables and rows (SQL), it uses **collections** and **documents** (JSON-like objects).

```
SQL Database          →  MongoDB
Database              →  Database
Table                 →  Collection
Row                   →  Document
Column                →  Field
Primary Key           →  _id (auto-generated ObjectId)
JOIN                  →  $lookup / populate()
```

### SQL vs NoSQL ⭐

| Feature | SQL (MySQL, PostgreSQL) | NoSQL (MongoDB) |
|---------|------------------------|-----------------|
| Structure | Fixed schema (tables, rows) | Flexible schema (documents) |
| Data format | Rows & columns | JSON-like (BSON) |
| Relationships | JOINs | Embedded docs or references |
| Scaling | Vertical (bigger server) | Horizontal (more servers) |
| Best for | Complex relationships, transactions | Flexible data, rapid development |
| Schema | Must define before inserting | Can insert anything |
| Examples | Banks, ERP systems | Social media, IoT, CMS |

### MongoDB Document Example:
```json
{
  "_id": "64a1b2c3d4e5f6a7b8c9d0e1",
  "name": "Aryan",
  "email": "aryan@test.com",
  "age": 22,
  "hobbies": ["coding", "gaming"],
  "address": {
    "city": "Delhi",
    "state": "Delhi"
  },
  "createdAt": "2024-01-15T10:30:00Z"
}
```

---

## What is Mongoose?

Mongoose is an **ODM (Object Data Modeling)** library for MongoDB. It provides:
- **Schemas** — define structure for documents
- **Validation** — ensure data is correct before saving
- **Type casting** — auto-converts types
- **Query building** — fluent API for queries
- **Middleware** — pre/post hooks

```bash
npm install mongoose
```

---

## Connecting to MongoDB

```js
// config/db.js
const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGODB_URI, {
      // These options are no longer needed in Mongoose 6+
      // but you might see them in tutorials:
      // useNewUrlParser: true,
      // useUnifiedTopology: true,
    });
    
    console.log(`MongoDB Connected: ${conn.connection.host}`);
  } catch (err) {
    console.error(`Error: ${err.message}`);
    process.exit(1);
  }
};

module.exports = connectDB;
```

```js
// server.js
const connectDB = require('./config/db');
require('dotenv').config();

connectDB();

// ... rest of your Express app
```

```env
# .env
MONGODB_URI=mongodb://localhost:27017/myapp
# OR for MongoDB Atlas (cloud):
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/myapp
```

---

## Schema & Model ⭐

### Schema = Structure definition. Model = Interacts with database.

```js
// models/User.js
const mongoose = require('mongoose');

// 1. Define Schema
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Name is required'],
    trim: true,
    minlength: [2, 'Name must be at least 2 characters'],
    maxlength: [50, 'Name cannot exceed 50 characters'],
  },
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    trim: true,
    match: [/^\S+@\S+\.\S+$/, 'Please provide a valid email'],
  },
  password: {
    type: String,
    required: [true, 'Password is required'],
    minlength: [6, 'Password must be at least 6 characters'],
    select: false,  // Don't return password in queries by default
  },
  age: {
    type: Number,
    min: [0, 'Age cannot be negative'],
    max: [150, 'Age seems too high'],
  },
  role: {
    type: String,
    enum: ['user', 'admin', 'moderator'],
    default: 'user',
  },
  isActive: {
    type: Boolean,
    default: true,
  },
  hobbies: [String],  // Array of strings
  address: {
    city: String,
    state: String,
    zipCode: String,
  },
  profilePicture: {
    type: String,
    default: 'default.jpg',
  },
}, {
  timestamps: true,  // Adds createdAt and updatedAt automatically
});

// 2. Create Model
const User = mongoose.model('User', userSchema);
// 'User' → collection name will be 'users' (lowercase, plural)

module.exports = User;
```

### Schema Types:
```js
String, Number, Boolean, Date, Buffer, ObjectId, Array, Mixed, Decimal128, Map
```

### Common Schema Options:
| Option | What it does |
|--------|-------------|
| `required: true` | Field must be provided |
| `unique: true` | No duplicates allowed |
| `default: value` | Default value if not provided |
| `enum: [...]` | Value must be from this list |
| `min / max` | Min/max for numbers |
| `minlength / maxlength` | Min/max for strings |
| `trim: true` | Remove whitespace |
| `lowercase: true` | Convert to lowercase |
| `select: false` | Exclude from queries by default |
| `match: /regex/` | Must match regex pattern |
| `timestamps: true` | Auto add `createdAt`/`updatedAt` |

---

## CRUD Operations ⭐

### CREATE

```js
// Method 1: Model.create()
const user = await User.create({
  name: 'Aryan',
  email: 'aryan@test.com',
  password: 'hashedpassword',
});

// Method 2: new Model() + .save()
const user = new User({
  name: 'Aryan',
  email: 'aryan@test.com',
  password: 'hashedpassword',
});
await user.save();

// Method 3: Insert many
const users = await User.insertMany([
  { name: 'User1', email: 'u1@test.com', password: '123456' },
  { name: 'User2', email: 'u2@test.com', password: '123456' },
]);
```

### READ

```js
// Find ALL
const users = await User.find();

// Find with FILTER
const admins = await User.find({ role: 'admin' });
const activeUsers = await User.find({ isActive: true, age: { $gte: 18 } });

// Find ONE
const user = await User.findOne({ email: 'aryan@test.com' });

// Find by ID
const user = await User.findById('64a1b2c3d4e5f6a7b8c9d0e1');

// Select specific fields
const user = await User.findById(id).select('name email -_id');
// Returns only name and email, excludes _id

// Sorting
const users = await User.find().sort({ createdAt: -1 }); // newest first
const users = await User.find().sort('-createdAt');        // same thing
const users = await User.find().sort('name');              // A-Z

// Pagination
const page = 2;
const limit = 10;
const users = await User.find()
  .skip((page - 1) * limit)
  .limit(limit);

// Count documents
const count = await User.countDocuments({ role: 'admin' });

// Check if exists
const exists = await User.exists({ email: 'aryan@test.com' });
```

### Query Operators:
```js
// Comparison
{ age: { $gt: 18 } }     // Greater than
{ age: { $gte: 18 } }    // Greater than or equal
{ age: { $lt: 65 } }     // Less than
{ age: { $lte: 65 } }    // Less than or equal
{ age: { $ne: 25 } }     // Not equal
{ role: { $in: ['admin', 'moderator'] } }   // In array
{ role: { $nin: ['banned'] } }              // Not in array

// Logical
{ $and: [{ age: { $gte: 18 } }, { role: 'admin' }] }
{ $or: [{ role: 'admin' }, { role: 'moderator' }] }

// Text search
{ name: { $regex: 'aryan', $options: 'i' } }  // Case-insensitive search
```

### UPDATE

```js
// Find and update (returns the updated document)
const user = await User.findByIdAndUpdate(
  id,
  { name: 'New Name', age: 25 },
  { 
    new: true,          // Return updated doc (default returns old)
    runValidators: true // Run schema validations on update
  }
);

// Find one and update
const user = await User.findOneAndUpdate(
  { email: 'aryan@test.com' },  // filter
  { $set: { name: 'Aryan G' } },  // update
  { new: true, runValidators: true }
);

// Update one (doesn't return the document)
await User.updateOne(
  { _id: id },
  { $set: { isActive: false } }
);

// Update many
await User.updateMany(
  { role: 'user' },
  { $set: { isActive: false } }
);

// Update operators
{ $set: { name: 'New' } }       // Set a field
{ $unset: { age: 1 } }          // Remove a field
{ $inc: { age: 1 } }            // Increment
{ $push: { hobbies: 'cooking' } }  // Add to array
{ $pull: { hobbies: 'gaming' } }   // Remove from array
```

### DELETE

```js
// Find and delete (returns deleted doc)
const user = await User.findByIdAndDelete(id);

// Delete one
await User.deleteOne({ _id: id });

// Delete many
await User.deleteMany({ isActive: false });
```

---

## Mongoose Middleware (Hooks) ⭐

Run code **before** or **after** certain operations.

```js
const userSchema = new mongoose.Schema({ ... });

// PRE middleware — runs BEFORE the operation
userSchema.pre('save', async function(next) {
  // 'this' refers to the document being saved
  
  // Hash password before saving
  if (this.isModified('password')) {
    this.password = await bcrypt.hash(this.password, 10);
  }
  
  next();
});

// POST middleware — runs AFTER the operation
userSchema.post('save', function(doc, next) {
  console.log(`User ${doc.name} was saved`);
  next();
});

// Pre-find middleware
userSchema.pre(/^find/, function(next) {
  // 'this' refers to the query
  // Automatically exclude inactive users from all find queries
  this.find({ isActive: { $ne: false } });
  next();
});

// Pre-delete middleware
userSchema.pre('findOneAndDelete', async function(next) {
  // Clean up related data before deleting
  const user = await this.model.findOne(this.getFilter());
  await Post.deleteMany({ author: user._id });
  next();
});
```

---

## `populate()` — References Between Collections ⭐

Instead of embedding everything, you can **reference** other documents.

```js
// models/Post.js
const postSchema = new mongoose.Schema({
  title: String,
  content: String,
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',       // References the User model
    required: true,
  },
  comments: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Comment',
  }],
});

const Post = mongoose.model('Post', postSchema);
```

```js
// Without populate — just returns the ObjectId
const post = await Post.findById(postId);
// { title: 'My Post', author: '64a1b2c3...' }  ← just an ID

// With populate — replaces ID with actual document
const post = await Post.findById(postId).populate('author');
// { 
//   title: 'My Post', 
//   author: { _id: '64a1b2c3...', name: 'Aryan', email: 'aryan@test.com' }
// }

// Populate specific fields only
const post = await Post.findById(postId)
  .populate('author', 'name email');  // Only get name and email

// Populate multiple fields
const post = await Post.findById(postId)
  .populate('author', 'name')
  .populate('comments');

// Nested populate
const post = await Post.findById(postId)
  .populate({
    path: 'comments',
    populate: { path: 'user', select: 'name' }
  });
```

### Embedding vs Referencing:
```
Embedding (nested documents):
✅ Faster reads (one query gets everything)
❌ Data duplication
❌ Hard to update shared data
→ Use for: data that's always accessed together

Referencing (ObjectId + populate):
✅ No duplication
✅ Easy to update independently
❌ Slower (needs extra query/populate)
→ Use for: data accessed independently, many-to-many
```

---

## Virtuals — Computed Fields

Fields that don't exist in the database but are computed on the fly.

```js
const userSchema = new mongoose.Schema({
  firstName: String,
  lastName: String,
});

// Virtual property
userSchema.virtual('fullName').get(function() {
  return `${this.firstName} ${this.lastName}`;
});

// Make virtuals show up in JSON
userSchema.set('toJSON', { virtuals: true });
userSchema.set('toObject', { virtuals: true });

const user = await User.findById(id);
console.log(user.fullName);  // 'Aryan Gupta' — not stored in DB
```

---

## Aggregation Pipeline (Basics)

For complex data processing — more powerful than simple find queries.

```js
// Count users by role
const stats = await User.aggregate([
  { $group: { _id: '$role', count: { $sum: 1 } } },
]);
// [{ _id: 'admin', count: 5 }, { _id: 'user', count: 42 }]

// Average age by role
const avgAge = await User.aggregate([
  { $group: { _id: '$role', avgAge: { $avg: '$age' } } },
]);

// Full pipeline example
const results = await Order.aggregate([
  // Stage 1: Filter
  { $match: { status: 'completed' } },
  
  // Stage 2: Group and calculate
  { $group: {
    _id: '$userId',
    totalOrders: { $sum: 1 },
    totalSpent: { $sum: '$amount' },
    avgOrderValue: { $avg: '$amount' },
  }},
  
  // Stage 3: Sort
  { $sort: { totalSpent: -1 } },
  
  // Stage 4: Limit
  { $limit: 10 },
  
  // Stage 5: Select fields
  { $project: {
    _id: 0,
    userId: '$_id',
    totalOrders: 1,
    totalSpent: { $round: ['$totalSpent', 2] },
  }},
]);
```

### Common Aggregation Stages:
```
$match    → Filter documents (like WHERE)
$group    → Group by a field (like GROUP BY)
$sort     → Sort results
$limit    → Limit number of results
$skip     → Skip results (pagination)
$project  → Select/rename fields (like SELECT)
$lookup   → Join with another collection (like JOIN)
$unwind   → Deconstruct arrays
```

---

## 🎯 Interview Answers

### Q: What is Mongoose and why use it with MongoDB?
> Mongoose is an ODM (Object Data Modeling) library for MongoDB in Node.js. It provides schema definitions to enforce data structure, built-in validation, type casting, query building, middleware (hooks), and population of referenced documents. While you can use the native MongoDB driver directly, Mongoose adds a layer of structure and convenience that makes development faster and more maintainable.

### Q: What is a Schema in Mongoose?
> A Schema defines the structure of documents in a MongoDB collection — the fields, their types, validations, and defaults. It acts as a blueprint. For example, you can define that a `name` field is a required String with min/max length. The Schema is then compiled into a Model using `mongoose.model()`, which provides methods to interact with the database.

### Q: Explain `populate()` in Mongoose.
> `populate()` replaces an ObjectId reference in a document with the actual referenced document from another collection. For example, if a Post has `author: ObjectId("...")`, calling `Post.findById(id).populate('author')` replaces the ObjectId with the full User document. It's similar to a JOIN in SQL. You can populate specific fields with `.populate('author', 'name email')`.

### Q: How do you add validation in Mongoose?
> Validation is defined in the Schema. Built-in validators include `required`, `unique`, `min/max` (numbers), `minlength/maxlength` (strings), `enum` (allowed values), and `match` (regex). You can also add custom validators using the `validate` property. Validation runs automatically on `save()` and can be enabled for updates with `runValidators: true`.

### Q: What are pre/post hooks in Mongoose?
> Pre/post hooks (middleware) let you run code before or after specific operations like `save`, `find`, `delete`. Common use: hashing passwords in a `pre('save')` hook before storing them, logging in `post('save')`, or cleaning up related data in `pre('findOneAndDelete')`. They're defined on the schema using `schema.pre()` and `schema.post()`.

### Q: SQL vs NoSQL — when to use which?
> Use SQL when you need complex relationships, ACID transactions, and a fixed schema — like banking systems or ERP. Use NoSQL (MongoDB) when you need flexible schemas, horizontal scaling, rapid development, and your data is document-oriented — like social media, content management, or real-time analytics. In MERN stack, MongoDB pairs naturally with JavaScript/JSON on the frontend and backend.

---

[⬅️ Module 9: Error Handling](../09-error-handling/README.md) | [Next: Authentication ➡️](../11-auth/README.md)
