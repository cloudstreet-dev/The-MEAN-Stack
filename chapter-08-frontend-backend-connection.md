# Chapter 8: Connecting Frontend and Backend

## The Full Stack Comes Together

You've built a backend with Express and MongoDB. You've built a frontend with Angular. Now it's time to connect them and create a truly full-stack application.

This chapter covers:
- Configuring CORS to allow cross-origin requests
- Creating an Angular service that communicates with your API
- Handling errors and loading states
- Managing application state
- Building a complete feature from end to end

By the end, your Angular app will talk to your Express API, which will talk to MongoDB, and everything will work together beautifully.

## Setting Up CORS

When your Angular app (running on `localhost:4200`) tries to talk to your Express API (running on `localhost:3000`), browsers block the request by default for security reasons. This is called the Cross-Origin Resource Sharing (CORS) policy.

You need to configure your Express server to allow requests from your Angular app.

Install the CORS package:

```bash
npm install cors
```

Update your Express server (`server.js` or `index.js`):

```javascript
import express from 'express';
import cors from 'cors';
import 'dotenv/config';
import connectDatabase from './database.js';
import taskRoutes from './routes/tasks.js';
import authRoutes from './routes/auth.js';

const app = express();
const PORT = process.env.PORT || 3000;

// CORS configuration
app.use(cors({
  origin: 'http://localhost:4200', // Angular dev server
  credentials: true // Allow cookies/auth headers
}));

// Or allow all origins (development only!)
// app.use(cors());

// Middleware
app.use(express.json());

// Routes
app.use('/api/tasks', taskRoutes);
app.use('/api/auth', authRoutes);

// Error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: err.message });
});

// Connect to database and start server
connectDatabase().then(() => {
  app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
  });
});
```

Now your Angular app can make requests to your Express API.

## Creating the Backend API

Let's build a complete backend for tasks. Create `models/Task.js`:

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
  }
}, {
  timestamps: true
});

const Task = mongoose.model('Task', taskSchema);

export default Task;
```

Create `routes/tasks.js`:

```javascript
import express from 'express';
import Task from '../models/Task.js';

const router = express.Router();

