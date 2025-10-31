# Chapter 3: Node.js Fundamentals

## What Exactly Is Node.js?

Node.js is JavaScript running outside the browser. That's it. That's the big reveal.

For years, JavaScript was confined to browsers, manipulating DOM elements and responding to button clicks. It was the language you used to make things blink annoyingly and validate forms. Then Ryan Dahl had a thought: "What if JavaScript could do... more?"

He took V8 (Google Chrome's JavaScript engine), added some APIs for file system access, networking, and other system-level operations, wrapped it all up, and called it Node.js. Suddenly, JavaScript could read files, create servers, handle databases, and do all the backend work that previously required PHP, Ruby, Python, or Java.

The JavaScript running in Node.js is the same language. It's the environment that's different. In a browser, you have `window`, `document`, and the DOM. In Node.js, you have `global`, `process`, and access to the file system. Same language, different playground.

## Why Node.js Matters

### Event-Driven Architecture

Node.js uses an event-driven, non-blocking I/O model. This sounds fancy, but here's what it means in practice:

**Traditional (Blocking) Approach:**
```
1. Start reading file
2. Wait for file to finish reading (doing nothing)
3. Process file contents
4. Start database query
5. Wait for database (doing nothing)
6. Process database results
```

**Node.js (Non-Blocking) Approach:**
```
1. Start reading file
2. While file is reading, start database query
3. While both are happening, handle incoming HTTP requests
4. When file finishes, process it
5. When database responds, process that
6. All while handling new requests
```

Node.js doesn't sit idle waiting for slow operations. It delegates them to the system, continues working, and processes results when they're ready. This makes Node.js excellent at handling many concurrent operations—perfect for web servers.

### Single-Threaded (But Not Really)

Node.js is single-threaded for your JavaScript code. There's one event loop handling execution. But behind the scenes, Node.js uses a thread pool for I/O operations. You write single-threaded code, but Node.js handles the complexity of making it efficient.

This is great because:
- You avoid complex multithreading bugs
- You don't deal with locks, semaphores, and deadlocks
- Your code is easier to reason about

This is potentially limiting because:
- CPU-intensive tasks block the event loop
- You can't fully utilize multiple CPU cores without extra work
- Heavy computation can make your server unresponsive

For web applications (which spend most of their time waiting for I/O), this trade-off is usually worth it.

## Your First Real Node.js Script

Let's start simple. Create a file called `hello-node.js`:

```javascript
console.log('Hello from Node.js!');
console.log('Process ID:', process.pid);
console.log('Node version:', process.version);
console.log('Platform:', process.platform);
console.log('Current directory:', process.cwd());
```

Run it:

```bash
node hello-node.js
```

You should see output with your system information. The `process` object is globally available in Node.js—it provides information about the current Node.js process.

## Working with Modules

Node.js uses a module system to organize code. Prior to ES6 modules, Node.js used CommonJS. Modern Node.js supports both CommonJS and ES modules, but CommonJS is still common (pun intended).

### CommonJS Modules

Create a file called `math.js`:

```javascript
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

function multiply(a, b) {
  return a * b;
}

function divide(a, b) {
  if (b === 0) {
    throw new Error('Division by zero is forbidden (even in JavaScript)');
  }
  return a / b;
}

// Export functions
module.exports = {
  add,
  subtract,
  multiply,
  divide
};
```

Now create `calculator.js`:

```javascript
const math = require('./math');

console.log('10 + 5 =', math.add(10, 5));
console.log('10 - 5 =', math.subtract(10, 5));
console.log('10 * 5 =', math.multiply(10, 5));
console.log('10 / 5 =', math.divide(10, 5));

try {
  console.log('10 / 0 =', math.divide(10, 0));
} catch (error) {
  console.log('Error:', error.message);
}
```

Run it:

```bash
node calculator.js
```

The `require()` function loads modules. The `./` prefix means "in the current directory." Without it, Node.js looks for npm packages in `node_modules`.

### ES Modules

Modern JavaScript uses ES modules with `import` and `export`. To use them in Node.js, either:
1. Use the `.mjs` file extension, or
2. Add `"type": "module"` to your `package.json`

Let's use option 2. Create a `package.json`:

```json
{
  "type": "module"
}
```

Now create `math-es.js`:

```javascript
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

export function multiply(a, b) {
  return a * b;
}

export function divide(a, b) {
  if (b === 0) {
    throw new Error('Division by zero is still forbidden');
  }
  return a / b;
}

// Default export
export default {
  add,
  subtract,
  multiply,
  divide
};
```

And `calculator-es.js`:

```javascript
import math from './math-es.js';
// Or import individual functions:
// import { add, subtract } from './math-es.js';

console.log('10 + 5 =', math.add(10, 5));
console.log('10 - 5 =', math.subtract(10, 5));
console.log('10 * 5 =', math.multiply(10, 5));
console.log('10 / 5 =', math.divide(10, 5));
```

Run it:

```bash
node calculator-es.js
```

**Note:** With ES modules, you must include the `.js` extension in imports. CommonJS lets you omit it; ES modules don't.

For this book, I'll primarily use ES modules (because they're the future), but you'll see plenty of CommonJS in the wild, so be comfortable with both.

