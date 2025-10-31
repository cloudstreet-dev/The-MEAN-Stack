# Chapter 5: MongoDB - Embracing NoSQL

## What Is MongoDB?

MongoDB is a NoSQL document database. Instead of storing data in tables with rows and columns (like MySQL or PostgreSQL), MongoDB stores data as documents that look like JSON objects. It's schema-flexible, meaning you don't need to define your data structure upfront, and different documents in the same collection can have different fields.

Here's what a user might look like in a SQL database:

```
users table:
+----+-----------+----------------------+----------+
| id | username  | email                | age      |
+----+-----------+----------------------+----------+
| 1  | alice     | alice@example.com    | 25       |
| 2  | bob       | bob@example.com      | 30       |
+----+-----------+----------------------+----------+
```

Here's the same data in MongoDB:

```javascript
// users collection
{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  username: "alice",
  email: "alice@example.com",
  age: 25
}

{
  _id: ObjectId("507f1f77bcf86cd799439012"),
  username: "bob",
  email: "bob@example.com",
  age: 30,
  location: "New York"  // Different structure - totally fine!
}
```

Notice that Bob has a `location` field but Alice doesn't? That's okay in MongoDB. You can add fields to documents without migrating an entire table. This flexibility is both powerful and dangerous—with great power comes great responsibility to not make a mess of your data.

## SQL vs. NoSQL: A Quick Comparison

### SQL (Relational) Databases

**Pros:**
- Structured data with enforced schemas
- ACID transactions (Atomicity, Consistency, Isolation, Durability)
- Powerful query language (SQL)
- Great for complex relationships
- Mature tooling and ecosystem

**Cons:**
- Schema changes can be painful
- Scaling horizontally is complex
- Can be overkill for simple data
- Joins can be slow with large datasets

**Best for:** Banking systems, inventory management, anything requiring complex transactions and rigid data consistency.

### NoSQL (Document) Databases

**Pros:**
- Flexible schemas
- Easy to scale horizontally
- Fast for reads/writes
- Data structure matches application objects
- Great for rapid development

**Cons:**
- Less structure can lead to inconsistent data
- Limited transaction support (though MongoDB has improved this)
- No joins (you embed or reference)
- Easier to make data modeling mistakes

**Best for:** Content management, real-time analytics, catalogs, user profiles, anything with flexible or evolving data structures.

Neither is "better"—they solve different problems. For the MEAN stack, MongoDB's JSON-like structure makes it a natural fit with JavaScript.

## MongoDB Terminology

Coming from SQL? Here's the translation:

| SQL | MongoDB |
|-----|---------|
| Database | Database |
| Table | Collection |
| Row | Document |
| Column | Field |
| Index | Index |
| Join | Embedded document or reference |
| Primary Key | `_id` field (automatic) |

## Installing and Running MongoDB

You should have MongoDB installed from Chapter 2. Let's verify it's running.

### Local Installation

```bash
# macOS
brew services start mongodb-community

# Linux
sudo systemctl start mongod

# Windows
# Start MongoDB service from Services app
```

### Docker (Recommended for Development)

```bash
docker run -d -p 27017:27017 --name mongodb mongo
```

### MongoDB Atlas (Cloud)

If you're using Atlas, you already have your connection string. We'll use it later.

## The MongoDB Shell

Connect to your local MongoDB:

```bash
mongosh
```

You should see the MongoDB shell. Try some commands:

```javascript
// Show databases
show dbs

// Create/switch to database
use myapp

// Show current database
db

// Create collection (implicitly created when you insert data)
db.createCollection('users')

// Show collections
show collections

// Insert document
db.users.insertOne({
  username: 'alice',
  email: 'alice@example.com',
  age: 25
})

// Find documents
db.users.find()

// Find one document
db.users.findOne({ username: 'alice' })

// Update document
db.users.updateOne(
  { username: 'alice' },
  { $set: { age: 26 } }
)

// Delete document
db.users.deleteOne({ username: 'alice' })

// Drop collection
db.users.drop()

// Drop database
db.dropDatabase()

// Exit shell
exit
```

The shell is great for quick queries and testing, but you'll interact with MongoDB from Node.js most of the time.

## Using MongoDB from Node.js

MongoDB provides an official Node.js driver. Let's build a simple app.

Install the driver:

```bash
npm install mongodb
```

Create `mongo-demo.js`:

```javascript
import { MongoClient } from 'mongodb';

// Connection URL
const url = 'mongodb://localhost:27017';
const client = new MongoClient(url);

// Database Name
const dbName = 'taskdb';

async function main() {
  try {
    // Connect to server
    await client.connect();
    console.log('Connected to MongoDB');

    const db = client.db(dbName);
    const tasks = db.collection('tasks');

    // Insert a document
    const result = await tasks.insertOne({
      title: 'Learn MongoDB',
      completed: false,
      createdAt: new Date()
    });
    console.log('Inserted task with _id:', result.insertedId);

    // Find all documents
    const allTasks = await tasks.find({}).toArray();
    console.log('All tasks:', allTasks);

    // Find one document
    const task = await tasks.findOne({ title: 'Learn MongoDB' });
    console.log('Found task:', task);

    // Update a document
    await tasks.updateOne(
      { title: 'Learn MongoDB' },
      { $set: { completed: true } }
    );
    console.log('Updated task');

    // Delete a document
    await tasks.deleteOne({ title: 'Learn MongoDB' });
    console.log('Deleted task');

  } catch (error) {
    console.error('Error:', error);
  } finally {
    // Close connection
    await client.close();
  }
}

main();
```

Run it:

```bash
node mongo-demo.js
```

You should see the operations execute successfully.

## CRUD Operations in Detail

### Create (Insert)

**Insert One:**

```javascript
const result = await collection.insertOne({
  title: 'Learn Express',
  completed: false,
  tags: ['backend', 'javascript']
});

console.log('Inserted ID:', result.insertedId);
```

**Insert Many:**

```javascript
const result = await collection.insertMany([
  { title: 'Task 1', completed: false },
  { title: 'Task 2', completed: true },
  { title: 'Task 3', completed: false }
]);

console.log('Inserted count:', result.insertedCount);
console.log('Inserted IDs:', result.insertedIds);
```

### Read (Find)

**Find All:**

```javascript
const tasks = await collection.find({}).toArray();
console.log(tasks);
```

**Find with Query:**

```javascript
// Find incomplete tasks
const incomplete = await collection.find({ completed: false }).toArray();

// Find tasks with specific title
const task = await collection.find({ title: 'Learn Express' }).toArray();
```

**Find One:**

```javascript
const task = await collection.findOne({ title: 'Learn Express' });
```

**Find with Conditions:**

```javascript
// Greater than
const tasks = await collection.find({ priority: { $gt: 5 } }).toArray();

// Less than or equal
const tasks = await collection.find({ priority: { $lte: 3 } }).toArray();

// Not equal
const tasks = await collection.find({ status: { $ne: 'done' } }).toArray();

// In array
const tasks = await collection.find({ status: { $in: ['pending', 'active'] } }).toArray();

// Contains (regex)
const tasks = await collection.find({ title: /express/i }).toArray();

// AND condition (implicit)
const tasks = await collection.find({
  completed: false,
  priority: { $gt: 5 }
}).toArray();

// OR condition
const tasks = await collection.find({
  $or: [
    { completed: true },
    { priority: { $gt: 8 } }
  ]
}).toArray();
```

**Projection (Select Specific Fields):**

```javascript
// Include only title and completed fields
const tasks = await collection.find(
  {},
  { projection: { title: 1, completed: 1 } }
).toArray();

// Exclude _id
const tasks = await collection.find(
  {},
  { projection: { _id: 0, title: 1, completed: 1 } }
).toArray();
```

**Sorting:**

```javascript
// Sort ascending
const tasks = await collection.find({}).sort({ title: 1 }).toArray();

// Sort descending
const tasks = await collection.find({}).sort({ createdAt: -1 }).toArray();

// Multiple fields
const tasks = await collection.find({})
  .sort({ completed: 1, priority: -1 })
  .toArray();
```

**Pagination:**

```javascript
const page = 1;
const pageSize = 10;

const tasks = await collection.find({})
  .skip((page - 1) * pageSize)
  .limit(pageSize)
  .toArray();

// Get total count
const total = await collection.countDocuments({});
const pages = Math.ceil(total / pageSize);
```

### Update

**Update One:**

```javascript
// Set field
await collection.updateOne(
  { title: 'Learn Express' },
  { $set: { completed: true, updatedAt: new Date() } }
);

// Increment number
await collection.updateOne(
  { title: 'Learn Express' },
  { $inc: { views: 1 } }
);

// Add to array
await collection.updateOne(
  { title: 'Learn Express' },
  { $push: { tags: 'new-tag' } }
);

// Remove from array
await collection.updateOne(
  { title: 'Learn Express' },
  { $pull: { tags: 'old-tag' } }
);

// Unset (remove) field
await collection.updateOne(
  { title: 'Learn Express' },
  { $unset: { tempField: '' } }
);
```

