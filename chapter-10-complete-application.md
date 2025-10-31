# Chapter 10: Building Your First Complete MEAN Application

## Bringing It All Together

You've learned all the pieces of the MEAN stack. Now it's time to build a complete, polished application that puts everything together. This chapter walks you through building a full-featured task manager with:

- User authentication and authorization
- Full CRUD operations
- Real-time updates
- Filtering and searching
- Responsive design
- Error handling
- Loading states
- Form validation

This is where theory becomes practice.

## Project Planning

Before writing code, plan your application:

### Features

1. **Authentication**
   - User registration
   - User login
   - Password reset (bonus)
   - Profile management

2. **Task Management**
   - Create tasks
   - View task list
   - Edit tasks
   - Delete tasks
   - Mark tasks complete/incomplete

3. **Organization**
   - Filter by status (all, active, completed)
   - Sort by date, priority, title
   - Search tasks
   - Tag system

4. **User Experience**
   - Responsive design
   - Loading indicators
   - Error messages
   - Success feedback

### Project Structure

**Backend:**
```
task-manager-api/
├── src/
│   ├── models/
│   │   ├── User.js
│   │   ├── Task.js
│   ├── routes/
│   │   ├── auth.js
│   │   ├── tasks.js
│   ├── middleware/
│   │   ├── auth.js
│   │   ├── validate.js
│   ├── utils/
│   │   ├── auth.js
│   │   ├── validators.js
│   ├── database.js
│   ├── app.js
│   ├── server.js
├── .env
├── .gitignore
├── package.json
```

**Frontend:**
```
task-manager-frontend/
├── src/
│   ├── app/
│   │   ├── components/
│   │   │   ├── auth/
│   │   │   │   ├── login/
│   │   │   │   ├── register/
│   │   │   ├── tasks/
│   │   │   │   ├── task-list/
│   │   │   │   ├── task-form/
│   │   │   │   ├── task-item/
│   │   │   │   ├── task-filter/
│   │   │   ├── shared/
│   │   │   │   ├── header/
│   │   │   │   ├── footer/
│   │   │   │   ├── loading/
│   │   ├── services/
│   │   │   ├── auth.service.ts
│   │   │   ├── task.service.ts
│   │   ├── guards/
│   │   │   ├── auth.guard.ts
│   │   ├── interceptors/
│   │   │   ├── auth.interceptor.ts
│   │   ├── models/
│   │   │   ├── user.model.ts
│   │   │   ├── task.model.ts
│   │   ├── app.routes.ts
│   │   ├── app.config.ts
│   ├── environments/
│   │   ├── environment.ts
│   │   ├── environment.development.ts
```

## Backend Implementation

### Complete Task Model with Advanced Features

Update `models/Task.js`:

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
    min: 0,
    max: 10,
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
  },

  completedAt: {
    type: Date
  }
}, {
  timestamps: true
});

// Indexes for performance
taskSchema.index({ userId: 1, completed: 1 });
taskSchema.index({ userId: 1, createdAt: -1 });
taskSchema.index({ userId: 1, dueDate: 1 });
taskSchema.index({ userId: 1, priority: -1 });
taskSchema.index({ title: 'text', description: 'text' });

// Virtual: isOverdue
taskSchema.virtual('isOverdue').get(function() {
  return this.dueDate && !this.completed && this.dueDate < new Date();
});

// Update completedAt when marking complete
taskSchema.pre('save', function(next) {
  if (this.isModified('completed')) {
    if (this.completed) {
      this.completedAt = new Date();
    } else {
      this.completedAt = undefined;
    }
  }
  next();
});

const Task = mongoose.model('Task', taskSchema);

export default Task;
```

### Advanced Task Routes with Filtering and Sorting

Update `routes/tasks.js`:

```javascript
import express from 'express';
import Task from '../models/Task.js';
import { requireAuth } from '../middleware/auth.js';

const router = express.Router();

router.use(requireAuth);

