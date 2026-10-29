# Module 12: File Upload & Static Files

## Serving Static Files

`express.static()` serves files (images, CSS, JS, HTML) from a directory.

```js
const express = require('express');
const path = require('path');
const app = express();

// Serve files from 'public' folder
app.use(express.static('public'));
// Files accessible as:
// public/style.css  →  http://localhost:3000/style.css
// public/logo.png   →  http://localhost:3000/logo.png

// With a URL prefix
app.use('/static', express.static('public'));
// public/style.css  →  http://localhost:3000/static/style.css

// Best practice: use absolute path
app.use(express.static(path.join(__dirname, 'public')));

// Serve multiple directories
app.use(express.static('public'));
app.use(express.static('uploads'));
```

### Folder structure:
```
project/
├── public/
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   └── script.js
│   └── images/
│       └── logo.png
├── uploads/        ← user uploaded files
└── server.js
```

---

## File Uploads with Multer ⭐

**Multer** is the standard middleware for handling file uploads in Express.

```bash
npm install multer
```

### Basic Setup

```js
const multer = require('multer');

// Simple setup (stores in memory)
const upload = multer({ dest: 'uploads/' });
// Files saved as random names in 'uploads/' folder
```

### Disk Storage (Control filename and destination)

```js
const multer = require('multer');
const path = require('path');

// Configure storage
const storage = multer.diskStorage({
  // Where to save
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
    // cb(null, 'uploads/avatars/');  // different folder per type
  },
  
  // What to name the file
  filename: (req, file, cb) => {
    // Create unique filename: user-123-1700000000.jpg
    const uniqueName = `${file.fieldname}-${Date.now()}${path.extname(file.originalname)}`;
    cb(null, uniqueName);
  },
});

// File filter — only allow certain types
const fileFilter = (req, file, cb) => {
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];
  
  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);   // Accept file
  } else {
    cb(new Error('Only image files are allowed!'), false);  // Reject file
  }
};

// Create upload middleware
const upload = multer({
  storage,
  fileFilter,
  limits: {
    fileSize: 5 * 1024 * 1024,  // 5MB max file size
  },
});
```

### Using in Routes

```js
// SINGLE file upload
app.post('/api/upload', upload.single('image'), (req, res) => {
  // 'image' is the field name from the form
  console.log(req.file);
  // {
  //   fieldname: 'image',
  //   originalname: 'photo.jpg',
  //   encoding: '7bit',
  //   mimetype: 'image/jpeg',
  //   destination: 'uploads/',
  //   filename: 'image-1700000000.jpg',
  //   path: 'uploads/image-1700000000.jpg',
  //   size: 245678
  // }
  
  res.json({
    message: 'File uploaded',
    file: req.file.filename,
    path: `/uploads/${req.file.filename}`,
  });
});

// MULTIPLE files (same field)
app.post('/api/gallery', upload.array('photos', 5), (req, res) => {
  // Max 5 files, field name 'photos'
  console.log(req.files);  // Array of file objects
  
  const filePaths = req.files.map(f => `/uploads/${f.filename}`);
  res.json({ message: 'Files uploaded', files: filePaths });
});

// MULTIPLE fields
app.post('/api/product', upload.fields([
  { name: 'thumbnail', maxCount: 1 },
  { name: 'images', maxCount: 4 },
]), (req, res) => {
  console.log(req.files.thumbnail);  // Array with 1 file
  console.log(req.files.images);     // Array with up to 4 files
  res.json({ message: 'Product images uploaded' });
});
```

### Complete Example: Profile Picture Upload

```js
const express = require('express');
const multer = require('multer');
const path = require('path');
const app = express();

app.use(express.json());
app.use('/uploads', express.static('uploads'));  // Serve uploaded files

const storage = multer.diskStorage({
  destination: 'uploads/avatars/',
  filename: (req, file, cb) => {
    cb(null, `avatar-${Date.now()}${path.extname(file.originalname)}`);
  },
});

const upload = multer({
  storage,
  limits: { fileSize: 2 * 1024 * 1024 },  // 2MB
  fileFilter: (req, file, cb) => {
    const allowed = /jpeg|jpg|png|gif/;
    const ext = allowed.test(path.extname(file.originalname).toLowerCase());
    const mime = allowed.test(file.mimetype);
    
    if (ext && mime) return cb(null, true);
    cb(new Error('Only images allowed'));
  },
});

// Upload route
app.post('/api/users/avatar', upload.single('avatar'), async (req, res) => {
  if (!req.file) {
    return res.status(400).json({ message: 'Please upload an image' });
  }
  
  // Save file path to user in DB
  // await User.findByIdAndUpdate(req.user.id, {
  //   profilePicture: `/uploads/avatars/${req.file.filename}`
  // });
  
  res.json({
    message: 'Avatar uploaded',
    imageUrl: `/uploads/avatars/${req.file.filename}`,
  });
});

// Handle multer errors
app.use((err, req, res, next) => {
  if (err instanceof multer.MulterError) {
    if (err.code === 'LIMIT_FILE_SIZE') {
      return res.status(400).json({ message: 'File too large. Max 2MB.' });
    }
    return res.status(400).json({ message: err.message });
  }
  
  if (err.message === 'Only images allowed') {
    return res.status(400).json({ message: err.message });
  }
  
  next(err);
});

app.listen(3000);
```

---

## 🎯 Interview Answers

### Q: How do you serve static files in Express?
> Use `express.static()` middleware. Pass the directory name — e.g., `app.use(express.static('public'))` makes all files in the `public` folder accessible. You can add a URL prefix like `app.use('/static', express.static('public'))`. Always use `path.join(__dirname, 'public')` for absolute paths to avoid issues.

### Q: How do you handle file uploads in Express?
> Use the `multer` middleware. Configure it with `diskStorage` to control the destination and filename, add `fileFilter` to restrict file types, and set `limits` for file size. Use `upload.single('fieldname')` for single files, `upload.array('fieldname', max)` for multiple files. The uploaded file info is available on `req.file` (single) or `req.files` (multiple).

---

[⬅️ Module 11: Auth](../11-auth/README.md) | [Next: Environment Config ➡️](../13-env-config/README.md)