**Update Many:**

```javascript
// Mark all incomplete tasks as archived
const result = await collection.updateMany(
  { completed: false },
  { $set: { archived: true } }
);

console.log('Modified count:', result.modifiedCount);
```

**Replace One:**

```javascript
// Completely replace document (except _id)
await collection.replaceOne(
  { title: 'Old Title' },
  {
    title: 'New Title',
    completed: false,
    createdAt: new Date()
  }
);
```

**Upsert (Update or Insert):**

```javascript
// Update if exists, insert if not
await collection.updateOne(
  { title: 'Learn Express' },
  { $set: { completed: false } },
  { upsert: true }
);
```

### Delete

**Delete One:**

```javascript
const result = await collection.deleteOne({ title: 'Learn Express' });
console.log('Deleted count:', result.deletedCount);
```

**Delete Many:**

```javascript
// Delete all completed tasks
const result = await collection.deleteMany({ completed: true });
console.log('Deleted count:', result.deletedCount);

// Delete all documents
const result = await collection.deleteMany({});
```

## Query Operators

MongoDB has powerful query operators:

### Comparison

```javascript
$eq   // Equal
$ne   // Not equal
$gt   // Greater than
$gte  // Greater than or equal
$lt   // Less than
$lte  // Less than or equal
$in   // In array
$nin  // Not in array
```

### Logical

```javascript
$and  // AND
$or   // OR
$not  // NOT
$nor  // NOR
```

### Element

```javascript
$exists  // Field exists
$type    // Field type

// Example
await collection.find({ email: { $exists: true } }).toArray();
```

### Array

```javascript
$all      // Array contains all elements
$elemMatch // Array element matches condition
$size     // Array size

// Example
await collection.find({ tags: { $all: ['javascript', 'backend'] } }).toArray();
```

## Indexes

Indexes make queries faster. Without an index, MongoDB scans every document. With an index, it goes straight to what you need.

```javascript
// Create index on single field
await collection.createIndex({ username: 1 }); // 1 = ascending

// Create unique index
await collection.createIndex({ email: 1 }, { unique: true });

// Compound index
await collection.createIndex({ completed: 1, createdAt: -1 });

// Text index (for text search)
await collection.createIndex({ title: 'text', description: 'text' });

// Text search
const results = await collection.find({ $text: { $search: 'mongodb tutorial' } }).toArray();

// List indexes
const indexes = await collection.listIndexes().toArray();
console.log(indexes);

// Drop index
await collection.dropIndex('username_1');
```

**When to use indexes:**
- Fields you query frequently
- Fields you sort by
- Unique fields (like email)