## Built-in Node.js Modules

Node.js comes with many built-in modules. Let's explore the most important ones.

### File System (fs)

Reading and writing files:

```javascript
import fs from 'fs';
import { promises as fsPromises } from 'fs';

// Synchronous (blocks execution)
try {
  const data = fs.readFileSync('file.txt', 'utf8');
  console.log(data);
} catch (error) {
  console.error('Error reading file:', error);
}

// Asynchronous with callbacks (old way)
fs.readFile('file.txt', 'utf8', (error, data) => {
  if (error) {
    console.error('Error reading file:', error);
    return;
  }
  console.log(data);
});

// Asynchronous with promises (modern way)
try {
  const data = await fsPromises.readFile('file.txt', 'utf8');
  console.log(data);
} catch (error) {
  console.error('Error reading file:', error);
}

// Writing files
await fsPromises.writeFile('output.txt', 'Hello, World!', 'utf8');

// Appending to files
await fsPromises.appendFile('output.txt', '\nMore text!', 'utf8');

// Check if file exists
try {
  await fsPromises.access('file.txt');
  console.log('File exists');
} catch {
  console.log('File does not exist');
}

// Get file stats
const stats = await fsPromises.stat('file.txt');
console.log('File size:', stats.size, 'bytes');
console.log('Is directory?', stats.isDirectory());
console.log('Modified:', stats.mtime);
```

**Pro tip:** Always use the async versions in production. Synchronous file operations block the entire event loop, making your server unresponsive.

### Path

Working with file paths:

```javascript
import path from 'path';

const filePath = '/users/john/documents/file.txt';

console.log('Directory:', path.dirname(filePath));    // /users/john/documents
console.log('Filename:', path.basename(filePath));    // file.txt
console.log('Extension:', path.extname(filePath));    // .txt
console.log('Filename without ext:', path.basename(filePath, path.extname(filePath))); // file

// Join paths (handles separators correctly)
const fullPath = path.join('users', 'john', 'documents', 'file.txt');
console.log(fullPath); // users/john/documents/file.txt (or users\john\documents\file.txt on Windows)

// Resolve to absolute path
const absolute = path.resolve('documents', 'file.txt');
console.log(absolute); // /current/working/directory/documents/file.txt

// Get the current file's directory (in ES modules)
import { fileURLToPath } from 'url';
import { dirname } from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

console.log('Current file:', __filename);
console.log('Current directory:', __dirname);
```

Always use `path.join()` or `path.resolve()` instead of concatenating paths with `/` or `\`. It handles Windows vs. Unix differences automatically.

### HTTP/HTTPS

Creating web servers:

```javascript
import http from 'http';

