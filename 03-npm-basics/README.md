# Module 3: npm & package.json

## What is npm?

**npm** = Node Package Manager. It's two things:
1. **A CLI tool** — installed with Node.js, used to install/manage packages
2. **A registry** — online database of 2M+ packages (npmjs.com)

---

## Initializing a Project

```bash
# Interactive — asks questions
npm init

# Quick — uses defaults (recommended for learning)
npm init -y
```

This creates `package.json` — the **heart** of any Node.js project.

---

## `package.json` Deep Dive

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "A sample Node.js app",
  "main": "index.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.6.3"
  },
  "devDependencies": {
    "nodemon": "^3.0.1",
    "jest": "^29.7.0"
  },
  "keywords": ["node", "express", "api"],
  "author": "Aryan Gupta",
  "license": "MIT"
}
```

### Key fields:
| Field | What it does |
|-------|-------------|
| `name` | Project name (must be lowercase, no spaces) |
| `version` | Project version (semver) |
| `main` | Entry point file |
| `scripts` | Custom commands you can run with `npm run <name>` |
| `dependencies` | Packages needed in production |
| `devDependencies` | Packages needed only during development |

---

## Installing Packages

```bash
# Install a production dependency
npm install express
npm i express                    # shorthand

# Install a dev dependency
npm install nodemon --save-dev
npm i nodemon -D                 # shorthand

# Install globally (available anywhere in terminal)
npm install -g nodemon

# Install specific version
npm install express@4.17.1

# Install all dependencies from package.json
npm install
npm i                            # shorthand

# Uninstall a package
npm uninstall express
npm un express                   # shorthand
```

---

## `dependencies` vs `devDependencies` ⭐

| | `dependencies` | `devDependencies` |
|--|---------------|-------------------|
| **What** | Needed to RUN the app | Needed only during DEVELOPMENT |
| **Installed in prod?** | ✅ Yes | ❌ No (`npm i --production` skips them) |
| **Examples** | express, mongoose, jsonwebtoken | nodemon, jest, eslint |
| **Flag** | `npm i <pkg>` | `npm i <pkg> -D` |

```bash
# In production, only install dependencies (skip devDependencies)
npm install --production
# OR
NODE_ENV=production npm install
```

---

## Semantic Versioning (SemVer) ⭐

Version format: `MAJOR.MINOR.PATCH` → e.g., `4.18.2`

```
4  .  18  .  2
│     │     │
│     │     └── PATCH — bug fixes (safe to update)
│     └──────── MINOR — new features, backward compatible (safe to update)
└────────────── MAJOR — breaking changes (might break your code!)
```

### Version ranges in package.json:

```json
{
  "dependencies": {
    "express": "4.18.2",     // Exact version only
    "express": "^4.18.2",   // ^caret: 4.x.x (any minor/patch, same major)
    "express": "~4.18.2",   // ~tilde: 4.18.x (any patch, same minor)
    "express": ">=4.18.2",  // Greater than or equal
    "express": "*"           // Any version (dangerous!)
  }
}
```

### Most common:
| Symbol | Meaning | Example: `^4.18.2` |
|--------|---------|-------------------|
| `^` (caret) | Same major version | Allows `4.18.2` to `4.99.99` |
| `~` (tilde) | Same minor version | Allows `4.18.2` to `4.18.99` |
| none | Exact version | Only `4.18.2` |

> **Default behavior:** `npm install express` adds `"express": "^4.18.2"` (caret).

---

## `package.json` vs `package-lock.json` ⭐

| | `package.json` | `package-lock.json` |
|--|---------------|---------------------|
| **What** | Declares dependencies with version ranges | Locks exact versions of ALL dependencies |
| **Version** | `"^4.18.2"` (range) | `"4.18.2"` (exact) |
| **Includes sub-deps?** | ❌ No | ✅ Yes (entire dependency tree) |
| **Commit to git?** | ✅ Yes | ✅ Yes |
| **Edit manually?** | ✅ Yes | ❌ Never |

### Why `package-lock.json` matters:
```
Without lock file:
  Dev A installs → gets express 4.18.2
  Dev B installs → gets express 4.18.5 (newer patch released)
  → Different versions! Bugs may appear on one machine.

With lock file:
  Both get EXACTLY express 4.18.2
  → Consistent across all machines.
```

---

## npm Scripts

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest",
    "build": "tsc",
    "lint": "eslint .",
    "seed": "node seed.js"
  }
}
```

### Running scripts:
```bash
# Special scripts (don't need 'run')
npm start          # runs "start" script
npm test           # runs "test" script

# Custom scripts (need 'run')
npm run dev        # runs "dev" script
npm run build      # runs "build" script
npm run seed       # runs "seed" script
```

---

## npx — Execute Packages Without Installing

```bash
# Without npx — need to install globally first
npm install -g create-react-app
create-react-app my-app

# With npx — runs without installing
npx create-react-app my-app

# npx finds the package, downloads it temporarily, runs it, then cleans up
```

### Common npx usage:
```bash
npx create-react-app my-app
npx create-next-app my-app
npx nodemon server.js          # if nodemon isn't installed globally
npx eslint .
```

---

## `.gitignore` — What to Ignore

Always create a `.gitignore` file:

```
# Dependencies
node_modules/

# Environment variables
.env
.env.local

# Logs
*.log
npm-debug.log*

# OS files
.DS_Store
Thumbs.db

# Build output
dist/
build/
```

> **NEVER** commit `node_modules/` to git. It's huge and can be recreated with `npm install`.

---

## Common npm Commands Cheat Sheet

```bash
npm init -y                # Initialize project
npm install                # Install all dependencies
npm i express              # Install production dep
npm i nodemon -D           # Install dev dep
npm i -g nodemon           # Install globally
npm uninstall express      # Remove package
npm update                 # Update packages to latest (within range)
npm outdated               # Check for outdated packages
npm list                   # List installed packages
npm list --depth=0         # List only top-level packages
npm cache clean --force    # Clear npm cache
npm audit                  # Check for vulnerabilities
npm audit fix              # Auto-fix vulnerabilities
```

---

## 🎯 Interview Answers

### Q: What is the difference between `dependencies` and `devDependencies`?
> `dependencies` are packages required for the application to run in production (like express, mongoose). `devDependencies` are packages only needed during development (like nodemon, jest, eslint). When deploying, you can run `npm install --production` to skip dev dependencies and reduce the deployment size.

### Q: What is `package-lock.json` and why is it important?
> `package-lock.json` locks the exact version of every installed dependency (including nested sub-dependencies). While `package.json` uses version ranges like `^4.18.2`, the lock file records the exact version `4.18.2` that was installed. This ensures every developer and every deployment gets the exact same dependency tree, preventing "works on my machine" bugs.

### Q: What does `^` and `~` mean in version numbers?
> In semantic versioning (MAJOR.MINOR.PATCH), `^` (caret) allows updates that don't change the major version — so `^4.18.2` accepts any version from `4.18.2` to `4.x.x`. `~` (tilde) allows updates that don't change the minor version — so `~4.18.2` accepts `4.18.2` to `4.18.x` only. Caret is the default when you `npm install`.

### Q: What is `npx`?
> `npx` is a tool that comes with npm (v5.2+) that lets you execute packages without installing them globally. It downloads the package temporarily, runs the command, and cleans up. It's commonly used for one-time commands like `npx create-react-app my-app` or `npx nodemon server.js`.

---

[⬅️ Module 2: Core Modules](../02-core-modules/README.md) | [Next: Express.js Basics ➡️](../04-express-basics/README.md)
