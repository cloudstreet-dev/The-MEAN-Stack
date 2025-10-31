# Chapter 4: Express.js - Your Backend Framework

## What Is Express.js?

Express.js is a minimal and flexible Node.js web application framework. That's the official description. The unofficial description is: "Express is the thing that makes building web servers in Node.js actually enjoyable."

Remember in Chapter 3 when we built a web server using raw Node.js? It looked like this:

```javascript
import http from 'http';

const server = http.createServer((req, res) => {
  if (req.url === '/' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end('<h1>Hello</h1>');
  } else if (req.url === '/api/data' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ data: 'here' }));
  } else {
    res.writeHead(404);
    res.end('Not Found');
  }
});
```

That works for exactly one route. Add ten routes, and you're drowning in nested if-statements. Try to handle POST data, and you're writing parsers. Want to add authentication? Good luck.

Express turns that mess into this:

```javascript
import express from 'express';
const app = express();

app.get('/', (req, res) => {
  res.send('<h1>Hello</h1>');
});

app.get('/api/data', (req, res) => {
  res.json({ data: 'here' });
});

app.listen(3000);
```

Better, right? Express handles routing, request parsing, response formatting, and provides a clean API for common web server tasks. It's the foundation of most Node.js web applications, and for good reason.

## Setting Up Your First Express App

Create a new directory and initialize a project:

```bash
mkdir express-demo
cd express-demo
npm init -y
```

Install Express:

```bash
npm install express
```

Add `"type": "module"` to your `package.json` to use ES modules:

```json
{
  "name": "express-demo",
  "version": "1.0.0",
  "type": "module",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

Install nodemon for development (auto-restarts on file changes):

```bash
npm install --save-dev nodemon
```

Create `index.js`:

```javascript
import express from 'express';

const app = express();
const PORT = 3000;

app.get('/', (req, res) => {
  res.send('Hello, Express!');
});

app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});
```

Run it:

```bash
npm run dev
```

Visit `http://localhost:3000` in your browser. You should see "Hello, Express!"

Congratulations, you've built an Express server. It's not impressive yet, but neither was learning to walk.

## Routing Basics

Routes define how your application responds to client requests at specific endpoints (URLs).

### Basic Routes

```javascript
import express from 'express';
const app = express();

// GET request to /
app.get('/', (req, res) => {
  res.send('Home Page');
});

// GET request to /about
app.get('/about', (req, res) => {
  res.send('About Page');
});

// POST request to /api/users
app.post('/api/users', (req, res) => {
  res.json({ message: 'User created' });
});

// PUT request to /api/users/123
app.put('/api/users/:id', (req, res) => {
  res.json({ message: `User ${req.params.id} updated` });
});

// DELETE request to /api/users/123
app.delete('/api/users/:id', (req, res) => {
  res.json({ message: `User ${req.params.id} deleted` });
});

app.listen(3000);
```

Express supports all HTTP methods:
- `GET` - Retrieve data
- `POST` - Create new data
- `PUT` / `PATCH` - Update data
- `DELETE` - Delete data
- `OPTIONS`, `HEAD`, etc.

You can also use `app.all()` to handle all methods:

```javascript
app.all('/secret', (req, res) => {
  res.send('You found the secret page!');
});
```

### Route Parameters

Capture values from the URL:

```javascript
// /users/42 -> req.params.id = '42'
app.get('/users/:id', (req, res) => {
  const userId = req.params.id;
  res.send(`User ID: ${userId}`);
});

// Multiple parameters
app.get('/users/:userId/posts/:postId', (req, res) => {
  res.json({
    userId: req.params.userId,
    postId: req.params.postId
  });
});

// Optional parameters (with ?)
app.get('/users/:id/:action?', (req, res) => {
  const { id, action } = req.params;
  res.send(`User: ${id}, Action: ${action || 'none'}`);
});
```

**Important:** Route parameters are always strings. Convert them when needed:

```javascript
app.get('/users/:id', (req, res) => {
  const userId = parseInt(req.params.id, 10);
  if (isNaN(userId)) {
    return res.status(400).json({ error: 'Invalid user ID' });
  }
  res.json({ userId });
});
```

### Query Parameters

Access query strings (`?key=value&key2=value2`):

