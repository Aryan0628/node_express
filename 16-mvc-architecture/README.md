# Module 16: MVC Architecture

## What is MVC?

**MVC** = **M**odel - **V**iew - **C**ontroller

A design pattern that separates your application into 3 layers:

```
Client Request
      ↓
   Routes      → "Which controller handles this?"
      ↓
  Controller   → Business logic, validation, orchestration
      ↓
    Model       → Data layer (Mongoose schemas, DB operations)
      ↓
  Controller   → Formats response
      ↓
   Response    → JSON (for APIs) or View/Template (for SSR)
```

| Layer | Responsibility | Example |
|-------|---------------|---------|
| **Model** | Data structure, DB operations, validation | Mongoose schemas |
| **View** | Presentation (HTML/templates OR JSON for APIs) | React frontend / res.json() |
| **Controller** | Business logic, connects Model and View | Route handler functions |

> In MERN stack, React IS the View. Express just sends JSON. So it's more like **MC** (Model-Controller) on the backend.

---

## Project Structure ⭐

```
project/
├── config/
│   └── db.js                # Database connection
├── controllers/
│   ├── authController.js    # Auth business logic
│   ├── userController.js    # User business logic
│   └── productController.js # Product business logic
├── middleware/
│   ├── auth.js              # Authentication middleware
│   ├── errorHandler.js      # Centralized error handler
│   └── validate.js          # Input validation middleware
├── models/
│   ├── User.js              # User schema & model
│   ├── Product.js           # Product schema & model
│   └── Order.js             # Order schema & model
├── routes/
│   ├── authRoutes.js        # Auth routes
│   ├── userRoutes.js        # User routes
│   └── productRoutes.js     # Product routes
├── utils/
│   ├── AppError.js          # Custom error class
│   └── catchAsync.js        # Async error wrapper
├── .env                     # Environment variables
├── .gitignore
├── app.js                   # Express app setup (middleware, routes)
├── server.js                # Entry point (DB connect + start server)
└── package.json
```

---

## Why Separate `app.js` and `server.js`? ⭐

```js
// app.js — Express configuration (testable, importable)
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');

const authRoutes = require('./routes/authRoutes');
const userRoutes = require('./routes/userRoutes');
const productRoutes = require('./routes/productRoutes');
const errorHandler = require('./middleware/errorHandler');

const app = express();

// Middleware
app.use(helmet());
app.use(cors());
app.use(morgan('dev'));
app.use(express.json());

// Routes
app.use('/api/auth', authRoutes);
app.use('/api/users', userRoutes);
app.use('/api/products', productRoutes);

// 404 Handler
app.use((req, res) => {
  res.status(404).json({ message: `Route ${req.originalUrl} not found` });
});

// Error Handler
app.use(errorHandler);

module.exports = app;  // Export app (for testing)
```

```js
// server.js — Entry point (connects DB, starts server)
require('dotenv').config();
const app = require('./app');
const connectDB = require('./config/db');

// Connect to database
connectDB();

const PORT = process.env.PORT || 5000;

const server = app.listen(PORT, () => {
  console.log(`Server running in ${process.env.NODE_ENV} mode on port ${PORT}`);
});

// Handle unhandled promise rejections
process.on('unhandledRejection', (err) => {
  console.error(`Error: ${err.message}`);
  server.close(() => process.exit(1));
});
```

### Why separate?
- **app.js** can be imported in tests without starting the server
- **server.js** handles server lifecycle (start, shutdown, crash handling)
- Clean separation of concerns

---

## Full MVC Example

### Model

```js
// models/Product.js
const mongoose = require('mongoose');

const productSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Product name is required'],
    trim: true,
  },
  price: {
    type: Number,
    required: [true, 'Price is required'],
    min: [0, 'Price cannot be negative'],
  },
  description: String,
  category: {
    type: String,
    enum: ['electronics', 'clothing', 'books', 'food'],
    required: true,
  },
  inStock: {
    type: Boolean,
    default: true,
  },
  createdBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true,
  },
}, { timestamps: true });

module.exports = mongoose.model('Product', productSchema);
```

### Controller

