# Chapter 6: Mongoose - The ODM You Didn't Know You Needed

## What Is Mongoose?

Mongoose is an Object Document Mapper (ODM) for MongoDB and Node.js. It provides a schema-based solution to model your application data, along with built-in validation, query building, business logic hooks, and more.

In Chapter 5, you learned to work with MongoDB using the raw MongoDB driver. It works, but it's like writing plain JavaScript—functional but lacking structure. Mongoose is like adding TypeScript: it adds schemas, type checking, and helpful abstractions that prevent mistakes and make your code cleaner.

Here's the same operation with and without Mongoose:

**Raw MongoDB Driver:**

```javascript
await db.collection('users').insertOne({
  username: 'alice',
  email: 'alice@example.com',
  age: '25', // Oops, should be a number
  createdAt: new Date()
});
```

No error, but your data is wrong.

**With Mongoose:**

```javascript
const user = new User({
  username: 'alice',
  email: 'alice@example.com',
  age: '25' // Mongoose converts to number automatically
});

await user.save();
```

Mongoose converts the string to a number. If it couldn't, it would throw an error. This prevents bad data from entering your database.

## Installing Mongoose

```bash
npm install mongoose
```

That's it. Mongoose includes the MongoDB driver, so you don't need both.

## Connecting to MongoDB

Create a file called `database.js`:

```javascript
import mongoose from 'mongoose';

const MONGODB_URL = process.env.MONGODB_URL || 'mongodb://localhost:27017/taskmanager';

async function connectDatabase() {
  try {
    await mongoose.connect(MONGODB_URL);
    console.log('Connected to MongoDB via Mongoose');
  } catch (error) {
    console.error('MongoDB connection error:', error);
    process.exit(1);
  }
}

// Handle connection events
mongoose.connection.on('connected', () => {
  console.log('Mongoose connected to MongoDB');
});

mongoose.connection.on('error', (err) => {
  console.error('Mongoose connection error:', err);
});

mongoose.connection.on('disconnected', () => {
  console.log('Mongoose disconnected');
});

// Graceful shutdown
process.on('SIGINT', async () => {
  await mongoose.connection.close();
  console.log('Mongoose connection closed');
  process.exit(0);
});

export default connectDatabase;
```

Use it in your app:

```javascript
import connectDatabase from './database.js';

await connectDatabase();

// Your app code here
```

## Schemas: Defining Your Data Structure

Schemas define the structure of documents within a collection.

Create `models/Task.js`:

```javascript
import mongoose from 'mongoose';

const taskSchema = new mongoose.Schema({
  title: {
    type: String,
    required: [true, 'Title is required'],
    trim: true,
    maxlength: [200, 'Title cannot exceed 200 characters']
  },

  description: {
    type: String,
    trim: true,
    default: ''
  },

  completed: {
    type: Boolean,
    default: false
  },

  priority: {
    type: Number,
    min: [0, 'Priority cannot be negative'],
    max: [10, 'Priority cannot exceed 10'],
    default: 0
  },

  dueDate: {
    type: Date,
    validate: {
      validator: function(value) {
        return !value || value > new Date();
      },
      message: 'Due date must be in the future'
    }
  },

  tags: {
    type: [String],
    default: []
  },

  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  }
}, {
  timestamps: true // Automatically adds createdAt and updatedAt
});

const Task = mongoose.model('Task', taskSchema);

export default Task;
```

Let's break down what's happening:

### Schema Types

```javascript
String      // Text
Number      // Numbers (integers and floats)
Date        // Dates
Boolean     // true/false
ObjectId    // MongoDB ObjectId (for references)
Array       // Arrays
Buffer      // Binary data
Mixed       // Any type (use sparingly)
Map         // Map data structure
Decimal128  // High-precision decimals
```

### Schema Options

**required:** Field must have a value

```javascript
title: {
  type: String,
  required: true
}

// With custom error message
title: {
  type: String,
  required: [true, 'Title is required']
}

// Dynamic requirement
title: {
  type: String,
  required: function() {
    return this.status === 'active';
  }
}
```

**default:** Default value if none provided