const server = http.createServer((req, res) => {
  console.log(`${req.method} ${req.url}`);

  if (req.url === '/') {
    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end('<h1>Hello, Node.js!</h1>');
  } else if (req.url === '/api/data') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ message: 'Hello from API', timestamp: Date.now() }));
  } else {
    res.writeHead(404, { 'Content-Type': 'text/plain' });
    res.end('404 Not Found');
  }
});

const PORT = 3000;
server.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});
```

This is a basic HTTP server. We'll use Express.js to make this much easier in the next chapter.

### Events

Node.js has a built-in event system:

```javascript
import { EventEmitter } from 'events';

class TaskManager extends EventEmitter {
  constructor() {
    super();
    this.tasks = [];
  }

  addTask(task) {
    this.tasks.push(task);
    this.emit('taskAdded', task);
  }

  completeTask(index) {
    if (index >= 0 && index < this.tasks.length) {
      const task = this.tasks[index];
      this.tasks.splice(index, 1);
      this.emit('taskCompleted', task);
    }
  }
}

const manager = new TaskManager();

// Listen for events
manager.on('taskAdded', (task) => {
  console.log('New task added:', task);
});

manager.on('taskCompleted', (task) => {
  console.log('Task completed:', task);
});

// Use it
manager.addTask('Write code');
manager.addTask('Test code');
manager.completeTask(0);
```

Many Node.js APIs use events. Understanding EventEmitter is essential.

### URL

Parsing URLs:

```javascript
import { URL } from 'url';

const myUrl = new URL('https://example.com:8080/path/to/page?name=John&age=30#section');

console.log('Protocol:', myUrl.protocol);      // https:
console.log('Host:', myUrl.host);              // example.com:8080
console.log('Hostname:', myUrl.hostname);      // example.com
console.log('Port:', myUrl.port);              // 8080
console.log('Pathname:', myUrl.pathname);      // /path/to/page
console.log('Search:', myUrl.search);          // ?name=John&age=30
console.log('Hash:', myUrl.hash);              // #section

// Query parameters
console.log('Name:', myUrl.searchParams.get('name'));  // John
console.log('Age:', myUrl.searchParams.get('age'));    // 30

// Modify URL
myUrl.searchParams.set('name', 'Jane');
myUrl.searchParams.append('city', 'New York');
console.log('Updated URL:', myUrl.href);
```

## Understanding Asynchronous Programming

This is the most important section in this chapter. Node.js is asynchronous. Understanding async programming is essential.

### Callbacks (The Old Way)

```javascript
import fs from 'fs';

fs.readFile('file.txt', 'utf8', (error, data) => {
  if (error) {
    console.error('Error:', error);
    return;
  }

  // Process data
  const processed = data.toUpperCase();

  fs.writeFile('output.txt', processed, 'utf8', (error) => {
    if (error) {
      console.error('Error writing:', error);
      return;
    }

    console.log('File processed successfully');
  });
});
```

This works, but nested callbacks become unwieldy quickly—welcome to "callback hell."

### Promises (Better)

```javascript
import { promises as fs } from 'fs';

fs.readFile('file.txt', 'utf8')
  .then(data => {
    const processed = data.toUpperCase();
    return fs.writeFile('output.txt', processed, 'utf8');
  })
  .then(() => {
    console.log('File processed successfully');
  })
  .catch(error => {
    console.error('Error:', error);
  });
```

Promises chain better than callbacks. You can catch errors in one place.

### Async/Await (Best)

```javascript
import { promises as fs } from 'fs';

async function processFile() {
  try {
    const data = await fs.readFile('file.txt', 'utf8');
    const processed = data.toUpperCase();
    await fs.writeFile('output.txt', processed, 'utf8');
    console.log('File processed successfully');
  } catch (error) {
    console.error('Error:', error);
  }
}