// Get all tasks
router.get('/', async (req, res) => {
  try {
    const tasks = await Task.find()
      .sort({ createdAt: -1 })
      .limit(100);

    res.json(tasks);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Get single task
router.get('/:id', async (req, res) => {
  try {
    const task = await Task.findById(req.params.id);

    if (!task) {
      return res.status(404).json({ error: 'Task not found' });
    }

    res.json(task);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Create task
router.post('/', async (req, res) => {
  try {
    const task = new Task({
      ...req.body,
      userId: '000000000000000000000001' // Hardcoded for now, will use auth later
    });

    await task.save();
    res.status(201).json(task);
  } catch (error) {
    if (error.name === 'ValidationError') {
      return res.status(400).json({ error: error.message });
    }
    res.status(500).json({ error: error.message });
  }
});

// Update task
router.put('/:id', async (req, res) => {
  try {
    const task = await Task.findByIdAndUpdate(
      req.params.id,
      req.body,
      { new: true, runValidators: true }
    );

    if (!task) {
      return res.status(404).json({ error: 'Task not found' });
    }

    res.json(task);
  } catch (error) {
    if (error.name === 'ValidationError') {
      return res.status(400).json({ error: error.message });
    }
    res.status(500).json({ error: error.message });
  }
});

// Delete task
router.delete('/:id', async (req, res) => {
  try {
    const task = await Task.findByIdAndDelete(req.params.id);

    if (!task) {
      return res.status(404).json({ error: 'Task not found' });
    }

    res.json({ message: 'Task deleted', task });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

export default router;
```

Start your backend:

```bash
npm run dev
```

Test it with curl or Postman:

```bash
curl http://localhost:3000/api/tasks
```

## Creating the Angular Service

Now let's create an Angular service that communicates with the API.

First, create a Task interface in `src/app/models/task.model.ts`:

```typescript
export interface Task {
  _id?: string;
  title: string;
  description?: string;
  completed: boolean;
  priority: number;
  dueDate?: string;
  tags?: string[];
  userId?: string;
  createdAt?: string;
  updatedAt?: string;
}
```

Create the service:

```bash
ng g service services/task
```

Edit `task.service.ts`:

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError, map, tap } from 'rxjs/operators';
import { Task } from '../models/task.model';

@Injectable({
  providedIn: 'root'
})
export class TaskService {
  private http = inject(HttpClient);
  private apiUrl = 'http://localhost:3000/api/tasks';

  getTasks(): Observable<Task[]> {
    return this.http.get<Task[]>(this.apiUrl).pipe(
      tap(tasks => console.log('Fetched tasks:', tasks.length)),
      catchError(this.handleError)
    );
  }

  getTask(id: string): Observable<Task> {
    return this.http.get<Task>(`${this.apiUrl}/${id}`).pipe(
      tap(task => console.log('Fetched task:', task.title)),
      catchError(this.handleError)
    );
  }

  createTask(task: Partial<Task>): Observable<Task> {
    return this.http.post<Task>(this.apiUrl, task).pipe(
      tap(newTask => console.log('Created task:', newTask.title)),
      catchError(this.handleError)
    );
  }

  updateTask(id: string, updates: Partial<Task>): Observable<Task> {
    return this.http.put<Task>(`${this.apiUrl}/${id}`, updates).pipe(
      tap(task => console.log('Updated task:', task.title)),
      catchError(this.handleError)
    );
  }

  deleteTask(id: string): Observable<any> {
    return this.http.delete(`${this.apiUrl}/${id}`).pipe(
      tap(() => console.log('Deleted task:', id)),
      catchError(this.handleError)
    );
  }

  private handleError(error: HttpErrorResponse) {
    let errorMessage = 'An error occurred';

    if (error.error instanceof ErrorEvent) {
      // Client-side error
      errorMessage = `Error: ${error.error.message}`;
    } else {
      // Server-side error
      errorMessage = `Error Code: ${error.status}\nMessage: ${error.message}`;
      if (error.error?.error) {
        errorMessage = error.error.error;
      }
    }

    console.error(errorMessage);
    return throwError(() => new Error(errorMessage));
  }
}
```

## Building the Task List Component

Create a component to display tasks:

```bash
ng g c components/task-list
```

Edit `task-list.component.ts`:

```typescript
import { Component, inject, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { TaskService } from '../../services/task.service';
import { Task } from '../../models/task.model';

@Component({
  selector: 'app-task-list',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './task-list.component.html',
  styleUrl: './task-list.component.css'
})
export class TaskListComponent implements OnInit {
  private taskService = inject(TaskService);

  tasks: Task[] = [];
  loading = false;
  error = '';

  ngOnInit() {
    this.loadTasks();
  }

  loadTasks() {
    this.loading = true;
    this.error = '';

    this.taskService.getTasks().subscribe({
      next: (tasks) => {
        this.tasks = tasks;
        this.loading = false;
      },
      error: (err) => {
        this.error = err.message;
        this.loading = false;
      }
    });
  }

  toggleComplete(task: Task) {
    if (!task._id) return;

    this.taskService.updateTask(task._id, {
      completed: !task.completed
    }).subscribe({
      next: (updatedTask) => {
        const index = this.tasks.findIndex(t => t._id === task._id);
        if (index !== -1) {
          this.tasks[index] = updatedTask;
        }
      },
      error: (err) => {
        console.error('Failed to update task:', err);
      }
    });
  }

  deleteTask(id: string) {
    if (!confirm('Are you sure you want to delete this task?')) {
      return;
    }

    this.taskService.deleteTask(id).subscribe({
      next: () => {
        this.tasks = this.tasks.filter(t => t._id !== id);
      },
      error: (err) => {
        console.error('Failed to delete task:', err);
      }
    });
  }
}
```

Edit `task-list.component.html`:

```html
<div class="task-list">
  <h2>My Tasks</h2>

  <div *ngIf="loading" class="loading">
    Loading tasks...
  </div>

  <div *ngIf="error" class="error">
    {{ error }}
    <button (click)="loadTasks()">Retry</button>
  </div>

  <div *ngIf="!loading && !error">
    <div *ngIf="tasks.length === 0" class="empty">
      No tasks yet. Create one to get started!
    </div>

    <div class="tasks">
      <div *ngFor="let task of tasks" class="task-item">
        <input
          type="checkbox"
          [checked]="task.completed"
          (change)="toggleComplete(task)"
        />

        <div class="task-content">
          <h3 [class.completed]="task.completed">{{ task.title }}</h3>
          <p *ngIf="task.description" class="description">{{ task.description }}</p>

          <div class="metadata">
            <span *ngIf="task.priority > 0" class="priority">
              Priority: {{ task.priority }}
            </span>
            <span *ngIf="task.dueDate" class="due-date">
              Due: {{ task.dueDate | date:'short' }}
            </span>
            <span *ngIf="task.tags && task.tags.length > 0" class="tags">
              <span *ngFor="let tag of task.tags" class="tag">{{ tag }}</span>
            </span>
          </div>
        </div>

        <button (click)="deleteTask(task._id!)" class="delete-btn">Delete</button>
      </div>
    </div>
  </div>
</div>
```

Edit `task-list.component.css`:

```css
.task-list {
  max-width: 800px;
  margin: 20px auto;
  padding: 20px;
}

.loading, .error, .empty {
  text-align: center;
  padding: 40px;
  color: #666;
}

.error {
  color: #d32f2f;
  background: #ffebee;
  border-radius: 4px;
}

.error button {
  margin-left: 10px;
  padding: 5px 15px;
  background: #d32f2f;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.tasks {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.task-item {
  display: flex;
  align-items: start;
  gap: 15px;
  padding: 15px;
  background: #f5f5f5;
  border-radius: 8px;
  transition: all 0.2s;
}

.task-item:hover {
  background: #eeeeee;
}

.task-item input[type="checkbox"] {
  margin-top: 5px;
  cursor: pointer;
}

.task-content {
  flex: 1;
}

.task-content h3 {
  margin: 0 0 5px 0;
  font-size: 18px;
}

.task-content h3.completed {
  text-decoration: line-through;
  color: #999;
}

.description {
  margin: 5px 0;
  color: #666;
  font-size: 14px;
}

.metadata {
  display: flex;
  gap: 15px;
  margin-top: 10px;
  font-size: 12px;
  color: #666;
}

.priority {
  font-weight: bold;
}

.tags {
  display: flex;
  gap: 5px;
}

.tag {
  background: #2196F3;
  color: white;
  padding: 2px 8px;
  border-radius: 12px;
}

.delete-btn {
  padding: 8px 15px;
  background: #d32f2f;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.delete-btn:hover {
  background: #b71c1c;
}
```

## Building the Task Form Component

Create a form to add new tasks:

```bash
ng g c components/task-form
```

Edit `task-form.component.ts`:

```typescript
import { Component, inject, Output, EventEmitter } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormBuilder, FormGroup, Validators, ReactiveFormsModule } from '@angular/forms';
import { TaskService } from '../../services/task.service';
import { Task } from '../../models/task.model';

@Component({
  selector: 'app-task-form',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  templateUrl: './task-form.component.html',
  styleUrl: './task-form.component.css'
})
export class TaskFormComponent {
  private fb = inject(FormBuilder);
  private taskService = inject(TaskService);

  @Output() taskCreated = new EventEmitter<Task>();

  taskForm: FormGroup = this.fb.group({
    title: ['', [Validators.required, Validators.maxLength(200)]],
    description: [''],
    priority: [0, [Validators.min(0), Validators.max(10)]],
    dueDate: [''],
    tags: ['']
  });

  submitting = false;
  error = '';

  get title() {
    return this.taskForm.get('title');
  }

  onSubmit() {
    if (this.taskForm.invalid) {
      this.taskForm.markAllAsTouched();
      return;
    }

    this.submitting = true;
    this.error = '';

    const formValue = this.taskForm.value;
    const task: Partial<Task> = {
      title: formValue.title,
      description: formValue.description,
      completed: false,
      priority: formValue.priority,
      dueDate: formValue.dueDate || undefined,
      tags: formValue.tags ? formValue.tags.split(',').map((t: string) => t.trim()) : []
    };

    this.taskService.createTask(task).subscribe({
      next: (newTask) => {
        this.taskCreated.emit(newTask);
        this.taskForm.reset({ priority: 0 });
        this.submitting = false;
      },
      error: (err) => {
        this.error = err.message;
        this.submitting = false;
      }
    });
  }
}
```

Edit `task-form.component.html`:

```html
<div class="task-form">
  <h3>Add New Task</h3>

  <div *ngIf="error" class="error">
    {{ error }}
  </div>

  <form [formGroup]="taskForm" (ngSubmit)="onSubmit()">
    <div class="form-group">
      <label for="title">Title *</label>
      <input
        id="title"
        type="text"
        formControlName="title"
        placeholder="Enter task title"
      />
      <div *ngIf="title?.invalid && title?.touched" class="field-error">
        <div *ngIf="title?.errors?.['required']">Title is required</div>
        <div *ngIf="title?.errors?.['maxlength']">Title is too long</div>
      </div>
    </div>

    <div class="form-group">
      <label for="description">Description</label>
      <textarea
        id="description"
        formControlName="description"
        placeholder="Enter task description"
        rows="3"
      ></textarea>
    </div>

    <div class="form-row">
      <div class="form-group">
        <label for="priority">Priority (0-10)</label>
        <input
          id="priority"
          type="number"
          formControlName="priority"
          min="0"
          max="10"
        />
      </div>

      <div class="form-group">
        <label for="dueDate">Due Date</label>
        <input
          id="dueDate"
          type="date"
          formControlName="dueDate"
        />
      </div>
    </div>

    <div class="form-group">
      <label for="tags">Tags (comma-separated)</label>
      <input
        id="tags"
        type="text"
        formControlName="tags"
        placeholder="work, urgent, personal"
      />
    </div>

    <button
      type="submit"
      [disabled]="taskForm.invalid || submitting"
      class="submit-btn"
    >
      {{ submitting ? 'Creating...' : 'Create Task' }}
    </button>
  </form>
</div>
```

Edit `task-form.component.css`:

```css
.task-form {
  max-width: 600px;
  margin: 20px auto;
  padding: 20px;
  background: #f5f5f5;
  border-radius: 8px;
}

.task-form h3 {
  margin-top: 0;
}

.error {
  padding: 10px;
  background: #ffebee;
  color: #d32f2f;
  border-radius: 4px;
  margin-bottom: 15px;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: 500;
  color: #333;
}

.form-group input,
.form-group textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  box-sizing: border-box;
}

.form-group input:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #2196F3;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
}

.field-error {
  color: #d32f2f;
  font-size: 12px;
  margin-top: 5px;
}

.submit-btn {
  width: 100%;
  padding: 12px;
  background: #2196F3;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
  font-weight: 500;
}

.submit-btn:hover:not(:disabled) {
  background: #1976D2;
}

.submit-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}
```

## Putting It All Together

Update your main app component to use both components.

Edit `app.component.ts`:

```typescript
import { Component } from '@angular/core';
import { TaskListComponent } from './components/task-list/task-list.component';
import { TaskFormComponent } from './components/task-form/task-form.component';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TaskListComponent, TaskFormComponent],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
export class AppComponent {
  title = 'Task Manager';
}
```

Edit `app.component.html`:

```html
<div class="app">
  <header>
    <h1>{{ title }}</h1>
  </header>

  <main>
    <app-task-form (taskCreated)="onTaskCreated()"></app-task-form>
    <app-task-list #taskList></app-task-list>
  </main>
</div>
```

Add event handler in `app.component.ts`:

```typescript
import { Component, ViewChild } from '@angular/core';
import { TaskListComponent } from './components/task-list/task-list.component';
import { TaskFormComponent } from './components/task-form/task-form.component';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TaskListComponent, TaskFormComponent],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
export class AppComponent {
  title = 'Task Manager';

  @ViewChild('taskList') taskList!: TaskListComponent;

  onTaskCreated() {
    this.taskList.loadTasks();
  }
}
```

Edit `app.component.css`:

```css
.app {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
}

header {
  text-align: center;
  color: white;
  margin-bottom: 30px;
}

header h1 {
  font-size: 36px;
  font-weight: 700;
  margin: 0;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.2);
}

main {
  max-width: 1200px;
  margin: 0 auto;
}
```

## Handling Global State

For larger apps, managing state becomes important. Create a simple state management service:

```bash
ng g service services/task-state
```

Edit `task-state.service.ts`:

```typescript
import { Injectable, inject } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';
import { Task } from '../models/task.model';
import { TaskService } from './task.service';

@Injectable({
  providedIn: 'root'
})
export class TaskStateService {
  private taskService = inject(TaskService);

  private tasksSubject = new BehaviorSubject<Task[]>([]);
  private loadingSubject = new BehaviorSubject<boolean>(false);
  private errorSubject = new BehaviorSubject<string>('');

  tasks$ = this.tasksSubject.asObservable();
  loading$ = this.loadingSubject.asObservable();
  error$ = this.errorSubject.asObservable();

  loadTasks() {
    this.loadingSubject.next(true);
    this.errorSubject.next('');

    this.taskService.getTasks().subscribe({
      next: (tasks) => {
        this.tasksSubject.next(tasks);
        this.loadingSubject.next(false);
      },
      error: (err) => {
        this.errorSubject.next(err.message);
        this.loadingSubject.next(false);
      }
    });
  }

  addTask(task: Partial<Task>) {
    this.taskService.createTask(task).subscribe({
      next: (newTask) => {
        const currentTasks = this.tasksSubject.value;
        this.tasksSubject.next([newTask, ...currentTasks]);
      },
      error: (err) => {
        this.errorSubject.next(err.message);
      }
    });
  }

  updateTask(id: string, updates: Partial<Task>) {
    this.taskService.updateTask(id, updates).subscribe({
      next: (updatedTask) => {
        const currentTasks = this.tasksSubject.value;
        const index = currentTasks.findIndex(t => t._id === id);
        if (index !== -1) {
          const newTasks = [...currentTasks];
          newTasks[index] = updatedTask;
          this.tasksSubject.next(newTasks);
        }
      },
      error: (err) => {
        this.errorSubject.next(err.message);
      }
    });
  }

  deleteTask(id: string) {
    this.taskService.deleteTask(id).subscribe({
      next: () => {
        const currentTasks = this.tasksSubject.value;
        this.tasksSubject.next(currentTasks.filter(t => t._id !== id));
      },
      error: (err) => {
        this.errorSubject.next(err.message);
      }
    });
  }
}
```

Now components can use this shared state instead of managing their own.

## Environment Configuration

Create environment-specific configurations.

Create `src/environments/environment.development.ts`:

```typescript
export const environment = {
  production: false,
  apiUrl: 'http://localhost:3000/api'
};
```

Create `src/environments/environment.ts`:

```typescript
export const environment = {
  production: true,
  apiUrl: 'https://your-api-domain.com/api'
};
```

Use in service:

```typescript
import { environment } from '../../environments/environment';

@Injectable({
  providedIn: 'root'
})
export class TaskService {
  private apiUrl = `${environment.apiUrl}/tasks`;
  // ...
}
```

## Interceptors for Global HTTP Configuration

Create an interceptor to add authentication tokens or handle errors globally:

```bash
ng g interceptor interceptors/auth
```

Edit `auth.interceptor.ts`:

```typescript
import { HttpInterceptorFn } from '@angular/common/http';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  // Get token from localStorage
  const token = localStorage.getItem('authToken');

  // Clone request and add authorization header
  if (token) {
    const clonedReq = req.clone({
      headers: req.headers.set('Authorization', `Bearer ${token}`)
    });
    return next(clonedReq);
  }

  return next(req);
};
```

Register in `app.config.ts`:

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { authInterceptor } from './interceptors/auth.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    // ...
    provideHttpClient(
      withInterceptors([authInterceptor])
    )
  ]
};
```

## Error Handling Best Practices

Create a global error handler service:

```bash
ng g service services/error-handler
```

```typescript
import { Injectable } from '@angular/core';
import { HttpErrorResponse } from '@angular/common/http';

@Injectable({
  providedIn: 'root'
})
export class ErrorHandlerService {
  handleError(error: HttpErrorResponse): string {
    let message = 'An unknown error occurred';

    if (error.error instanceof ErrorEvent) {
      // Client-side error
      message = `Error: ${error.error.message}`;
    } else {
      // Server-side error
      switch (error.status) {
        case 400:
          message = error.error.error || 'Bad request';
          break;
        case 401:
          message = 'Unauthorized. Please log in.';
          break;
        case 403:
          message = 'Forbidden. You don't have permission.';
          break;
        case 404:
          message = 'Resource not found';
          break;
        case 500:
          message = 'Server error. Please try again later.';
          break;
        default:
          message = error.error.error || message;
      }
    }

    return message;
  }

  displayError(message: string) {
    // Could integrate with a toast/notification service
    alert(message);
  }
}
```

## Loading States

Create a loading spinner component:

```bash
ng g c components/loading-spinner
```

```typescript
import { Component, Input } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-loading-spinner',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div *ngIf="loading" class="spinner-overlay">
      <div class="spinner"></div>
      <p *ngIf="message">{{ message }}</p>
    </div>
  `,
  styles: [`
    .spinner-overlay {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      padding: 40px;
    }

    .spinner {
      border: 4px solid #f3f3f3;
      border-top: 4px solid #2196F3;
      border-radius: 50%;
      width: 40px;
      height: 40px;
      animation: spin 1s linear infinite;
    }

    @keyframes spin {
      0% { transform: rotate(0deg); }
      100% { transform: rotate(360deg); }
    }

    p {
      margin-top: 15px;
      color: #666;
    }
  `]
})
export class LoadingSpinnerComponent {
  @Input() loading = false;
  @Input() message = '';
}
```

## Testing the Full Stack

Now test your complete application:

1. Start MongoDB
2. Start your Express server: `npm run dev`
3. Start your Angular app: `ng serve`
4. Open `http://localhost:4200`

You should be able to:
- View tasks from the database
- Create new tasks
- Toggle task completion
- Delete tasks
- See loading states
- Handle errors

Congratulations! You have a working full-stack MEAN application!

## What We've Covered

You now know:

- How to configure CORS for cross-origin requests
- How to create Angular services that communicate with APIs
- How to handle loading states and errors
- How to manage global state
- How to use environment configurations
- How to create HTTP interceptors
- How to build a complete feature from end to end

## What's Next

Your app works, but it's not secure. Anyone can create, edit, or delete tasks. In the next chapter, we'll add authentication and authorization, allowing users to register, log in, and manage only their own tasks. You'll learn about JWT tokens, password hashing, protected routes, and security best practices.

---

*"Connecting frontend to backend is like introducing two friends who speak the same language. Once they start talking, beautiful things happen."* — Full-Stack Developer