```javascript
completed: {
  type: Boolean,
  default: false
}

createdAt: {
  type: Date,
  default: Date.now // Function reference (no parentheses!)
}
```

**unique:** Ensure field value is unique

```javascript
email: {
  type: String,
  unique: true
}
```

**min/max:** Minimum and maximum values (numbers/dates)

```javascript
age: {
  type: Number,
  min: [0, 'Age cannot be negative'],
  max: [150, 'Age seems unrealistic']
}
```

**minlength/maxlength:** String length limits

```javascript
username: {
  type: String,
  minlength: [3, 'Username must be at least 3 characters'],
  maxlength: [30, 'Username cannot exceed 30 characters']
}
```

**trim:** Remove whitespace from strings

```javascript
title: {
  type: String,
  trim: true
}
```

**lowercase/uppercase:** Convert case

```javascript
email: {
  type: String,
  lowercase: true
}
```

**enum:** Limit to specific values

```javascript
status: {
  type: String,
  enum: ['pending', 'active', 'completed', 'archived'],
  default: 'pending'
}
```

**match:** Validate with regex

```javascript
email: {
  type: String,
  match: [/^\S+@\S+\.\S+$/, 'Invalid email format']
}
```

**validate:** Custom validation

```javascript
password: {
  type: String,
  validate: {
    validator: function(value) {
      return value.length >= 8;
    },
    message: 'Password must be at least 8 characters'
  }
}
```

### Schema-Level Options

```javascript
const schema = new mongoose.Schema({
  // fields
}, {
  timestamps: true,           // Adds createdAt and updatedAt
  collection: 'tasks',        // Specify collection name
  toJSON: { virtuals: true }, // Include virtuals in JSON
  toObject: { virtuals: true } // Include virtuals in objects
});
```

## Creating and Saving Documents

### Using `new` and `save()`

```javascript
import Task from './models/Task.js';

const task = new Task({
  title: 'Learn Mongoose',
  description: 'Study Mongoose ODM',
  priority: 8,
  userId: someUserId
});

await task.save();
console.log('Task saved:', task);
```

### Using `create()`

```javascript
const task = await Task.create({
  title: 'Learn Mongoose',
  description: 'Study Mongoose ODM',
  priority: 8,
  userId: someUserId
});
```

### Creating Multiple Documents

```javascript
const tasks = await Task.insertMany([
  { title: 'Task 1', userId: someUserId },
  { title: 'Task 2', userId: someUserId },
  { title: 'Task 3', userId: someUserId }
]);
```

### Validation Errors

```javascript
try {
  const task = new Task({
    // title is missing (required)
    completed: false
  });

  await task.save();
} catch (error) {
  if (error.name === 'ValidationError') {
    console.log('Validation failed:');
    for (const field in error.errors) {
      console.log(`${field}: ${error.errors[field].message}`);
    }
  }
}
```

## Querying Documents

Mongoose provides a rich query API built on top of MongoDB's.

### Finding Documents

```javascript
// Find all tasks
const tasks = await Task.find();

// Find with filter
const completedTasks = await Task.find({ completed: true });

// Find one document
const task = await Task.findOne({ title: 'Learn Mongoose' });

// Find by ID
const task = await Task.findById(taskId);
```

### Query Conditions

```javascript
// Greater than
const highPriority = await Task.find({ priority: { $gt: 7 } });

// Less than or equal
const lowPriority = await Task.find({ priority: { $lte: 3 } });

// In array
const tasks = await Task.find({ status: { $in: ['pending', 'active'] } });

// Regex
const tasks = await Task.find({ title: /mongoose/i });

// AND conditions
const tasks = await Task.find({
  completed: false,
  priority: { $gte: 5 }
});

// OR conditions
const tasks = await Task.find({
  $or: [
    { completed: true },
    { priority: { $gt: 8 } }
  ]
});
```

### Selecting Fields

```javascript
// Include only specific fields
const tasks = await Task.find().select('title completed');

// Exclude specific fields
const tasks = await Task.find().select('-description -tags');
```

### Sorting

```javascript
// Sort ascending
const tasks = await Task.find().sort('title');

// Sort descending
const tasks = await Task.find().sort('-createdAt');

// Multiple fields
const tasks = await Task.find().sort({ completed: 1, priority: -1 });
```