// Get tasks with filtering, sorting, pagination
router.get('/', async (req, res) => {
  try {
    const {
      completed,
      priority,
      tag,
      search,
      sortBy = 'createdAt',
      order = 'desc',
      page = 1,
      limit = 50
    } = req.query;

    // Build query
    const query = { userId: req.user._id };

    if (completed !== undefined) {
      query.completed = completed === 'true';
    }

    if (priority !== undefined) {
      query.priority = { $gte: parseInt(priority) };
    }

    if (tag) {
      query.tags = tag;
    }

    // Text search
    if (search) {
      query.$text = { $search: search };
    }

    // Build sort
    const sort = {};
    sort[sortBy] = order === 'desc' ? -1 : 1;

    // Execute query with pagination
    const tasks = await Task.find(query)
      .sort(sort)
      .limit(parseInt(limit))
      .skip((parseInt(page) - 1) * parseInt(limit));

    const total = await Task.countDocuments(query);

    res.json({
      tasks,
      pagination: {
        page: parseInt(page),
        limit: parseInt(limit),
        total,
        pages: Math.ceil(total / parseInt(limit))
      }
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Get task statistics
router.get('/stats', async (req, res) => {
  try {
    const stats = await Task.aggregate([
      { $match: { userId: req.user._id } },
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
          overdue: {
            $sum: {
              $cond: [
                {
                  $and: [
                    { $eq: ['$completed', false] },
                    { $lt: ['$dueDate', new Date()] },
                    { $ne: ['$dueDate', null] }
                  ]
                },
                1,
                0
              ]
            }
          },
          avgPriority: { $avg: '$priority' }
        }
      }
    ]);

    res.json(stats[0] || {
      total: 0,
      completed: 0,
      incomplete: 0,
      overdue: 0,
      avgPriority: 0
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Rest of CRUD operations (from previous chapters)
// GET /:id, POST /, PUT /:id, DELETE /:id

export default router;
```

## Frontend Implementation

### Task List with Filtering and Sorting

Update `task-list.component.ts`:

```typescript
import { Component, inject, OnInit, signal } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { TaskService } from '../../../services/task.service';
import { Task } from '../../../models/task.model';
import { TaskItemComponent } from '../task-item/task-item.component';
import { TaskFormComponent } from '../task-form/task-form.component';
import { LoadingSpinnerComponent } from '../../shared/loading/loading.component';

type FilterType = 'all' | 'active' | 'completed';
type SortType = 'createdAt' | 'dueDate' | 'priority' | 'title';

@Component({
  selector: 'app-task-list',
  standalone: true,
  imports: [
    CommonModule,
    FormsModule,
    TaskItemComponent,
    TaskFormComponent,
    LoadingSpinnerComponent
  ],
  templateUrl: './task-list.component.html',
  styleUrl: './task-list.component.css'
})
export class TaskListComponent implements OnInit {
  private taskService = inject(TaskService);

  tasks = signal<Task[]>([]);
  stats = signal<any>({});
  loading = signal(false);
  error = signal('');

  // Filters
  filter = signal<FilterType>('all');
  sortBy = signal<SortType>('createdAt');
  sortOrder = signal<'asc' | 'desc'>('desc');
  searchQuery = signal('');

  ngOnInit() {
    this.loadTasks();
    this.loadStats();
  }

  loadTasks() {
    this.loading.set(true);
    this.error.set('');

    const params: any = {
      sortBy: this.sortBy(),
      order: this.sortOrder()
    };

    if (this.filter() !== 'all') {
      params.completed = this.filter() === 'completed';
    }

    if (this.searchQuery()) {
      params.search = this.searchQuery();
    }

    this.taskService.getTasks(params).subscribe({
      next: (response) => {
        this.tasks.set(response.tasks);
        this.loading.set(false);
      },
      error: (err) => {
        this.error.set(err.message);
        this.loading.set(false);
      }
    });
  }

  loadStats() {
    this.taskService.getStats().subscribe({
      next: (stats) => this.stats.set(stats),
      error: (err) => console.error('Failed to load stats:', err)
    });
  }

  onFilterChange(filter: FilterType) {
    this.filter.set(filter);
    this.loadTasks();
  }

  onSortChange(sortBy: SortType) {
    if (this.sortBy() === sortBy) {
      // Toggle order
      this.sortOrder.set(this.sortOrder() === 'asc' ? 'desc' : 'asc');
    } else {
      this.sortBy.set(sortBy);
      this.sortOrder.set('desc');
    }
    this.loadTasks();
  }

  onSearch() {
    this.loadTasks();
  }

  onTaskCreated() {
    this.loadTasks();
    this.loadStats();
  }

  onTaskUpdated() {
    this.loadTasks();
    this.loadStats();
  }

  onTaskDeleted() {
    this.loadTasks();
    this.loadStats();
  }
}
```

### Task Item Component

Create a reusable task item component:

```bash
ng g c components/tasks/task-item
```

```typescript
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { CommonModule } from '@angular/common';
import { Task } from '../../../models/task.model';
import { TaskService } from '../../../services/task.service';

@Component({
  selector: 'app-task-item',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './task-item.component.html',
  styleUrl: './task-item.component.css'
})
export class TaskItemComponent {
  @Input() task!: Task;
  @Output() taskUpdated = new EventEmitter<void>();
  @Output() taskDeleted = new EventEmitter<void>();

  editing = false;
  updating = false;

  constructor(private taskService: TaskService) {}

  toggleComplete() {
    if (!this.task._id) return;

    this.updating = true;
    this.taskService.updateTask(this.task._id, {
      completed: !this.task.completed
    }).subscribe({
      next: () => {
        this.task.completed = !this.task.completed;
        this.updating = false;
        this.taskUpdated.emit();
      },
      error: (err) => {
        console.error('Failed to update task:', err);
        this.updating = false;
      }
    });
  }

  delete() {
    if (!this.task._id) return;
    if (!confirm(`Delete task "${this.task.title}"?`)) return;

    this.taskService.deleteTask(this.task._id).subscribe({
      next: () => {
        this.taskDeleted.emit();
      },
      error: (err) => {
        console.error('Failed to delete task:', err);
      }
    });
  }

  get isOverdue(): boolean {
    return !!this.task.dueDate &&
           !this.task.completed &&
           new Date(this.task.dueDate) < new Date();
  }

  get priorityClass(): string {
    if (this.task.priority >= 8) return 'priority-high';
    if (this.task.priority >= 5) return 'priority-medium';
    return 'priority-low';
  }
}
```

### Dashboard Component

Create a dashboard showing statistics:

```bash
ng g c components/dashboard
```

```typescript
import { Component, inject, OnInit, signal } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterLink } from '@angular/router';
import { TaskService } from '../../services/task.service';
import { AuthService } from '../../services/auth.service';

@Component({
  selector: 'app-dashboard',
  standalone: true,
  imports: [CommonModule, RouterLink],
  templateUrl: './dashboard.component.html',
  styleUrl: './dashboard.component.css'
})
export class DashboardComponent implements OnInit {
  private taskService = inject(TaskService);
  authService = inject(AuthService);

  stats = signal<any>({});
  recentTasks = signal<any[]>([]);
  loading = signal(false);

  ngOnInit() {
    this.loadData();
  }

  loadData() {
    this.loading.set(true);

    // Load stats
    this.taskService.getStats().subscribe({
      next: (stats) => this.stats.set(stats),
      error: (err) => console.error(err)
    });

    // Load recent tasks
    this.taskService.getTasks({ limit: 5 }).subscribe({
      next: (response) => {
        this.recentTasks.set(response.tasks);
        this.loading.set(false);
      },
      error: (err) => {
        console.error(err);
        this.loading.set(false);
      }
    });
  }

  get completionRate(): number {
    const total = this.stats().total || 0;
    if (total === 0) return 0;
    const completed = this.stats().completed || 0;
    return Math.round((completed / total) * 100);
  }
}
```

## Polishing the Application

### Responsive Navigation

Create a header component:

```bash
ng g c components/shared/header
```

```html
<header class="header">
  <div class="container">
    <div class="logo">
      <a routerLink="/">Task Manager</a>
    </div>

    <nav class="nav">
      <a routerLink="/dashboard" routerLinkActive="active">Dashboard</a>
      <a routerLink="/tasks" routerLinkActive="active">Tasks</a>

      @if (authService.isAuthenticated()) {
        <div class="user-menu">
          <span class="username">{{ authService.currentUser()?.username }}</span>
          <button (click)="authService.logout()" class="logout-btn">Logout</button>
        </div>
      }
    </nav>
  </div>
</header>
```

### Toast Notifications

Create a notification service:

```bash
ng g service services/notification
```

```typescript
import { Injectable, signal } from '@angular/core';

export interface Notification {
  id: number;
  type: 'success' | 'error' | 'info';
  message: string;
}

@Injectable({
  providedIn: 'root'
})
export class NotificationService {
  notifications = signal<Notification[]>([]);
  private nextId = 1;

  show(type: 'success' | 'error' | 'info', message: string, duration = 3000) {
    const notification: Notification = {
      id: this.nextId++,
      type,
      message
    };

    this.notifications.update(notifs => [...notifs, notification]);

    if (duration > 0) {
      setTimeout(() => {
        this.remove(notification.id);
      }, duration);
    }
  }

  success(message: string) {
    this.show('success', message);
  }

  error(message: string) {
    this.show('error', message);
  }

  info(message: string) {
    this.show('info', message);
  }

  remove(id: number) {
    this.notifications.update(notifs =>
      notifs.filter(n => n.id !== id)
    );
  }
}
```

### Global Styles

Update `src/styles.css`:

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: #f5f7fa;
  color: #2d3748;
  line-height: 1.6;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

/* Buttons */
.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-primary {
  background: #4299e1;
  color: white;
}

.btn-primary:hover {
  background: #3182ce;
}

.btn-danger {
  background: #f56565;
  color: white;
}

.btn-danger:hover {
  background: #e53e3e;
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* Cards */
.card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  padding: 20px;
  margin-bottom: 20px;
}

/* Form elements */
input,
textarea,
select {
  width: 100%;
  padding: 10px;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  font-size: 14px;
  transition: border-color 0.2s;
}

input:focus,
textarea:focus,
select:focus {
  outline: none;
  border-color: #4299e1;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: 500;
  color: #4a5568;
}

/* Utility classes */
.text-center {
  text-align: center;
}

.mt-4 {
  margin-top: 1rem;
}

.mb-4 {
  margin-bottom: 1rem;
}

.text-error {
  color: #f56565;
}

.text-success {
  color: #48bb78;
}
```

## Testing the Complete Application

### Backend Tests

Create `tests/task.test.js`:

```javascript
import request from 'supertest';
import app from '../src/app.js';
import User from '../src/models/User.js';
import Task from '../src/models/Task.js';
import { connectTestDatabase, closeTestDatabase, clearTestDatabase } from './testSetup.js';

let authToken;
let userId;

beforeAll(async () => {
  await connectTestDatabase();
});

afterAll(async () => {
  await closeTestDatabase();
});

beforeEach(async () => {
  await clearTestDatabase();

  // Create test user
  const response = await request(app)
    .post('/api/auth/register')
    .send({
      username: 'testuser',
      email: 'test@example.com',
      password: 'password123'
    });

  authToken = response.body.token;
  userId = response.body.user._id;
});

describe('Task API', () => {
  describe('POST /api/tasks', () => {
    it('should create a task', async () => {
      const response = await request(app)
        .post('/api/tasks')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          title: 'Test Task',
          description: 'Test description',
          priority: 5
        });

      expect(response.status).toBe(201);
      expect(response.body.title).toBe('Test Task');
    });

    it('should require authentication', async () => {
      const response = await request(app)
        .post('/api/tasks')
        .send({ title: 'Test' });

      expect(response.status).toBe(401);
    });
  });

  describe('GET /api/tasks', () => {
    it('should get user tasks', async () => {
      // Create task
      await request(app)
        .post('/api/tasks')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ title: 'Task 1' });

      const response = await request(app)
        .get('/api/tasks')
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(200);
      expect(response.body.tasks).toHaveLength(1);
    });
  });
});
```

## Deployment Preparation

### Environment Configuration

Create `.env.example`:

```
# MongoDB
MONGODB_URL=mongodb://localhost:27017/taskmanager

# JWT
JWT_SECRET=your-super-secret-key

# Server
PORT=3000
NODE_ENV=development

# Frontend URL
FRONTEND_URL=http://localhost:4200
```

### Production Build Scripts

Update `package.json`:

```json
{
  "scripts": {
    "dev": "nodemon src/server.js",
    "start": "node src/server.js",
    "test": "jest",
    "build": "echo 'No build step for backend'"
  }
}
```

Angular `package.json`:

```json
{
  "scripts": {
    "start": "ng serve",
    "build": "ng build",
    "build:prod": "ng build --configuration=production",
    "test": "ng test"
  }
}
```

## What We've Built

You now have a complete MEAN stack application with:

✅ User authentication and authorization
✅ Full CRUD operations
✅ Filtering, sorting, and search
✅ Responsive design
✅ Error handling
✅ Loading states
✅ Statistics dashboard
✅ Notifications
✅ Tests
✅ Production-ready code

## What We've Covered

- Project planning and structure
- Advanced backend features
- Polished frontend components
- Responsive design
- Testing
- Production preparation

## What's Next

Your application works locally, but it's not yet accessible to others. In the next chapter, we'll deploy your MEAN stack application to the cloud, making it available to the world. You'll learn about hosting options, environment configuration, continuous deployment, and monitoring.

---

*"A complete application is more than working code. It's thoughtful design, error handling, user experience, and attention to detail."* — Professional Developer