processFile();
```

Async/await makes asynchronous code look synchronous. It's syntactic sugar over promises, but it's delicious sugar.

**Key points:**
- `async` functions always return a Promise
- `await` pauses execution until the Promise resolves
- `await` only works inside `async` functions
- Use `try/catch` for error handling

### Running Multiple Async Operations

**Sequential (one after another):**

```javascript
async function sequential() {
  const result1 = await doSomething();
  const result2 = await doSomethingElse();
  const result3 = await doAnotherThing();
  return [result1, result2, result3];
}
```

**Parallel (all at once):**

```javascript
async function parallel() {
  const [result1, result2, result3] = await Promise.all([
    doSomething(),
    doSomethingElse(),
    doAnotherThing()
  ]);
  return [result1, result2, result3];
}
```

Use `Promise.all()` when operations are independent. It's much faster than running them sequentially.

**Race (first one wins):**

```javascript
async function race() {
  const result = await Promise.race([
    fetchFromServer1(),
    fetchFromServer2(),
    fetchFromServer3()
  ]);
  return result; // Whichever finishes first
}
```

## Error Handling in Node.js

Errors in Node.js come in different flavors.

### Synchronous Errors

Use `try/catch`:

```javascript
try {
  const data = JSON.parse('invalid json');
} catch (error) {
  console.error('Parse error:', error.message);
}
```

### Asynchronous Errors (Promises)

Use `.catch()` or `try/catch` with async/await:

```javascript
// With .catch()
doAsyncThing()
  .then(result => console.log(result))
  .catch(error => console.error('Error:', error));

// With async/await
async function example() {
  try {
    const result = await doAsyncThing();
    console.log(result);
  } catch (error) {
    console.error('Error:', error);
  }
}
```

### Callback Errors

First parameter is the error (Node.js convention):

```javascript
fs.readFile('file.txt', 'utf8', (error, data) => {
  if (error) {
    console.error('Error:', error);
    return;
  }
  console.log(data);
});
```

### Uncaught Exceptions

Handle them globally to prevent crashes:

```javascript
process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error);
  process.exit(1); // Exit after logging
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection at:', promise, 'reason:', reason);
  process.exit(1);
});
```

**Important:** These are last-resort handlers. You should catch errors properly, not rely on these.

## Working with npm Packages

Node.js alone is powerful, but npm packages make it phenomenal.

### Installing Packages

```bash
# Install locally (adds to package.json)
npm install package-name

# Install as a dev dependency
npm install --save-dev package-name

# Install globally
npm install -g package-name

# Install specific version
npm install package-name@1.2.3
```

### package.json

This file defines your project:

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "description": "My awesome project",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.20"
  }
}
```

Create one with:

```bash
npm init -y
```

### Useful npm Commands

```bash
npm install              # Install all dependencies
npm update              # Update packages
npm outdated            # Check for outdated packages
npm list                # List installed packages
npm uninstall package   # Remove a package
npm run script-name     # Run a script from package.json
```

### Popular npm Packages You'll Use

- **lodash** - Utility functions for JavaScript
- **axios** - HTTP client (easier than fetch)
- **dotenv** - Load environment variables from .env files
- **joi** or **yup** - Data validation
- **bcrypt** - Password hashing
- **jsonwebtoken** - JWT authentication
- **nodemon** - Auto-restart server on file changes
- **winston** - Logging
- **helmet** - Security headers for Express
- **cors** - Cross-Origin Resource Sharing

## Environment Variables

Never hardcode sensitive data (passwords, API keys, secrets) in your code. Use environment variables.

Install dotenv:

```bash
npm install dotenv
```

Create a `.env` file:

```
DATABASE_URL=mongodb://localhost:27017/mydb
API_KEY=super-secret-key
PORT=3000
NODE_ENV=development
```

**IMPORTANT:** Add `.env` to your `.gitignore`:

```
.env
node_modules/
```

Use in your code:

```javascript
import 'dotenv/config';

const dbUrl = process.env.DATABASE_URL;
const apiKey = process.env.API_KEY;
const port = process.env.PORT || 3000;

console.log('Connecting to:', dbUrl);
console.log('Port:', port);
// Never log API keys or passwords!
```

Create a `.env.example` file with dummy values for other developers:

```
DATABASE_URL=mongodb://localhost:27017/yourdb
API_KEY=your-api-key-here
PORT=3000
NODE_ENV=development
```

This goes in git. The real `.env` doesn't.