### Limiting and Skipping

```javascript
// Get first 10 tasks
const tasks = await Task.find().limit(10);

// Skip first 10, get next 10 (pagination)
const tasks = await Task.find().skip(10).limit(10);
```

### Counting

```javascript
// Count documents
const count = await Task.countDocuments({ completed: false });
```

### Chaining Queries

```javascript
const tasks = await Task
  .find({ completed: false })
  .where('priority').gte(5)
  .select('title priority')
  .sort('-priority')
  .limit(10)
  .exec();
```

## Updating Documents

### findByIdAndUpdate

```javascript
const task = await Task.findByIdAndUpdate(
  taskId,
  { completed: true },
  {
    new: true,           // Return updated document
    runValidators: true  // Run schema validators
  }
);
```

### findOneAndUpdate

```javascript
const task = await Task.findOneAndUpdate(
  { title: 'Learn Mongoose' },
  { completed: true },
  { new: true, runValidators: true }
);
```

### updateOne / updateMany

```javascript
// Update one document
const result = await Task.updateOne(
  { _id: taskId },
  { completed: true }
);

console.log('Modified:', result.modifiedCount);

// Update many documents
const result = await Task.updateMany(
  { completed: false },
  { archived: true }
);

console.log('Modified:', result.modifiedCount);
```

### Update with Instance Method

```javascript
const task = await Task.findById(taskId);
task.completed = true;
task.priority = 10;
await task.save();
```

**Important:** Use instance method (`save()`) when you want middleware to run. Use `findByIdAndUpdate()` for direct updates that skip middleware.

## Deleting Documents

### findByIdAndDelete

```javascript
const task = await Task.findByIdAndDelete(taskId);
console.log('Deleted task:', task);
```

### findOneAndDelete

```javascript
const task = await Task.findOneAndDelete({ title: 'Learn Mongoose' });
```

### deleteOne / deleteMany

```javascript
// Delete one
const result = await Task.deleteOne({ _id: taskId });
console.log('Deleted:', result.deletedCount);

// Delete many
const result = await Task.deleteMany({ completed: true });
console.log('Deleted:', result.deletedCount);
```

## Virtuals

Virtuals are document properties that aren't stored in MongoDB but are computed on the fly.

```javascript
const userSchema = new mongoose.Schema({
  firstName: String,
  lastName: String,
  email: String
});

// Virtual getter
userSchema.virtual('fullName').get(function() {
  return `${this.firstName} ${this.lastName}`;
});

// Virtual setter
userSchema.virtual('fullName').set(function(name) {
  const parts = name.split(' ');
  this.firstName = parts[0];
  this.lastName = parts[1];
});

const User = mongoose.model('User', userSchema);

const user = new User({
  firstName: 'Alice',
  lastName: 'Smith'
});

console.log(user.fullName); // "Alice Smith"

user.fullName = 'Bob Johnson';
console.log(user.firstName); // "Bob"
console.log(user.lastName);  // "Johnson"
```

**Note:** Virtuals aren't included in `toJSON()` by default. Enable them:

```javascript
const userSchema = new mongoose.Schema({
  // fields
}, {
  toJSON: { virtuals: true },
  toObject: { virtuals: true }
});
```

## Instance Methods

Add custom methods to document instances:

```javascript
const taskSchema = new mongoose.Schema({
  title: String,
  completed: Boolean,
  priority: Number
});

taskSchema.methods.markComplete = function() {
  this.completed = true;
  this.completedAt = new Date();
  return this.save();
};

taskSchema.methods.isHighPriority = function() {
  return this.priority >= 8;
};

const Task = mongoose.model('Task', taskSchema);

// Usage
const task = await Task.findById(taskId);
await task.markComplete();

if (task.isHighPriority()) {
  console.log('This is a high priority task!');
}
```

## Static Methods

Add custom methods to the model itself:

```javascript
taskSchema.statics.findHighPriority = function() {
  return this.find({ priority: { $gte: 8 } });
};

taskSchema.statics.findByUserId = function(userId) {
  return this.find({ userId });
};

// Usage
const highPriorityTasks = await Task.findHighPriority();
const userTasks = await Task.findByUserId(someUserId);
```