```javascript
// GET /search?q=express&limit=10
app.get('/search', (req, res) => {
  const query = req.query.q;
  const limit = parseInt(req.query.limit) || 20;

  res.json({
    query,
    limit,
    results: [] // Your search results here
  });
});
```

Query parameters are always optional. Route parameters are part of the route itself.

### Pattern Matching

Express uses path-to-regexp for pattern matching:

```javascript
// Matches /users/123 but not /users/john
app.get('/users/:id(\\d+)', (req, res) => {
  res.send(`Numeric user ID: ${req.params.id}`);
});

// Matches anything starting with /files/
app.get('/files/*', (req, res) => {
  res.send('File request');
});

// Regular expressions
app.get(/.*fly$/, (req, res) => {
  res.send('Butterfly? Dragonfly?');
});
```

## Request and Response Objects

### The Request Object (req)

The request object contains information about the HTTP request:

```javascript
app.get('/demo', (req, res) => {
  console.log('Method:', req.method);           // GET
  console.log('URL:', req.url);                 // /demo?key=value
  console.log('Path:', req.path);               // /demo
  console.log('Query:', req.query);             // { key: 'value' }
  console.log('Params:', req.params);           // Route parameters
  console.log('Headers:', req.headers);         // HTTP headers
  console.log('IP:', req.ip);                   // Client IP
  console.log('Protocol:', req.protocol);       // http or https
  console.log('Hostname:', req.hostname);       // localhost
  console.log('Body:', req.body);               // Parsed body (needs middleware)

  res.send('Check console');
});
```

Useful request methods:

```javascript
// Check header
const contentType = req.get('Content-Type');

// Check if request accepts certain types
if (req.accepts('json')) {
  res.json({ data: 'here' });
} else if (req.accepts('html')) {
  res.send('<h1>Data</h1>');
}

// Check for specific query parameter
if (req.query.debug) {
  console.log('Debug mode enabled');
}
```

### The Response Object (res)

The response object provides methods for sending responses:

```javascript
// Send plain text
res.send('Hello');

// Send JSON
res.json({ message: 'Hello', timestamp: Date.now() });

// Send status code
res.status(404).send('Not Found');
res.status(201).json({ message: 'Created' });

// Send file
res.sendFile('/path/to/file.html');

// Redirect
res.redirect('/other-page');
res.redirect(301, '/permanently-moved');

// Set headers
res.set('Content-Type', 'text/plain');
res.set({
  'Content-Type': 'text/html',
  'X-Custom-Header': 'value'
});

// Download file
res.download('/path/to/file.pdf', 'filename.pdf');

// Render template (with view engine)
res.render('index', { title: 'Home' });
```

### Chaining

Response methods can be chained:

```javascript
res
  .status(200)
  .set('Content-Type', 'application/json')
  .json({ message: 'Success' });
```

## Middleware: The Heart of Express

Middleware functions are functions that have access to the request and response objects and the `next` function. They can:

- Execute code
- Modify request/response objects
- End the request-response cycle
- Call the next middleware

Think of middleware as a pipeline. Each request flows through multiple middleware functions before reaching your route handler.

### Basic Middleware

```javascript
// Logger middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
  next(); // Pass control to next middleware
});

// Route handler
app.get('/', (req, res) => {
  res.send('Hello');
});
```

If you don't call `next()`, the request hangs—the client waits forever (or until timeout).

### Application-Level Middleware

Runs for every request (or specific paths):

```javascript
// Runs for ALL routes
app.use((req, res, next) => {
  req.requestTime = Date.now();
  next();
});

// Runs only for routes starting with /api
app.use('/api', (req, res, next) => {
  console.log('API request');
  next();
});
```

### Route-Level Middleware

Runs for specific routes:

```javascript
// Single middleware
function authenticate(req, res, next) {
  if (req.headers.authorization) {
    next();
  } else {
    res.status(401).json({ error: 'Unauthorized' });
  }
}

app.get('/protected', authenticate, (req, res) => {
  res.send('You are authenticated!');
});

// Multiple middleware
app.get('/admin',
  authenticate,
  checkAdmin,
  (req, res) => {
    res.send('Admin panel');
  }
);
```

### Built-in Middleware

Express includes several built-in middleware functions:

```javascript
// Parse JSON bodies
app.use(express.json());

// Parse URL-encoded bodies (form data)
app.use(express.urlencoded({ extended: true }));

// Serve static files from 'public' directory
app.use(express.static('public'));
```

**Important:** `express.json()` is essential for parsing JSON from POST/PUT requests:

```javascript
app.use(express.json());

app.post('/api/users', (req, res) => {
  const { name, email } = req.body; // Now available thanks to express.json()
  res.json({ name, email });
});
```

### Third-Party Middleware

Popular middleware packages:

**CORS (Cross-Origin Resource Sharing):**

```bash
npm install cors
```

```javascript
import cors from 'cors';

// Allow all origins (development only!)
app.use(cors());

// Specific origins (production)
app.use(cors({
  origin: 'https://yourdomain.com',
  credentials: true
}));
```

**Morgan (Logging):**

```bash
npm install morgan
```

```javascript
import morgan from 'morgan';

// Development logging
app.use(morgan('dev'));

// Production logging
app.use(morgan('combined'));
```

**Helmet (Security Headers):**

```bash
npm install helmet
```

```javascript
import helmet from 'helmet';

app.use(helmet());
```

**Cookie Parser:**

```bash
npm install cookie-parser
```

```javascript
import cookieParser from 'cookie-parser';

app.use(cookieParser());

app.get('/set-cookie', (req, res) => {
  res.cookie('username', 'john', { maxAge: 900000, httpOnly: true });
  res.send('Cookie set');
});

app.get('/read-cookie', (req, res) => {
  const username = req.cookies.username;
  res.send(`Username: ${username}`);
});
```

### Error-Handling Middleware

Error middleware has **four** parameters:

```javascript
// Regular middleware (3 parameters)
app.use((req, res, next) => {
  // ...
  next();
});

// Error middleware (4 parameters)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    error: 'Something went wrong!',
    message: err.message
  });
});
```

Error middleware must be defined **last**:

```javascript
// Routes
app.get('/error', (req, res) => {
  throw new Error('Oops!');
});

// 404 handler
app.use((req, res) => {
  res.status(404).json({ error: 'Not Found' });
});

// Error handler (must be last)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: err.message });
});
```

Async errors need special handling:

```javascript
// Without error handling - server crashes!
app.get('/bad', async (req, res) => {
  const data = await somethingThatMightFail();
  res.json(data);
});

// With error handling
app.get('/good', async (req, res, next) => {
  try {
    const data = await somethingThatMightFail();
    res.json(data);
  } catch (error) {
    next(error); // Pass to error middleware
  }
});
```

Or use a wrapper:

```javascript
function asyncHandler(fn) {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}

app.get('/users', asyncHandler(async (req, res) => {
  const users = await database.getUsers();
  res.json(users);
}));
```

Or use the `express-async-errors` package:

```bash
npm install express-async-errors
```

```javascript
import 'express-async-errors';

// Now async errors are caught automatically
app.get('/users', async (req, res) => {
  const users = await database.getUsers();
  res.json(users);
});
```

## Router: Organizing Routes

As your app grows, putting all routes in one file becomes messy. Use `express.Router()` to organize routes by resource.

Create `routes/users.js`:

```javascript
import express from 'express';
const router = express.Router();

// Middleware for this router only
router.use((req, res, next) => {
  console.log('User routes accessed');
  next();
});

// GET /users
router.get('/', (req, res) => {
  res.json([
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' }
  ]);
});

// GET /users/:id
router.get('/:id', (req, res) => {
  const userId = parseInt(req.params.id);
  res.json({ id: userId, name: 'Alice' });
});

// POST /users
router.post('/', (req, res) => {
  const { name, email } = req.body;
  res.status(201).json({ id: 3, name, email });
});

// PUT /users/:id
router.put('/:id', (req, res) => {
  const userId = parseInt(req.params.id);
  const { name, email } = req.body;
  res.json({ id: userId, name, email });
});

// DELETE /users/:id
router.delete('/:id', (req, res) => {
  const userId = parseInt(req.params.id);
  res.json({ message: `User ${userId} deleted` });
});

export default router;
```

In your main `index.js`:

```javascript
import express from 'express';
import userRoutes from './routes/users.js';

const app = express();

app.use(express.json());
app.use('/users', userRoutes);

app.listen(3000);
```