## Building a Practical Example: File-Based Task Manager

Let's build something useful: a simple task manager that stores tasks in a JSON file.

Create `task-manager.js`:

```javascript
import { promises as fs } from 'fs';
import path from 'path';

const DATA_FILE = path.join(process.cwd(), 'tasks.json');

class TaskManager {
  constructor() {
    this.tasks = [];
  }

  async load() {
    try {
      const data = await fs.readFile(DATA_FILE, 'utf8');
      this.tasks = JSON.parse(data);
      console.log(`Loaded ${this.tasks.length} tasks`);
    } catch (error) {
      if (error.code === 'ENOENT') {
        // File doesn't exist yet
        this.tasks = [];
        await this.save();
      } else {
        throw error;
      }
    }
  }

  async save() {
    await fs.writeFile(DATA_FILE, JSON.stringify(this.tasks, null, 2), 'utf8');
  }

  async addTask(title, description = '') {
    const task = {
      id: Date.now(),
      title,
      description,
      completed: false,
      createdAt: new Date().toISOString()
    };
    this.tasks.push(task);
    await this.save();
    return task;
  }

  async listTasks() {
    return this.tasks;
  }

  async completeTask(id) {
    const task = this.tasks.find(t => t.id === id);
    if (!task) {
      throw new Error(`Task ${id} not found`);
    }
    task.completed = true;
    task.completedAt = new Date().toISOString();
    await this.save();
    return task;
  }

  async deleteTask(id) {
    const index = this.tasks.findIndex(t => t.id === id);
    if (index === -1) {
      throw new Error(`Task ${id} not found`);
    }
    const deleted = this.tasks.splice(index, 1)[0];
    await this.save();
    return deleted;
  }
}

// CLI interface
async function main() {
  const manager = new TaskManager();
  await manager.load();

  const command = process.argv[2];
  const args = process.argv.slice(3);

  try {
    switch (command) {
      case 'add':
        if (args.length === 0) {
          console.error('Usage: node task-manager.js add "Task title" ["Description"]');
          process.exit(1);
        }
        const task = await manager.addTask(args[0], args[1]);
        console.log('Task added:', task);
        break;

      case 'list':
        const tasks = await manager.listTasks();
        if (tasks.length === 0) {
          console.log('No tasks yet!');
        } else {
          tasks.forEach(task => {
            const status = task.completed ? '✓' : ' ';
            console.log(`[${status}] ${task.id}: ${task.title}`);
            if (task.description) {
              console.log(`    ${task.description}`);
            }
          });
        }
        break;

      case 'complete':
        if (args.length === 0) {
          console.error('Usage: node task-manager.js complete <task-id>');
          process.exit(1);
        }
        const completed = await manager.completeTask(parseInt(args[0]));
        console.log('Task completed:', completed.title);
        break;

      case 'delete':
        if (args.length === 0) {
          console.error('Usage: node task-manager.js delete <task-id>');
          process.exit(1);
        }
        const deleted = await manager.deleteTask(parseInt(args[0]));
        console.log('Task deleted:', deleted.title);
        break;

      default:
        console.log('Usage:');
        console.log('  node task-manager.js add "Task title" ["Description"]');
        console.log('  node task-manager.js list');
        console.log('  node task-manager.js complete <task-id>');
        console.log('  node task-manager.js delete <task-id>');
    }
  } catch (error) {
    console.error('Error:', error.message);
    process.exit(1);
  }
}

main();
```

Try it out:

```bash
node task-manager.js add "Learn Node.js"
node task-manager.js add "Build something cool" "Like a web server"
node task-manager.js list
node task-manager.js complete 1234567890
node task-manager.js list
node task-manager.js delete 1234567890
```

This demonstrates:
- File system operations
- Async/await
- Command-line arguments
- Error handling
- JSON parsing/stringifying
- Class-based organization

## Debugging Node.js Applications

### Console Debugging (The Classic)

```javascript
console.log('Value:', value);
console.error('Error:', error);
console.table(arrayOfObjects); // Formats as a table
console.time('operation');
// ... do something ...
console.timeEnd('operation'); // Logs time elapsed
```