**When NOT to use indexes:**
- Fields that change frequently (indexes slow down writes)
- Small collections (overhead isn't worth it)
- Fields you rarely query

## Aggregation Pipeline

The aggregation pipeline transforms documents through multiple stages. Think of it as a data processing pipeline.

```javascript
const results = await collection.aggregate([
  // Stage 1: Filter documents
  { $match: { completed: false } },

  // Stage 2: Group by user
  {
    $group: {
      _id: '$userId',
      totalTasks: { $sum: 1 },
      averagePriority: { $avg: '$priority' }
    }
  },

  // Stage 3: Sort by total tasks
  { $sort: { totalTasks: -1 } },

  // Stage 4: Limit to top 10
  { $limit: 10 }
]).toArray();

console.log(results);
```

### Common Aggregation Stages

**$match** - Filter documents:

```javascript
{ $match: { completed: false } }
```

**$group** - Group documents:

```javascript
{
  $group: {
    _id: '$category',
    count: { $sum: 1 },
    total: { $sum: '$price' },
    average: { $avg: '$price' },
    max: { $max: '$price' },
    min: { $min: '$price' }
  }
}
```

**$project** - Select/reshape fields:

```javascript
{
  $project: {
    _id: 0,
    title: 1,
    year: { $year: '$createdAt' },
    fullName: { $concat: ['$firstName', ' ', '$lastName'] }
  }
}
```

**$sort** - Sort documents:

```javascript
{ $sort: { createdAt: -1 } }
```

**$limit** and **$skip** - Pagination:

```javascript
{ $skip: 10 }
{ $limit: 10 }
```

**$lookup** - Join collections:

```javascript
{
  $lookup: {
    from: 'users',           // Collection to join
    localField: 'userId',    // Field in current collection
    foreignField: '_id',     // Field in users collection
    as: 'user'              // Output array field
  }
}
```

**$unwind** - Deconstruct array:

```javascript
// Before: { _id: 1, tags: ['a', 'b', 'c'] }
{ $unwind: '$tags' }
// After:
// { _id: 1, tags: 'a' }
// { _id: 1, tags: 'b' }
// { _id: 1, tags: 'c' }
```

### Real-World Aggregation Example

```javascript
// Get statistics about tasks per user
const stats = await tasks.aggregate([
  // Join with users collection
  {
    $lookup: {
      from: 'users',
      localField: 'userId',
      foreignField: '_id',
      as: 'user'
    }
  },

  // Unwind user array (will be single element)
  { $unwind: '$user' },

  // Group by user
  {
    $group: {
      _id: '$user._id',
      username: { $first: '$user.username' },
      totalTasks: { $sum: 1 },
      completedTasks: {
        $sum: { $cond: ['$completed', 1, 0] }
      },
      incompleteTasks: {
        $sum: { $cond: ['$completed', 0, 1] }
      }
    }
  },

  // Add completion percentage
  {
    $project: {
      username: 1,
      totalTasks: 1,
      completedTasks: 1,
      incompleteTasks: 1,
      completionRate: {
        $multiply: [
          { $divide: ['$completedTasks', '$totalTasks'] },
          100
        ]
      }
    }
  },

  // Sort by completion rate
  { $sort: { completionRate: -1 } }
]).toArray();

console.log(stats);
```

## Data Modeling in MongoDB

### Embedded Documents vs. References

**Embedded (Denormalized):**

Good when data is always accessed together.

```javascript
// User with embedded addresses
{
  _id: ObjectId("..."),
  username: "alice",
  email: "alice@example.com",
  addresses: [
    {
      street: "123 Main St",
      city: "New York",
      type: "home"
    },
    {
      street: "456 Office Ave",
      city: "New York",
      type: "work"
    }
  ]
}
```

**Referenced (Normalized):**

Good when data is accessed independently or used across multiple documents.

```javascript
// User
{
  _id: ObjectId("user123"),
  username: "alice",
  email: "alice@example.com"
}

// Posts (separate collection)
{
  _id: ObjectId("post456"),
  userId: ObjectId("user123"),
  title: "My Post",
  content: "Content here"
}
```

### One-to-One Relationships

Usually embedded:

```javascript
{
  _id: ObjectId("..."),
  username: "alice",
  profile: {
    bio: "Developer",
    avatar: "url",
    website: "example.com"
  }
}
```

### One-to-Many Relationships

**Few** - Embed:

```javascript
{
  _id: ObjectId("..."),
  product: "Laptop",
  reviews: [
    { user: "alice", rating: 5, comment: "Great!" },
    { user: "bob", rating: 4, comment: "Good" }
  ]
}
```

**Many** - Reference:

```javascript
// Blog post
{
  _id: ObjectId("post123"),
  title: "My Post",
  authorId: ObjectId("user456")
}

// Comments (separate collection)
{
  _id: ObjectId("comment789"),
  postId: ObjectId("post123"),
  userId: ObjectId("user456"),
  text: "Great post!"
}
```

### Many-to-Many Relationships

Use references or a join collection:

```javascript
// Students
{
  _id: ObjectId("student1"),
  name: "Alice",
  courseIds: [
    ObjectId("course1"),
    ObjectId("course2")
  ]
}

// Courses
{
  _id: ObjectId("course1"),
  title: "Math 101",
  studentIds: [
    ObjectId("student1"),
    ObjectId("student2")
  ]
}
```

Or use a separate enrollment collection:

```javascript
// Enrollments
{
  _id: ObjectId("..."),
  studentId: ObjectId("student1"),
  courseId: ObjectId("course1"),
  enrolledAt: new Date(),
  grade: null
}
```

### Rules of Thumb

1. **Embed** when:
   - Data is always accessed together
   - One-to-few relationships
   - Data doesn't change frequently
   - Document size stays reasonable (< 16MB limit)

2. **Reference** when:
   - Data is accessed independently
   - One-to-many or many-to-many
   - Data changes frequently
   - Need to avoid duplication

## Transactions

MongoDB supports multi-document transactions (from v4.0):

```javascript
const session = client.startSession();

try {
  await session.withTransaction(async () => {
    // Transfer money between accounts
    await accounts.updateOne(
      { _id: 'account1' },
      { $inc: { balance: -100 } },
      { session }
    );

    await accounts.updateOne(
      { _id: 'account2' },
      { $inc: { balance: 100 } },
      { session }
    );

    // Both succeed or both fail
  });

  console.log('Transaction committed');
} catch (error) {
  console.error('Transaction aborted:', error);
} finally {
  await session.endSession();
}
```

Transactions are useful but have overhead. Use them only when necessary.

## Practical Example: Task Manager Database

Let's build a complete task manager database layer.

Create `taskDatabase.js`:

```javascript
import { MongoClient, ObjectId } from 'mongodb';

const url = process.env.MONGODB_URL || 'mongodb://localhost:27017';
const dbName = 'taskmanager';

class TaskDatabase {
  constructor() {
    this.client = new MongoClient(url);
    this.db = null;
    this.tasks = null;
    this.users = null;
  }

  async connect() {
    await this.client.connect();
    this.db = this.client.db(dbName);
    this.tasks = this.db.collection('tasks');
    this.users = this.db.collection('users');

    // Create indexes
    await this.tasks.createIndex({ userId: 1 });
    await this.tasks.createIndex({ completed: 1 });
    await this.tasks.createIndex({ createdAt: -1 });
    await this.users.createIndex({ email: 1 }, { unique: true });

    console.log('Connected to MongoDB');
  }

  async close() {
    await this.client.close();
    console.log('Disconnected from MongoDB');
  }

  // User methods
  async createUser(userData) {
    const user = {
      ...userData,
      createdAt: new Date()
    };
    const result = await this.users.insertOne(user);
    return { _id: result.insertedId, ...user };
  }

  async getUserByEmail(email) {
    return await this.users.findOne({ email });
  }

  async getUserById(userId) {
    return await this.users.findOne({ _id: new ObjectId(userId) });
  }

  // Task methods
  async createTask(userId, taskData) {
    const task = {
      userId: new ObjectId(userId),
      title: taskData.title,
      description: taskData.description || '',
      completed: false,
      priority: taskData.priority || 0,
      tags: taskData.tags || [],
      createdAt: new Date(),
      updatedAt: new Date()
    };

    const result = await this.tasks.insertOne(task);
    return { _id: result.insertedId, ...task };
  }

  async getTasksByUser(userId, filters = {}) {
    const query = { userId: new ObjectId(userId) };

    if (filters.completed !== undefined) {
      query.completed = filters.completed;
    }

    if (filters.tags && filters.tags.length > 0) {
      query.tags = { $in: filters.tags };
    }

    const tasks = await this.tasks
      .find(query)
      .sort({ createdAt: -1 })
      .toArray();

    return tasks;
  }

  async getTaskById(taskId) {
    return await this.tasks.findOne({ _id: new ObjectId(taskId) });
  }

  async updateTask(taskId, updates) {
    const result = await this.tasks.updateOne(
      { _id: new ObjectId(taskId) },
      {
        $set: {
          ...updates,
          updatedAt: new Date()
        }
      }
    );

    return result.modifiedCount > 0;
  }

  async deleteTask(taskId) {
    const result = await this.tasks.deleteOne({ _id: new ObjectId(taskId) });
    return result.deletedCount > 0;
  }

  async getTaskStats(userId) {
    const stats = await this.tasks.aggregate([
      { $match: { userId: new ObjectId(userId) } },
      {
        $group: {
          _id: null,
          total: { $sum: 1 },
          completed: {
            $sum: { $cond: ['$completed', 1, 0] }
          },
          incomplete: {
            $sum: { $cond: ['$completed', 0, 1] }
          },
          avgPriority: { $avg: '$priority' }
        }
      }
    ]).toArray();

    return stats[0] || {
      total: 0,
      completed: 0,
      incomplete: 0,
      avgPriority: 0
    };
  }

  async searchTasks(userId, searchText) {
    // Create text index if not exists
    try {
      await this.tasks.createIndex({ title: 'text', description: 'text' });
    } catch (e) {
      // Index might already exist
    }

    return await this.tasks
      .find({
        userId: new ObjectId(userId),
        $text: { $search: searchText }
      })
      .toArray();
  }
}

export default TaskDatabase;
```

Use it:

```javascript
import TaskDatabase from './taskDatabase.js';

const db = new TaskDatabase();

async function demo() {
  await db.connect();

  try {
    // Create user
    const user = await db.createUser({
      username: 'alice',
      email: 'alice@example.com',
      password: 'hashed_password_here'
    });
    console.log('Created user:', user);

    // Create tasks
    const task1 = await db.createTask(user._id, {
      title: 'Learn MongoDB',
      description: 'Study database operations',
      priority: 8,
      tags: ['learning', 'database']
    });

    const task2 = await db.createTask(user._id, {
      title: 'Build API',
      description: 'Create RESTful API',
      priority: 9,
      tags: ['backend', 'api']
    });

    // Get all tasks
    const tasks = await db.getTasksByUser(user._id);
    console.log('All tasks:', tasks);

    // Update task
    await db.updateTask(task1._id, { completed: true });

    // Get stats
    const stats = await db.getTaskStats(user._id);
    console.log('Task stats:', stats);

    // Search tasks
    const results = await db.searchTasks(user._id, 'mongodb');
    console.log('Search results:', results);

  } finally {
    await db.close();
  }
}

demo();
```

## MongoDB Best Practices

### 1. Design for Your Queries

Model data based on how you'll access it, not based on "proper" normalization.

### 2. Limit Document Size

Documents have a 16MB limit. Keep them reasonable (< 1MB ideally).

### 3. Use Indexes Wisely

Index fields you query frequently, but don't over-index.

### 4. Handle Connection Pooling

Reuse connections; don't create new clients for every operation.

```javascript
// Good - single client, reused
const client = new MongoClient(url);
await client.connect();

// Bad - new client every time
async function badExample() {
  const client = new MongoClient(url);
  await client.connect();
  // ...
  await client.close();
}
```

### 5. Use Projection

Don't fetch fields you don't need:

```javascript
// Bad - fetches everything
const users = await collection.find({}).toArray();

// Good - fetches only what's needed
const users = await collection.find(
  {},
  { projection: { _id: 1, username: 1, email: 1 } }
).toArray();
```

### 6. Validate Data

MongoDB is schema-flexible, not schema-less. Validate data before inserting.

### 7. Use Appropriate Data Types

```javascript
// Good
{
  age: 25,                    // Number
  active: true,               // Boolean
  createdAt: new Date(),      // Date
  _id: new ObjectId()         // ObjectId
}

// Bad
{
  age: "25",                  // String (should be number)
  active: "true",             // String (should be boolean)
  createdAt: "2024-01-01"     // String (should be Date)
}
```

### 8. Handle Errors Properly

```javascript
try {
  await collection.insertOne(document);
} catch (error) {
  if (error.code === 11000) {
    console.error('Duplicate key error');
  } else {
    console.error('Database error:', error);
  }
}
```

## Common Mistakes

### 1. Not Using ObjectId for IDs

```javascript
// Bad
const task = await collection.findOne({ _id: taskId });

// Good
const task = await collection.findOne({ _id: new ObjectId(taskId) });
```

### 2. Not Closing Cursors

```javascript
// Bad - cursor not closed
const cursor = collection.find({});
while (await cursor.hasNext()) {
  const doc = await cursor.next();
  // process doc
}

// Good - using toArray()
const docs = await collection.find({}).toArray();

// Also good - using for await
const cursor = collection.find({});
for await (const doc of cursor) {
  // process doc
}
```

### 3. Over-Embedding

Don't embed unlimited arrays:

```javascript
// Bad - array grows unbounded
{
  userId: 1,
  comments: [
    // Could grow to thousands...
  ]
}

// Good - reference separate collection
{
  userId: 1,
  commentCount: 145
}
// Comments in separate collection
```

### 4. Not Handling Connection Errors

Always handle connection failures gracefully.

## What We've Covered

You now know:

- What MongoDB is and when to use it
- SQL vs. NoSQL differences
- CRUD operations in detail
- Query operators and filters
- Indexes and when to use them
- Aggregation pipeline
- Data modeling strategies
- Transactions
- Best practices and common mistakes

MongoDB is powerful but requires thoughtful design. Unlike SQL databases that enforce structure, MongoDB trusts you to organize data sensibly.

## What's Next

In the next chapter, we'll introduce Mongoose—an ODM (Object Document Mapper) for MongoDB. While the raw MongoDB driver works fine, Mongoose adds schemas, validation, middleware, and convenient helpers that make working with MongoDB much easier. Think of it as TypeScript for MongoDB—optional, but highly recommended.

---

*"MongoDB gives you enough rope to hang yourself, but also enough flexibility to build amazing things. Choose wisely."* — Senior Developer Who Has Seen Things