Now all user routes are organized in one file, and the main file stays clean.

Create more routers for different resources:

```
routes/
  users.js
  posts.js
  comments.js
  auth.js
```

```javascript
import userRoutes from './routes/users.js';
import postRoutes from './routes/posts.js';
import commentRoutes from './routes/comments.js';
import authRoutes from './routes/auth.js';

app.use('/users', userRoutes);
app.use('/posts', postRoutes);
app.use('/comments', commentRoutes);
app.use('/auth', authRoutes);
```

## Building a RESTful API

REST (Representational State Transfer) is an architectural style for designing APIs. Here's a typical RESTful API for managing tasks:

```javascript
import express from 'express';
import 'express-async-errors';

const app = express();
app.use(express.json());

// In-memory database (replace with real database later)
let tasks = [
  { id: 1, title: 'Learn Express', completed: false },
  { id: 2, title: 'Build API', completed: false }
];
let nextId = 3;

// GET /api/tasks - Get all tasks
app.get('/api/tasks', (req, res) => {
  res.json(tasks);
});

// GET /api/tasks/:id - Get single task
app.get('/api/tasks/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const task = tasks.find(t => t.id === id);

  if (!task) {
    return res.status(404).json({ error: 'Task not found' });
  }

  res.json(task);
});

// POST /api/tasks - Create task
app.post('/api/tasks', (req, res) => {
  const { title, completed = false } = req.body;

  if (!title) {
    return res.status(400).json({ error: 'Title is required' });
  }

  const task = {
    id: nextId++,
    title,
    completed,
    createdAt: new Date().toISOString()
  };

  tasks.push(task);
  res.status(201).json(task);
});

// PUT /api/tasks/:id - Update task
app.put('/api/tasks/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const taskIndex = tasks.findIndex(t => t.id === id);

  if (taskIndex === -1) {
    return res.status(404).json({ error: 'Task not found' });
  }

  const { title, completed } = req.body;

  if (title !== undefined) tasks[taskIndex].title = title;
  if (completed !== undefined) tasks[taskIndex].completed = completed;
  tasks[taskIndex].updatedAt = new Date().toISOString();

  res.json(tasks[taskIndex]);
});

// DELETE /api/tasks/:id - Delete task
app.delete('/api/tasks/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const taskIndex = tasks.findIndex(t => t.id === id);

  if (taskIndex === -1) {
    return res.status(404).json({ error: 'Task not found' });
  }

  const deleted = tasks.splice(taskIndex, 1)[0];
  res.json({ message: 'Task deleted', task: deleted });
});

// 404 handler
app.use((req, res) => {
  res.status(404).json({ error: 'Not Found' });
});

// Error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Internal Server Error' });
});

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`API running at http://localhost:${PORT}`);
});
```

Test with cURL or Postman:

```bash
# Get all tasks
curl http://localhost:3000/api/tasks

# Create task
curl -X POST http://localhost:3000/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Learn MongoDB"}'

# Update task
curl -X PUT http://localhost:3000/api/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{"completed":true}'

# Delete task
curl -X DELETE http://localhost:3000/api/tasks/1
```

### RESTful Conventions

| HTTP Method | Route | Action | Response Code |
|-------------|-------|--------|---------------|
| GET | /resources | List all | 200 |
| GET | /resources/:id | Get one | 200 or 404 |
| POST | /resources | Create | 201 |
| PUT | /resources/:id | Update (full) | 200 or 404 |
| PATCH | /resources/:id | Update (partial) | 200 or 404 |
| DELETE | /resources/:id | Delete | 200 or 204 |

**Common HTTP Status Codes:**

- `200` OK - Request succeeded
- `201` Created - Resource created successfully
- `204` No Content - Success but no content to return
- `400` Bad Request - Invalid request data
- `401` Unauthorized - Authentication required
- `403` Forbidden - Authenticated but not allowed
- `404` Not Found - Resource doesn't exist
- `409` Conflict - Resource already exists
- `422` Unprocessable Entity - Validation failed
- `500` Internal Server Error - Server error

## Validation

Never trust client input. Validate everything.

### Manual Validation

```javascript
app.post('/api/tasks', (req, res) => {
  const { title, completed } = req.body;

  if (!title || typeof title !== 'string') {
    return res.status(400).json({ error: 'Title must be a string' });
  }

  if (title.trim().length === 0) {
    return res.status(400).json({ error: 'Title cannot be empty' });
  }

  if (title.length > 200) {
    return res.status(400).json({ error: 'Title too long (max 200 characters)' });
  }

  if (completed !== undefined && typeof completed !== 'boolean') {
    return res.status(400).json({ error: 'Completed must be boolean' });
  }

  // Validation passed, create task
  const task = { id: nextId++, title: title.trim(), completed: completed || false };
  tasks.push(task);
  res.status(201).json(task);
});
```

This gets tedious quickly. Use a validation library.

### Validation with Joi

```bash
npm install joi
```

```javascript
import Joi from 'joi';