```js
// controllers/productController.js
const Product = require('../models/Product');
const AppError = require('../utils/AppError');
const catchAsync = require('../utils/catchAsync');

// GET /api/products
exports.getAllProducts = catchAsync(async (req, res) => {
  const { category, sort, page = 1, limit = 10 } = req.query;
  
  // Build query
  const query = {};
  if (category) query.category = category;
  
  const products = await Product.find(query)
    .sort(sort || '-createdAt')
    .skip((page - 1) * limit)
    .limit(parseInt(limit))
    .populate('createdBy', 'name email');
  
  const total = await Product.countDocuments(query);
  
  res.json({
    success: true,
    count: products.length,
    total,
    page: parseInt(page),
    totalPages: Math.ceil(total / limit),
    data: products,
  });
});

// GET /api/products/:id
exports.getProduct = catchAsync(async (req, res) => {
  const product = await Product.findById(req.params.id)
    .populate('createdBy', 'name email');
  
  if (!product) {
    throw new AppError('Product not found', 404);
  }
  
  res.json({ success: true, data: product });
});

// POST /api/products
exports.createProduct = catchAsync(async (req, res) => {
  req.body.createdBy = req.user.id;  // From auth middleware
  
  const product = await Product.create(req.body);
  
  res.status(201).json({
    success: true,
    message: 'Product created',
    data: product,
  });
});

// PUT /api/products/:id
exports.updateProduct = catchAsync(async (req, res) => {
  const product = await Product.findByIdAndUpdate(
    req.params.id,
    req.body,
    { new: true, runValidators: true }
  );
  
  if (!product) {
    throw new AppError('Product not found', 404);
  }
  
  res.json({ success: true, data: product });
});

// DELETE /api/products/:id
exports.deleteProduct = catchAsync(async (req, res) => {
  const product = await Product.findByIdAndDelete(req.params.id);
  
  if (!product) {
    throw new AppError('Product not found', 404);
  }
  
  res.json({ success: true, message: 'Product deleted' });
});
```

### Routes

```js
// routes/productRoutes.js
const express = require('express');
const router = express.Router();
const {
  getAllProducts,
  getProduct,
  createProduct,
  updateProduct,
  deleteProduct,
} = require('../controllers/productController');
const { protect, authorize } = require('../middleware/auth');

router.route('/')
  .get(getAllProducts)                           // Public
  .post(protect, createProduct);                 // Auth required

router.route('/:id')
  .get(getProduct)                               // Public
  .put(protect, updateProduct)                   // Auth required
  .delete(protect, authorize('admin'), deleteProduct);  // Admin only

module.exports = router;
```

---

## Key Principles

### Separation of Concerns
Each file has ONE job:
- **Routes** → Define URL patterns and which controller handles them
- **Controllers** → Handle request/response logic
- **Models** → Define data structure and interact with DB
- **Middleware** → Reusable request processors

### Fat Models, Skinny Controllers
Put business logic in models (methods, hooks), keep controllers simple:

```js
// ❌ Fat controller — business logic in controller
exports.updatePrice = async (req, res) => {
  const product = await Product.findById(req.params.id);
  const discount = product.price * 0.1;
  product.price = product.price - discount;
  product.lastDiscountedAt = new Date();
  await product.save();
};

// ✅ Fat model — business logic in model
// In Product model:
productSchema.methods.applyDiscount = function(percentage) {
  this.price = this.price * (1 - percentage / 100);
  this.lastDiscountedAt = new Date();
  return this.save();
};

// In controller:
exports.updatePrice = async (req, res) => {
  const product = await Product.findById(req.params.id);
  await product.applyDiscount(10);
  res.json({ data: product });
};
```

---

## 🎯 Interview Answers

### Q: What is MVC architecture?
> MVC (Model-View-Controller) separates an application into three layers. The **Model** handles data and database operations (Mongoose schemas). The **View** handles presentation (in MERN, React is the view; Express just returns JSON). The **Controller** contains business logic, processes requests, interacts with the Model, and sends responses. This separation makes code organized, testable, and maintainable.

### Q: How do you structure an Express project?
> I use MVC pattern with dedicated folders: `models/` for Mongoose schemas, `controllers/` for business logic, `routes/` for URL definitions, `middleware/` for auth/error handling, `config/` for database connection, and `utils/` for helpers. I separate `app.js` (Express configuration and middleware) from `server.js` (database connection and server startup) so the app can be imported in tests without starting the server.

### Q: Why is separation of concerns important?
> It makes code easier to understand (each file has one job), easier to test (test controllers without starting the server), easier to maintain (change the database without touching routes), and enables team collaboration (different developers work on different layers). It also promotes code reuse — the same model methods work across multiple controllers.

---

[⬅️ Module 15: Async Patterns](../15-async-patterns/README.md) | [Next: Interview Questions ➡️](../17-interview-questions/README.md)