### Using the Debugger

Add `debugger` statement in your code:

```javascript
function problematicFunction(data) {
  debugger; // Execution pauses here
  const result = data.map(item => item * 2);
  return result;
}
```

Run with inspect:

```bash
node inspect script.js
```

Or use VS Code's built-in debugger:
1. Set breakpoints by clicking left of line numbers
2. Press F5 or click "Run and Debug"
3. Use the debug toolbar to step through code

### Logging Libraries

For production, use a logging library like Winston:

```bash
npm install winston
```

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
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

logger.info('Server started');
logger.error('Something went wrong', { error: error.message });
```

## Performance Considerations

### Don't Block the Event Loop

Bad:

```javascript
// This blocks for 5 seconds!
function sleep(ms) {
  const start = Date.now();
  while (Date.now() - start < ms) {}
}

app.get('/slow', (req, res) => {
  sleep(5000); // Blocks entire server
  res.send('Done');
});
```

Good:

```javascript
async function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

app.get('/slow', async (req, res) => {
  await sleep(5000); // Doesn't block
  res.send('Done');
});
```

### Use Streams for Large Data

Instead of reading entire files into memory:

```javascript
import fs from 'fs';

// Bad for large files
const data = await fs.promises.readFile('huge-file.txt', 'utf8');
response.send(data);

// Good for large files
const stream = fs.createReadStream('huge-file.txt');
stream.pipe(response);
```

### Monitor Memory Usage

```javascript
setInterval(() => {
  const usage = process.memoryUsage();
  console.log('Memory:', {
    rss: `${Math.round(usage.rss / 1024 / 1024)}MB`,
    heapTotal: `${Math.round(usage.heapTotal / 1024 / 1024)}MB`,
    heapUsed: `${Math.round(usage.heapUsed / 1024 / 1024)}MB`
  });
}, 5000);
```

## Testing Node.js Code

Use a testing framework like Jest:

```bash
npm install --save-dev jest
```

Create `math.test.js`:

```javascript
import { add, subtract, multiply, divide } from './math.js';

describe('Math functions', () => {
  test('add should add two numbers', () => {
    expect(add(2, 3)).toBe(5);
    expect(add(-1, 1)).toBe(0);
  });

  test('subtract should subtract two numbers', () => {
    expect(subtract(5, 3)).toBe(2);
    expect(subtract(0, 5)).toBe(-5);
  });

  test('multiply should multiply two numbers', () => {
    expect(multiply(3, 4)).toBe(12);
    expect(multiply(-2, 3)).toBe(-6);
  });

  test('divide should divide two numbers', () => {
    expect(divide(10, 2)).toBe(5);
    expect(divide(7, 2)).toBe(3.5);
  });

  test('divide should throw on division by zero', () => {
    expect(() => divide(10, 0)).toThrow('Division by zero');
  });
});
```

Add to `package.json`:

```json
{
  "scripts": {
    "test": "jest"
  }
}
```

Run tests:

```bash
npm test
```

## What We've Covered

You now understand:

- What Node.js is and how it works
- The module system (CommonJS and ES modules)
- Built-in Node.js modules (fs, path, http, events, url)
- Asynchronous programming (callbacks, promises, async/await)
- Error handling patterns
- Working with npm and packages
- Environment variables
- Building practical applications
- Debugging techniques
- Performance considerations
- Testing basics

These fundamentals will serve you throughout your Node.js journey.

## What's Next

In the next chapter, we'll dive into Express.js, the most popular Node.js web framework. While you could build web servers with raw Node.js (like we did earlier), Express.js makes it dramatically easier. It's like the difference between building a house with raw lumber versus using pre-fabricated components—both work, but one is much more efficient.

You'll learn routing, middleware, request handling, response formatting, and all the tools that make Express.js the backbone of countless web applications.

---

*"Understanding async/await is the difference between being a Node.js developer and being a confused person who occasionally gets Node.js to work."* — Senior Developer Who Learned the Hard Way