## Query Helpers

Add custom query methods:

```javascript
taskSchema.query.byUser = function(userId) {
  return this.where({ userId });
};

taskSchema.query.incomplete = function() {
  return this.where({ completed: false });
};

taskSchema.query.highPriority = function() {
  return this.where({ priority: { $gte: 8 } });
};

// Usage - chain query helpers
const tasks = await Task
  .find()
  .byUser(someUserId)
  .incomplete()
  .highPriority()
  .exec();
```

## Middleware (Hooks)

Middleware functions execute at specific stages of document lifecycle.

### Pre Hooks

Run before an operation:

```javascript
// Before saving
taskSchema.pre('save', function(next) {
  console.log('About to save task:', this.title);

  // Modify document
  this.updatedAt = new Date();

  next();
});

// Before validation
taskSchema.pre('validate', function(next) {
  // Sanitize data
  if (this.title) {
    this.title = this.title.trim();
  }
  next();
});

// Before removing
taskSchema.pre('remove', function(next) {
  console.log('About to remove task:', this.title);
  // Cleanup related data
  next();
});
```

### Post Hooks

Run after an operation:

```javascript
taskSchema.post('save', function(doc, next) {
  console.log('Task saved:', doc.title);
  next();
});

taskSchema.post('remove', function(doc, next) {
  console.log('Task removed:', doc.title);
  next();
});
```

### Practical Example: Password Hashing

```javascript
import bcrypt from 'bcrypt';

const userSchema = new mongoose.Schema({
  username: String,
  email: String,
  password: String
});

// Hash password before saving
userSchema.pre('save', async function(next) {
  // Only hash if password is modified
  if (!this.isModified('password')) {
    return next();
  }

  try {
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
    next();
  } catch (error) {
    next(error);
  }
});

// Method to compare password
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};

const User = mongoose.model('User', userSchema);

// Usage
const user = new User({
  username: 'alice',
  email: 'alice@example.com',
  password: 'mypassword123' // Plain text
});

await user.save(); // Password is hashed automatically

// Later, during login
const isMatch = await user.comparePassword('mypassword123');
console.log('Password correct:', isMatch);
```

## Population: Referencing Other Documents

Population automatically replaces references with actual documents.

### Basic Population

```javascript
// Task references User
const taskSchema = new mongoose.Schema({
  title: String,
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }
});

const Task = mongoose.model('Task', taskSchema);

// Find task and populate user
const task = await Task.findById(taskId).populate('userId');

console.log(task.userId.username); // Access user's username
```

### Selecting Fields

```javascript
// Only populate specific fields
const task = await Task
  .findById(taskId)
  .populate('userId', 'username email');
```

### Multiple Populations

```javascript
const task = await Task
  .findById(taskId)
  .populate('userId')
  .populate('assignedTo')
  .populate('comments');
```

### Nested Population

```javascript
const task = await Task
  .findById(taskId)
  .populate({
    path: 'userId',
    populate: {
      path: 'department'
    }
  });

console.log(task.userId.department.name);
```

### Conditional Population

```javascript
const task = await Task
  .findById(taskId)
  .populate({
    path: 'userId',
    match: { active: true },
    select: 'username email'
  });
```

## Subdocuments

Embed documents within documents:

```javascript
const commentSchema = new mongoose.Schema({
  text: String,
  author: String,
  createdAt: { type: Date, default: Date.now }
});

const postSchema = new mongoose.Schema({
  title: String,
  content: String,
  comments: [commentSchema] // Array of subdocuments
});

const Post = mongoose.model('Post', postSchema);

// Create post with comments
const post = new Post({
  title: 'My Post',
  content: 'Content here',
  comments: [
    { text: 'Great post!', author: 'Alice' },
    { text: 'Thanks!', author: 'Bob' }
  ]
});

await post.save();

// Add comment
post.comments.push({ text: 'Another comment', author: 'Charlie' });
await post.save();

// Find comment
const comment = post.comments.id(commentId);

// Remove comment
post.comments.pull({ _id: commentId });
await post.save();
```