const taskSchema = Joi.object({
  title: Joi.string().trim().min(1).max(200).required(),
  completed: Joi.boolean().default(false)
});

app.post('/api/tasks', (req, res) => {
  const { error, value } = taskSchema.validate(req.body);

  if (error) {
    return res.status(400).json({ error: error.details[0].message });
  }

  const task = { id: nextId++, ...value };
  tasks.push(task);
  res.status(201).json(task);
});
```

Or create a validation middleware:

```javascript
function validate(schema) {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body);

    if (error) {
      return res.status(400).json({ error: error.details[0].message });
    }

    req.body = value; // Use validated/sanitized value
    next();
  };
}

app.post('/api/tasks', validate(taskSchema), (req, res) => {
  const task = { id: nextId++, ...req.body };
  tasks.push(task);
  res.status(201).json(task);
});
```

## Environment Configuration

Use environment variables for configuration:

```bash
npm install dotenv
```

Create `.env`:

```
PORT=3000
NODE_ENV=development
API_VERSION=v1
```

Load in your app:

```javascript
import 'dotenv/config';

const PORT = process.env.PORT || 3000;
const NODE_ENV = process.env.NODE_ENV || 'development';

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT} in ${NODE_ENV} mode`);
});
```

Create different environments:

```
.env              # Default
.env.development  # Development settings
.env.production   # Production settings
.env.test         # Test settings
```

## File Uploads

Handle file uploads with `multer`:

```bash
npm install multer
```

```javascript
import multer from 'multer';

// Configure storage
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
  },
  filename: (req, file, cb) => {
    const uniqueName = `${Date.now()}-${file.originalname}`;
    cb(null, uniqueName);
  }
});

// File filter
const fileFilter = (req, file, cb) => {
  if (file.mimetype.startsWith('image/')) {
    cb(null, true);
  } else {
    cb(new Error('Only images allowed'), false);
  }
};

const upload = multer({
  storage,
  fileFilter,
  limits: { fileSize: 5 * 1024 * 1024 } // 5MB
});

// Single file
app.post('/upload', upload.single('photo'), (req, res) => {
  res.json({
    message: 'File uploaded',
    file: req.file
  });
});

// Multiple files
app.post('/upload-multiple', upload.array('photos', 10), (req, res) => {
  res.json({
    message: `${req.files.length} files uploaded`,
    files: req.files
  });
});
```

## Testing Express Apps

Use Jest and Supertest:

```bash
npm install --save-dev jest supertest
```

Create `app.test.js`:

```javascript
import request from 'supertest';
import express from 'express';

const app = express();
app.use(express.json());

app.get('/api/tasks', (req, res) => {
  res.json([{ id: 1, title: 'Test' }]);
});

app.post('/api/tasks', (req, res) => {
  res.status(201).json({ id: 2, ...req.body });
});

describe('Tasks API', () => {
  test('GET /api/tasks returns tasks', async () => {
    const response = await request(app).get('/api/tasks');
    expect(response.status).toBe(200);
    expect(response.body).toHaveLength(1);
  });

  test('POST /api/tasks creates task', async () => {
    const response = await request(app)
      .post('/api/tasks')
      .send({ title: 'New task' });

    expect(response.status).toBe(201);
    expect(response.body.title).toBe('New task');
  });
});
```

Run tests:

```bash
npm test
```

## Best Practices

### 1. Structure Your Code