## Validation

### Built-in Validators

```javascript
const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
    minlength: 3,
    maxlength: 30,
    unique: true,
    trim: true
  },

  age: {
    type: Number,
    min: 0,
    max: 150
  },

  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    match: /^\S+@\S+\.\S+$/
  },

  role: {
    type: String,
    enum: ['user', 'admin', 'moderator'],
    default: 'user'
  }
});
```

### Custom Validators

```javascript
const userSchema = new mongoose.Schema({
  password: {
    type: String,
    required: true,
    validate: {
      validator: function(value) {
        // Password must contain at least one uppercase, one lowercase, and one number
        return /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$/.test(value);
      },
      message: 'Password must be at least 8 characters and contain uppercase, lowercase, and number'
    }
  },

  confirmPassword: {
    type: String,
    required: true,
    validate: {
      validator: function(value) {
        return value === this.password;
      },
      message: 'Passwords do not match'
    }
  }
});
```

### Async Validators

```javascript
userSchema.path('username').validate(async function(value) {
  const count = await mongoose.model('User').countDocuments({ username: value });
  return count === 0;
}, 'Username already exists');
```

### Handling Validation Errors

```javascript
try {
  const user = new User({
    username: 'al',  // Too short
    age: -5,         // Negative
    email: 'invalid' // Invalid format
  });

  await user.save();
} catch (error) {
  if (error.name === 'ValidationError') {
    const errors = {};

    for (const field in error.errors) {
      errors[field] = error.errors[field].message;
    }

    console.log('Validation errors:', errors);
  }
}
```

## Transactions

Mongoose supports multi-document transactions:

```javascript
const session = await mongoose.startSession();

try {
  await session.withTransaction(async () => {
    // Deduct from one account
    await Account.updateOne(
      { _id: account1Id },
      { $inc: { balance: -100 } }
    ).session(session);

    // Add to another account
    await Account.updateOne(
      { _id: account2Id },
      { $inc: { balance: 100 } }
    ).session(session);

    // Both succeed or both fail
  });

  console.log('Transaction successful');
} catch (error) {
  console.error('Transaction failed:', error);
} finally {
  session.endSession();
}
```

## Practical Example: Complete User and Task Models

Create `models/User.js`:

```javascript
import mongoose from 'mongoose';
import bcrypt from 'bcrypt';

const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: [true, 'Username is required'],
    unique: true,
    trim: true,
    minlength: [3, 'Username must be at least 3 characters'],
    maxlength: [30, 'Username cannot exceed 30 characters']
  },

  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    trim: true,
    match: [/^\S+@\S+\.\S+$/, 'Invalid email format']
  },

  password: {
    type: String,
    required: [true, 'Password is required'],
    minlength: [8, 'Password must be at least 8 characters']
  },

  profile: {
    firstName: String,
    lastName: String,
    avatar: String,
    bio: { type: String, maxlength: 500 }
  },

  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user'
  },

  isActive: {
    type: Boolean,
    default: true
  }
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true }
});

// Indexes
userSchema.index({ email: 1 });
userSchema.index({ username: 1 });

// Virtual: fullName
userSchema.virtual('fullName').get(function() {
  if (this.profile.firstName && this.profile.lastName) {
    return `${this.profile.firstName} ${this.profile.lastName}`;
  }
  return this.username;
});

// Virtual: tasks (populate from Task model)
userSchema.virtual('tasks', {
  ref: 'Task',
  localField: '_id',
  foreignField: 'userId'
});

// Hash password before saving
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) {
    return next();
  }

  try {
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
    next();
  } catch (error) {
    next(error);
  }
});

// Remove password from JSON
userSchema.methods.toJSON = function() {
  const obj = this.toObject();
  delete obj.password;
  return obj;
};

// Compare password
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};

// Static: find by email
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email: email.toLowerCase() });
};

const User = mongoose.model('User', userSchema);

export default User;
```

Create `models/Task.js`:

```javascript
import mongoose from 'mongoose';

const taskSchema = new mongoose.Schema({
  title: {
    type: String,
    required: [true, 'Title is required'],
    trim: true,
    maxlength: [200, 'Title cannot exceed 200 characters']
  },

  description: {
    type: String,
    trim: true,
    default: ''
  },

  completed: {
    type: Boolean,
    default: false
  },

  priority: {
    type: Number,
    min: [0, 'Priority cannot be negative'],
    max: [10, 'Priority cannot exceed 10'],
    default: 0
  },

  dueDate: {
    type: Date
  },

  tags: {
    type: [String],
    default: []
  },

  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  }
}, {
  timestamps: true
});

// Indexes
taskSchema.index({ userId: 1, completed: 1 });
taskSchema.index({ dueDate: 1 });
taskSchema.index({ title: 'text', description: 'text' });

// Virtual: isOverdue
taskSchema.virtual('isOverdue').get(function() {
  return this.dueDate && !this.completed && this.dueDate < new Date();
});

// Instance method: mark complete
taskSchema.methods.markComplete = function() {
  this.completed = true;
  return this.save();
};

// Instance method: is high priority
taskSchema.methods.isHighPriority = function() {
  return this.priority >= 8;
};

// Static method: find by user
taskSchema.statics.findByUser = function(userId, options = {}) {
  let query = this.find({ userId });

  if (options.completed !== undefined) {
    query = query.where('completed', options.completed);
  }

  if (options.priority) {
    query = query.where('priority').gte(options.priority);
  }

  return query.sort('-createdAt');
};

// Static method: find overdue
taskSchema.statics.findOverdue = function(userId) {
  return this.find({
    userId,
    completed: false,
    dueDate: { $lt: new Date() }
  }).sort('dueDate');
};

// Query helper: incomplete
taskSchema.query.incomplete = function() {
  return this.where({ completed: false });
};

// Query helper: high priority
taskSchema.query.highPriority = function() {
  return this.where({ priority: { $gte: 8 } });
};

const Task = mongoose.model('Task', taskSchema);

export default Task;
```

Use them:

```javascript
import connectDatabase from './database.js';
import User from './models/User.js';
import Task from './models/Task.js';

await connectDatabase();

// Create user
const user = await User.create({
  username: 'alice',
  email: 'alice@example.com',
  password: 'SecurePass123',
  profile: {
    firstName: 'Alice',
    lastName: 'Smith'
  }
});

console.log('Created user:', user.fullName);

// Create tasks
await Task.create([
  {
    title: 'Complete project',
    description: 'Finish the MEAN stack app',
    priority: 9,
    dueDate: new Date('2024-12-31'),
    userId: user._id
  },
  {
    title: 'Write tests',
    priority: 7,
    userId: user._id
  }
]);

// Find user's high priority tasks
const highPriorityTasks = await Task
  .find()
  .byUser(user._id)
  .incomplete()
  .highPriority()
  .exec();

console.log('High priority tasks:', highPriorityTasks);

// Find overdue tasks
const overdueTasks = await Task.findOverdue(user._id);
console.log('Overdue tasks:', overdueTasks);

// Mark task complete
const task = await Task.findOne({ title: 'Write tests' });
await task.markComplete();

// Find user with tasks populated
const userWithTasks = await User
  .findById(user._id)
  .populate('tasks');

console.log('User tasks:', userWithTasks.tasks);
```

## Best Practices

### 1. Use Lean Queries for Performance

```javascript
// Regular query (returns Mongoose document)
const tasks = await Task.find({ userId });

// Lean query (returns plain JavaScript object)
const tasks = await Task.find({ userId }).lean();
```

Lean queries are faster and use less memory. Use them when you don't need Mongoose document features (methods, virtuals, etc.).

### 2. Select Only Needed Fields

```javascript
// Bad - fetches entire document
const users = await User.find();

// Good - fetches only needed fields
const users = await User.find().select('username email');
```

### 3. Use Indexes

Create indexes for fields you query frequently:

```javascript
userSchema.index({ email: 1 });
userSchema.index({ username: 1, createdAt: -1 });
```

### 4. Validate on Both Client and Server

Never trust client-side validation alone. Always validate on the server with Mongoose schemas.

### 5. Handle Errors Properly

```javascript
try {
  await user.save();
} catch (error) {
  if (error.name === 'ValidationError') {
    // Handle validation errors
  } else if (error.code === 11000) {
    // Handle duplicate key errors
  } else {
    // Handle other errors
  }
}
```

### 6. Use Transactions for Related Updates

When updating multiple documents that must succeed together, use transactions.

### 7. Don't Expose Sensitive Data

Remove sensitive fields from JSON responses:

```javascript
userSchema.methods.toJSON = function() {
  const obj = this.toObject();
  delete obj.password;
  delete obj.__v;
  return obj;
};
```

## Common Mistakes

### 1. Forgetting `new` with ObjectId

```javascript
// Bad
const tasks = await Task.find({ userId: userId });

// Good
import mongoose from 'mongoose';
const tasks = await Task.find({ userId: new mongoose.Types.ObjectId(userId) });

// Best - Mongoose converts automatically in most cases
const tasks = await Task.find({ userId });
```

### 2. Not Using `await`

```javascript
// Bad - returns a promise, not data
const task = Task.findById(taskId);

// Good
const task = await Task.findById(taskId);
```

### 3. Modifying `_id`

Never modify the `_id` field. It's immutable.

### 4. Over-Populating

Don't populate unnecessary data. It's slower than running separate queries.

```javascript
// Bad - populates everything
const tasks = await Task.find().populate('userId');

// Good - populate only what's needed
const tasks = await Task.find().populate('userId', 'username email');
```

### 5. Not Closing Connections in Tests

Always close database connections after tests:

```javascript
afterAll(async () => {
  await mongoose.connection.close();
});
```

## Testing with Mongoose

Use an in-memory MongoDB for tests:

```bash
npm install --save-dev mongodb-memory-server
```

Create `testDatabase.js`:

```javascript
import mongoose from 'mongoose';
import { MongoMemoryServer } from 'mongodb-memory-server';

let mongoServer;

export async function connectTestDatabase() {
  mongoServer = await MongoMemoryServer.create();
  const uri = mongoServer.getUri();
  await mongoose.connect(uri);
}

export async function closeTestDatabase() {
  await mongoose.connection.dropDatabase();
  await mongoose.connection.close();
  await mongoServer.stop();
}

export async function clearTestDatabase() {
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
}
```

Use in tests:

```javascript
import { connectTestDatabase, closeTestDatabase, clearTestDatabase } from './testDatabase.js';
import User from './models/User.js';

beforeAll(async () => {
  await connectTestDatabase();
});

afterAll(async () => {
  await closeTestDatabase();
});

afterEach(async () => {
  await clearTestDatabase();
});

describe('User Model', () => {
  test('should create user with hashed password', async () => {
    const user = await User.create({
      username: 'testuser',
      email: 'test@example.com',
      password: 'password123'
    });

    expect(user.password).not.toBe('password123');
    expect(user.password).toHaveLength(60); // bcrypt hash length
  });

  test('should compare password correctly', async () => {
    const user = await User.create({
      username: 'testuser',
      email: 'test@example.com',
      password: 'password123'
    });

    const isMatch = await user.comparePassword('password123');
    expect(isMatch).toBe(true);

    const isNotMatch = await user.comparePassword('wrongpassword');
    expect(isNotMatch).toBe(false);
  });
});
```

## What We've Covered

You now know:

- What Mongoose is and why it's useful
- Connecting to MongoDB with Mongoose
- Defining schemas with validation
- CRUD operations
- Virtuals, methods, and statics
- Middleware (hooks)
- Population (references)
- Subdocuments
- Transactions
- Best practices and common mistakes
- Testing with Mongoose

Mongoose adds structure to MongoDB without losing flexibility. It's the perfect balance for most applications.

## What's Next

You've now mastered the backend of the MEAN stack: Node.js, Express, MongoDB, and Mongoose. In the next chapter, we'll shift to the frontend and dive into Angular—a powerful framework for building modern, interactive web applications. You'll learn components, templates, services, routing, and how to build a frontend that communicates with your Express API.

---

*"Mongoose is like a safety harness for MongoDB. You could climb without it, but why would you?"* — Developer Who Made Mistakes So You Don't Have To