```
project/
├── src/
│   ├── routes/
│   │   ├── users.js
│   │   ├── posts.js
│   ├── controllers/
│   │   ├── userController.js
│   │   ├── postController.js
│   ├── middleware/
│   │   ├── auth.js
│   │   ├── validation.js
│   ├── models/
│   │   ├── User.js
│   │   ├── Post.js
│   ├── utils/
│   │   ├── database.js
│   │   ├── logger.js
│   ├── app.js
│   ├── server.js
├── tests/
├── package.json
└── .env
```

### 2. Separate App and Server

`app.js`:

```javascript
import express from 'express';
import userRoutes from './routes/users.js';

const app = express();

app.use(express.json());
app.use('/api/users', userRoutes);

export default app;
```

`server.js`:

```javascript
import app from './app.js';

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

This makes testing easier—you export `app` for tests, run `server.js` in production.

### 3. Use Controllers

Instead of putting logic in route files:

`routes/users.js`:

```javascript
import express from 'express';
import * as userController from '../controllers/userController.js';

const router = express.Router();

router.get('/', userController.getAllUsers);
router.get('/:id', userController.getUser);
router.post('/', userController.createUser);
router.put('/:id', userController.updateUser);
router.delete('/:id', userController.deleteUser);

export default router;
```

`controllers/userController.js`:

```javascript
export async function getAllUsers(req, res) {
  const users = await database.getUsers();
  res.json(users);
}

export async function getUser(req, res) {
  const user = await database.getUserById(req.params.id);
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  res.json(user);
}

// etc.
```

### 4. Handle Async Errors

Use `express-async-errors` or an async wrapper:

```bash
npm install express-async-errors
```

```javascript
import 'express-async-errors';

app.get('/users', async (req, res) => {
  const users = await database.getUsers(); // Errors caught automatically
  res.json(users);
});
```

### 5. Use Proper Status Codes

Be specific with HTTP status codes. Don't return 200 for everything.

### 6. Implement Rate Limiting

Prevent abuse with rate limiting:

```bash
npm install express-rate-limit
```

```javascript
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: 'Too many requests, please try again later'
});

app.use('/api', limiter);
```

### 7. Log Everything (But Wisely)

Use a logging library like Winston:

```javascript
import winston from 'winston';

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console());
}

app.use((req, res, next) => {
  logger.info(`${req.method} ${req.url}`);
  next();
});
```

## Common Pitfalls

### 1. Not Handling Async Errors

```javascript
// BAD - crashes server on error
app.get('/users', async (req, res) => {
  const users = await database.getUsers();
  res.json(users);
});

// GOOD
app.get('/users', async (req, res, next) => {
  try {
    const users = await database.getUsers();
    res.json(users);
  } catch (error) {
    next(error);
  }
});
```

### 2. Sending Response Multiple Times

```javascript
// BAD - can't send two responses
app.get('/bad', (req, res) => {
  res.send('First');
  res.send('Second'); // Error!
});

// GOOD - use return
app.get('/good', (req, res) => {
  if (error) {
    return res.status(400).json({ error: 'Bad request' });
  }
  res.json({ success: true });
});
```

### 3. Not Validating Input

Always validate user input. Never trust client data.

### 4. Exposing Sensitive Errors

```javascript
// BAD - exposes stack traces to client
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.stack });
});

// GOOD
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    error: process.env.NODE_ENV === 'production'
      ? 'Internal Server Error'
      : err.message
  });
});
```

## What We've Covered

You now know:

- What Express.js is and why it's useful
- Routing (basic, parameters, query strings, patterns)
- Request and response objects
- Middleware (application, route, error-handling)
- Routers for organizing code
- Building RESTful APIs
- Validation with Joi
- Environment configuration
- File uploads
- Testing Express apps
- Best practices and common pitfalls

Express.js is the backbone of your MEAN stack backend. Master it, and you'll be able to build robust APIs quickly.

## What's Next

Now that you can build APIs with Express, you need somewhere to store data persistently. In the next chapter, we'll dive into MongoDB—a NoSQL database that stores data as JSON-like documents. You'll learn how to model data, perform CRUD operations, work with relationships, and understand when NoSQL is (and isn't) the right choice.

---

*"Express is to Node.js what jQuery was to JavaScript: technically unnecessary, but it makes everything so much easier that you'd be crazy not to use it."* — Pragmatic Developer
