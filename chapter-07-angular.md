# Chapter 7: Angular Fundamentals

## What Is Angular?

Angular is a TypeScript-based framework for building web applications. It's maintained by Google and powers some of the biggest web applications you use daily. Angular provides everything you need to build a modern frontend: components, routing, forms, HTTP client, state management, and more—all in one package.

**Important note:** This chapter covers Angular (version 2+), not AngularJS (version 1.x). AngularJS is the old version from 2010 that was completely rewritten in 2016. When people say "Angular" today, they mean the modern framework. AngularJS is deprecated and you shouldn't use it for new projects.

### Why Angular?

**Pros:**
- Complete framework (batteries included)
- TypeScript by default (catch errors before runtime)
- Strong opinions (less decision fatigue)
- Excellent tooling (CLI, devtools, IDE support)
- Great for large teams and enterprise apps

**Cons:**
- Steeper learning curve than simpler frameworks
- More verbose than alternatives (React, Vue)
- Larger bundle size
- Can be overkill for small projects

If you're building a complex application with a team, Angular's structure and tooling are invaluable. If you're building a simple website, it might be too much framework.

## Creating Your First Angular App

You should have the Angular CLI installed from Chapter 2. Let's verify:

```bash
ng version
```

Create a new Angular application:

```bash
ng new task-manager-frontend
```

The CLI will ask some questions:

1. **Would you like to add Angular routing?** Yes
2. **Which stylesheet format would you like to use?** CSS (or your preference)

The CLI creates a new directory, installs dependencies, and sets up a complete Angular application. This takes a minute.

Navigate into your project:

```bash
cd task-manager-frontend
```

Start the development server:

```bash
ng serve
```

Open your browser to `http://localhost:4200`. You should see the default Angular welcome page.

**Pro tip:** Use `ng serve --open` (or `ng serve -o`) to automatically open the browser.

## Project Structure

Let's explore what Angular CLI generated:

```
task-manager-frontend/
├── src/
│   ├── app/
│   │   ├── app.component.ts      # Root component
│   │   ├── app.component.html    # Root component template
│   │   ├── app.component.css     # Root component styles
│   │   ├── app.component.spec.ts # Root component tests
│   │   ├── app.config.ts         # App configuration
│   │   ├── app.routes.ts         # Routing configuration
│   ├── assets/                   # Static files (images, etc.)
│   ├── index.html                # Main HTML file
│   ├── main.ts                   # Application entry point
│   ├── styles.css                # Global styles
├── angular.json                  # Angular configuration
├── package.json                  # Dependencies
├── tsconfig.json                 # TypeScript configuration
└── README.md
```

The heart of your app is in `src/app/`. That's where you'll spend most of your time.

## Components: The Building Blocks

Everything in Angular is a component. Your app is a tree of components, starting with the root `AppComponent`.

A component consists of:
1. **TypeScript class** - Logic and data
2. **HTML template** - Structure
3. **CSS styles** - Appearance
4. **Metadata** - Configuration (via decorator)

Let's examine the default `app.component.ts`:

```typescript
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
export class AppComponent {
  title = 'task-manager-frontend';
}
```

**Breakdown:**

- `@Component` decorator defines component metadata
- `selector` is the HTML tag name (`<app-root>`)
- `standalone: true` makes it a standalone component (Angular 14+)
- `imports` declares dependencies
- `templateUrl` points to the HTML template
- `styleUrl` points to the CSS file
- `AppComponent` class contains component logic

### Creating a Component

Generate a new component:

```bash
ng generate component components/task-list
```

Or shorthand:

```bash
ng g c components/task-list
```

This creates:
- `task-list.component.ts`
- `task-list.component.html`
- `task-list.component.css`
- `task-list.component.spec.ts`

Let's build a simple task list. Edit `task-list.component.ts`:

```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';

interface Task {
  id: number;
  title: string;
  completed: boolean;
}

@Component({
  selector: 'app-task-list',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './task-list.component.html',
  styleUrl: './task-list.component.css'
})
export class TaskListComponent {
  tasks: Task[] = [
    { id: 1, title: 'Learn Angular', completed: false },
    { id: 2, title: 'Build MEAN app', completed: false },
    { id: 3, title: 'Deploy to production', completed: false }
  ];

  addTask(title: string) {
    const newTask: Task = {
      id: this.tasks.length + 1,
      title,
      completed: false
    };
    this.tasks.push(newTask);
  }

  toggleTask(id: number) {
    const task = this.tasks.find(t => t.id === id);
    if (task) {
      task.completed = !task.completed;
    }
  }

  deleteTask(id: number) {
    this.tasks = this.tasks.filter(t => t.id !== id);
  }
}
```

Edit `task-list.component.html`:

```html
<div class="task-list">
  <h2>My Tasks</h2>

  <div class="tasks">
    <div *ngFor="let task of tasks" class="task-item">
      <input
        type="checkbox"
        [checked]="task.completed"
        (change)="toggleTask(task.id)"
      />
      <span [class.completed]="task.completed">{{ task.title }}</span>
      <button (click)="deleteTask(task.id)">Delete</button>
    </div>
  </div>

  <div *ngIf="tasks.length === 0" class="empty">
    No tasks yet!
  </div>
</div>
```

Edit `task-list.component.css`:

```css
.task-list {
  max-width: 600px;
  margin: 20px auto;
  padding: 20px;
}

.task-item {
  display: flex;
  align-items: center;
  padding: 10px;
  border-bottom: 1px solid #eee;
}

.task-item input {
  margin-right: 10px;
}

.task-item span {
  flex: 1;
}

.task-item .completed {
  text-decoration: line-through;
  color: #999;
}

.task-item button {
  background: #ff4444;
  color: white;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
  border-radius: 3px;
}

.task-item button:hover {
  background: #cc0000;
}

.empty {
  text-align: center;
  color: #999;
  padding: 20px;
}
```

Use the component in `app.component.html`:

```html
<h1>Task Manager</h1>
<app-task-list></app-task-list>
```

And import it in `app.component.ts`:

```typescript
import { Component } from '@angular/core';
import { TaskListComponent } from './components/task-list/task-list.component';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TaskListComponent],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
export class AppComponent {
  title = 'task-manager-frontend';
}
```

Now you should see your task list in the browser!

## Templates: Angular's Superpowered HTML

Angular templates are HTML with extra powers.

### Interpolation

Display component properties:

```html
<h1>{{ title }}</h1>
<p>Total tasks: {{ tasks.length }}</p>
<p>2 + 2 = {{ 2 + 2 }}</p>
```

### Property Binding

Bind HTML properties to component properties:

```html
<!-- Bind to element property -->
<img [src]="imageUrl" [alt]="imageAlt">

<!-- Bind to attribute -->
<div [attr.data-id]="taskId"></div>

<!-- Bind to class -->
<div [class.active]="isActive"></div>
<div [class]="classNames"></div>

<!-- Bind to style -->
<div [style.color]="textColor"></div>
<div [style.font-size.px]="fontSize"></div>
```

### Event Binding

Listen to events:

```html
<!-- Click event -->
<button (click)="handleClick()">Click me</button>

<!-- Input event -->
<input (input)="handleInput($event)">

<!-- Custom events -->
<app-child (customEvent)="handleCustom($event)"></app-child>

<!-- Prevent default -->
<form (submit)="onSubmit($event)">
  <!-- ... -->
</form>
```

In component:

```typescript
handleClick() {
  console.log('Button clicked');
}

handleInput(event: Event) {
  const input = event.target as HTMLInputElement;
  console.log('Input value:', input.value);
}

onSubmit(event: Event) {
  event.preventDefault();
  // Handle form submission
}
```

### Two-Way Binding

Combine property and event binding:

```typescript
import { FormsModule } from '@angular/forms';

@Component({
  // ...
  imports: [FormsModule]
})
export class MyComponent {
  username = '';
}
```

```html
<input [(ngModel)]="username">
<p>Hello, {{ username }}!</p>
```

`[(ngModel)]` is syntactic sugar for:

```html
<input [ngModel]="username" (ngModelChange)="username = $event">
```

### Structural Directives

Modify the DOM structure:

**\*ngIf** - Conditional rendering:

```html
<div *ngIf="isLoggedIn">
  Welcome back!
</div>

<div *ngIf="user; else loading">
  Hello, {{ user.name }}
</div>

<ng-template #loading>
  Loading...
</ng-template>
```

**\*ngFor** - Loop over arrays:

```html
<div *ngFor="let task of tasks">
  {{ task.title }}
</div>

<!-- With index -->
<div *ngFor="let task of tasks; let i = index">
  {{ i + 1 }}. {{ task.title }}
</div>

<!-- Track by for performance -->
<div *ngFor="let task of tasks; trackBy: trackByTaskId">
  {{ task.title }}
</div>
```

```typescript
trackByTaskId(index: number, task: Task): number {
  return task.id;
}
```

**\*ngSwitch** - Multiple conditions:

```html
<div [ngSwitch]="status">
  <p *ngSwitchCase="'pending'">Task is pending</p>
  <p *ngSwitchCase="'active'">Task is active</p>
  <p *ngSwitchCase="'completed'">Task is completed</p>
  <p *ngSwitchDefault>Unknown status</p>
</div>
```

### Attribute Directives

Modify element appearance or behavior:

**ngClass** - Dynamic classes:

```html
<div [ngClass]="{ 'active': isActive, 'disabled': isDisabled }">
  Content
</div>

<div [ngClass]="['class1', 'class2', condition ? 'class3' : '']">
  Content
</div>
```

**ngStyle** - Dynamic styles:

```html
<div [ngStyle]="{ 'color': textColor, 'font-size': fontSize + 'px' }">
  Content
</div>
```

### Template Reference Variables

Reference elements in the template:

```html
<input #nameInput type="text">
<button (click)="greet(nameInput.value)">Greet</button>

<!-- Access in template -->
<p>You typed: {{ nameInput.value }}</p>
```

### Pipes

Transform data for display:

```html
<!-- Uppercase -->
<p>{{ 'hello' | uppercase }}</p>
<!-- Output: HELLO -->

<!-- Lowercase -->
<p>{{ 'HELLO' | lowercase }}</p>
<!-- Output: hello -->

<!-- Date formatting -->
<p>{{ today | date:'short' }}</p>
<p>{{ today | date:'fullDate' }}</p>

<!-- Currency -->
<p>{{ price | currency }}</p>
<p>{{ price | currency:'EUR' }}</p>

<!-- Decimal -->
<p>{{ 3.14159 | number:'1.2-2' }}</p>
<!-- Output: 3.14 -->

<!-- JSON (for debugging) -->
<pre>{{ user | json }}</pre>

<!-- Slice -->
<p>{{ longText | slice:0:100 }}</p>

<!-- Percent -->
<p>{{ 0.75 | percent }}</p>
<!-- Output: 75% -->
```

**Chaining pipes:**

```html
<p>{{ today | date:'short' | uppercase }}</p>
```

**Custom pipe:**

```bash
ng g pipe pipes/truncate
```

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'truncate',
  standalone: true
})
export class TruncatePipe implements PipeTransform {
  transform(value: string, limit: number = 50, ellipsis: string = '...'): string {
    if (!value) return '';
    if (value.length <= limit) return value;
    return value.substring(0, limit) + ellipsis;
  }
}
```

Use it:

```html
<p>{{ longText | truncate:100 }}</p>
```

## Component Communication

### Parent to Child: @Input

Child component:

```typescript
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-task-item',
  standalone: true,
  template: `
    <div class="task">
      <h3>{{ task.title }}</h3>
      <p>Priority: {{ priority }}</p>
    </div>
  `
})
export class TaskItemComponent {
  @Input() task!: Task;
  @Input() priority: number = 0;
}
```

Parent component:

```html
<app-task-item [task]="currentTask" [priority]="5"></app-task-item>
```

### Child to Parent: @Output

Child component:

```typescript
import { Component, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-task-form',
  standalone: true,
  template: `
    <input #titleInput type="text">
    <button (click)="submit(titleInput.value)">Add</button>
  `
})
export class TaskFormComponent {
  @Output() taskCreated = new EventEmitter<string>();

  submit(title: string) {
    if (title.trim()) {
      this.taskCreated.emit(title);
    }
  }
}
```

Parent component:

```html
<app-task-form (taskCreated)="addTask($event)"></app-task-form>
```

```typescript
addTask(title: string) {
  console.log('New task:', title);
}
```

### Template Reference (Parent Access to Child)

```html
<app-child #childComponent></app-child>
<button (click)="childComponent.someMethod()">Call Child Method</button>
```

### Service for Complex Communication

We'll cover this in the Services section.

## Services and Dependency Injection

Services provide shared logic and data across components.

Generate a service:

```bash
ng g service services/task
```

Edit `task.service.ts`:

```typescript
import { Injectable } from '@angular/core';

interface Task {
  id: number;
  title: string;
  completed: boolean;
}

@Injectable({
  providedIn: 'root'
})
export class TaskService {
  private tasks: Task[] = [
    { id: 1, title: 'Learn Angular', completed: false },
    { id: 2, title: 'Build app', completed: false }
  ];

  private nextId = 3;

  getTasks(): Task[] {
    return this.tasks;
  }

  getTask(id: number): Task | undefined {
    return this.tasks.find(t => t.id === id);
  }

  addTask(title: string): Task {
    const task: Task = {
      id: this.nextId++,
      title,
      completed: false
    };
    this.tasks.push(task);
    return task;
  }

  updateTask(id: number, updates: Partial<Task>): boolean {
    const task = this.getTask(id);
    if (task) {
      Object.assign(task, updates);
      return true;
    }
    return false;
  }

  deleteTask(id: number): boolean {
    const index = this.tasks.findIndex(t => t.id === id);
    if (index !== -1) {
      this.tasks.splice(index, 1);
      return true;
    }
    return false;
  }
}
```

Use in component:

```typescript
import { Component, inject } from '@angular/core';
import { TaskService } from '../../services/task.service';

@Component({
  selector: 'app-task-list',
  standalone: true,
  // ...
})
export class TaskListComponent {
  private taskService = inject(TaskService);

  tasks = this.taskService.getTasks();

  addTask(title: string) {
    this.taskService.addTask(title);
    this.tasks = this.taskService.getTasks(); // Refresh
  }

  deleteTask(id: number) {
    this.taskService.deleteTask(id);
    this.tasks = this.taskService.getTasks();
  }
}
```

**Note:** `inject()` is the modern way (Angular 14+). You can also use constructor injection:

```typescript
constructor(private taskService: TaskService) {}
```

## HTTP Client

Angular's HTTP client for making API requests.

Import `provideHttpClient` in `app.config.ts`:

```typescript
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes),
    provideHttpClient()
  ]
};
```

Update `task.service.ts` to use HTTP:

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

interface Task {
  id: number;
  title: string;
  completed: boolean;
}

@Injectable({
  providedIn: 'root'
})
export class TaskService {
  private http = inject(HttpClient);
  private apiUrl = 'http://localhost:3000/api/tasks';

  getTasks(): Observable<Task[]> {
    return this.http.get<Task[]>(this.apiUrl);
  }

  getTask(id: number): Observable<Task> {
    return this.http.get<Task>(`${this.apiUrl}/${id}`);
  }

  addTask(task: Partial<Task>): Observable<Task> {
    return this.http.post<Task>(this.apiUrl, task);
  }

  updateTask(id: number, updates: Partial<Task>): Observable<Task> {
    return this.http.put<Task>(`${this.apiUrl}/${id}`, updates);
  }

  deleteTask(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

Use in component:

```typescript
import { Component, inject, OnInit } from '@angular/core';
import { TaskService } from '../../services/task.service';

@Component({
  selector: 'app-task-list',
  standalone: true,
  // ...
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
    this.taskService.getTasks().subscribe({
      next: (tasks) => {
        this.tasks = tasks;
        this.loading = false;
      },
      error: (err) => {
        this.error = 'Failed to load tasks';
        this.loading = false;
        console.error(err);
      }
    });
  }

  addTask(title: string) {
    this.taskService.addTask({ title }).subscribe({
      next: (task) => {
        this.tasks.push(task);
      },
      error: (err) => {
        console.error('Failed to add task:', err);
      }
    });
  }

  deleteTask(id: number) {
    this.taskService.deleteTask(id).subscribe({
      next: () => {
        this.tasks = this.tasks.filter(t => t.id !== id);
      },
      error: (err) => {
        console.error('Failed to delete task:', err);
      }
    });
  }
}
```

## Routing

Angular Router enables navigation between views.

Routes are defined in `app.routes.ts`:

```typescript
import { Routes } from '@angular/router';
import { TaskListComponent } from './components/task-list/task-list.component';
import { TaskDetailComponent } from './components/task-detail/task-detail.component';
import { HomeComponent } from './components/home/home.component';
import { NotFoundComponent } from './components/not-found/not-found.component';

export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'tasks', component: TaskListComponent },
  { path: 'tasks/:id', component: TaskDetailComponent },
  { path: '**', component: NotFoundComponent }
];
```

In `app.component.html`:

```html
<nav>
  <a routerLink="/">Home</a>
  <a routerLink="/tasks">Tasks</a>
</nav>

<router-outlet></router-outlet>
```

Import `RouterLink` in `app.component.ts`:

```typescript
import { RouterOutlet, RouterLink } from '@angular/router';

@Component({
  // ...
  imports: [RouterOutlet, RouterLink]
})
```

### Navigating Programmatically

```typescript
import { Router } from '@angular/router';

export class MyComponent {
  private router = inject(Router);

  goToTasks() {
    this.router.navigate(['/tasks']);
  }

  goToTask(id: number) {
    this.router.navigate(['/tasks', id]);
  }
}
```

### Route Parameters

```typescript
import { Component, inject, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-task-detail',
  // ...
})
export class TaskDetailComponent implements OnInit {
  private route = inject(ActivatedRoute);
  private taskService = inject(TaskService);

  task: Task | null = null;

  ngOnInit() {
    const id = Number(this.route.snapshot.paramMap.get('id'));
    this.taskService.getTask(id).subscribe({
      next: (task) => this.task = task,
      error: (err) => console.error(err)
    });
  }
}
```

### Query Parameters

```typescript
// Navigate with query params
this.router.navigate(['/tasks'], { queryParams: { filter: 'completed' } });

// Read query params
this.route.queryParams.subscribe(params => {
  const filter = params['filter'];
});
```

### Route Guards

Protect routes with guards:

```bash
ng g guard guards/auth
```

```typescript
import { inject } from '@angular/core';
import { Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

export const authGuard = () => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isLoggedIn()) {
    return true;
  } else {
    return router.createUrlTree(['/login']);
  }
};
```

Use in routes:

```typescript
{
  path: 'tasks',
  component: TaskListComponent,
  canActivate: [authGuard]
}
```

## Forms

### Template-Driven Forms

Import `FormsModule`:

```typescript
import { FormsModule } from '@angular/forms';

@Component({
  // ...
  imports: [FormsModule]
})
```

Use in template:

```html
<form #taskForm="ngForm" (ngSubmit)="onSubmit(taskForm)">
  <div>
    <label>Title:</label>
    <input
      type="text"
      name="title"
      [(ngModel)]="task.title"
      required
      #titleField="ngModel"
    >
    <div *ngIf="titleField.invalid && titleField.touched" class="error">
      Title is required
    </div>
  </div>

  <div>
    <label>Priority:</label>
    <input
      type="number"
      name="priority"
      [(ngModel)]="task.priority"
      min="0"
      max="10"
    >
  </div>

  <button type="submit" [disabled]="taskForm.invalid">Submit</button>
</form>
```

Component:

```typescript
task = {
  title: '',
  priority: 0
};

onSubmit(form: any) {
  if (form.valid) {
    console.log('Form submitted:', this.task);
    form.reset();
  }
}
```

### Reactive Forms (Recommended)

Import `ReactiveFormsModule`:

```typescript
import { ReactiveFormsModule } from '@angular/forms';

@Component({
  // ...
  imports: [ReactiveFormsModule]
})
```

Component:

```typescript
import { Component, inject } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  selector: 'app-task-form',
  // ...
})
export class TaskFormComponent {
  private fb = inject(FormBuilder);

  taskForm: FormGroup = this.fb.group({
    title: ['', [Validators.required, Validators.maxLength(200)]],
    description: [''],
    priority: [0, [Validators.min(0), Validators.max(10)]],
    dueDate: ['']
  });

  onSubmit() {
    if (this.taskForm.valid) {
      console.log('Form value:', this.taskForm.value);
      this.taskForm.reset();
    }
  }

  get title() {
    return this.taskForm.get('title');
  }
}
```

Template:

```html
<form [formGroup]="taskForm" (ngSubmit)="onSubmit()">
  <div>
    <label>Title:</label>
    <input type="text" formControlName="title">
    <div *ngIf="title?.invalid && title?.touched" class="error">
      <div *ngIf="title?.errors?.['required']">Title is required</div>
      <div *ngIf="title?.errors?.['maxlength']">Title too long</div>
    </div>
  </div>

  <div>
    <label>Description:</label>
    <textarea formControlName="description"></textarea>
  </div>

  <div>
    <label>Priority:</label>
    <input type="number" formControlName="priority">
  </div>

  <div>
    <label>Due Date:</label>
    <input type="date" formControlName="dueDate">
  </div>

  <button type="submit" [disabled]="taskForm.invalid">Submit</button>
</form>
```

### Custom Validators

```typescript
import { AbstractControl, ValidationErrors } from '@angular/forms';

export function futureDateValidator(control: AbstractControl): ValidationErrors | null {
  const value = control.value;
  if (!value) return null;

  const selectedDate = new Date(value);
  const today = new Date();

  return selectedDate > today ? null : { notFutureDate: true };
}

// Use it
this.taskForm = this.fb.group({
  dueDate: ['', futureDateValidator]
});
```

## Lifecycle Hooks

Angular components have lifecycle hooks:

```typescript
import { Component, OnInit, OnDestroy, OnChanges, SimpleChanges } from '@angular/core';

@Component({
  // ...
})
export class MyComponent implements OnInit, OnDestroy, OnChanges {
  ngOnInit() {
    // Component initialized (most common)
    console.log('Component initialized');
  }

  ngOnChanges(changes: SimpleChanges) {
    // Input properties changed
    console.log('Changes:', changes);
  }

  ngOnDestroy() {
    // Component destroyed (cleanup here)
    console.log('Component destroyed');
  }

  // Other hooks:
  // ngDoCheck() - custom change detection
  // ngAfterContentInit() - content projected
  // ngAfterContentChecked() - content checked
  // ngAfterViewInit() - view initialized
  // ngAfterViewChecked() - view checked
}
```

Most common hooks:
- `ngOnInit` - Initialize component
- `ngOnDestroy` - Cleanup (unsubscribe, clear timers)

## RxJS and Observables

Angular uses RxJS for async operations.

### Basic Observable

```typescript
import { Observable, of, from } from 'rxjs';

// Create observable
const numbers$ = of(1, 2, 3, 4, 5);

// Subscribe
numbers$.subscribe({
  next: (value) => console.log(value),
  error: (err) => console.error(err),
  complete: () => console.log('Complete')
});
```

### Operators

```typescript
import { map, filter, tap, catchError } from 'rxjs/operators';
import { of } from 'rxjs';

// Map
numbers$.pipe(
  map(n => n * 2)
).subscribe(n => console.log(n));

// Filter
numbers$.pipe(
  filter(n => n % 2 === 0)
).subscribe(n => console.log(n));

// Tap (for side effects)
numbers$.pipe(
  tap(n => console.log('Processing:', n)),
  map(n => n * 2)
).subscribe(n => console.log('Result:', n));

// CatchError
this.http.get('/api/tasks').pipe(
  catchError(error => {
    console.error('Error:', error);
    return of([]); // Return empty array on error
  })
).subscribe(tasks => console.log(tasks));
```

### Unsubscribing

Always unsubscribe to prevent memory leaks:

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Subscription } from 'rxjs';

@Component({
  // ...
})
export class MyComponent implements OnInit, OnDestroy {
  private subscription!: Subscription;

  ngOnInit() {
    this.subscription = this.taskService.getTasks().subscribe(
      tasks => console.log(tasks)
    );
  }

  ngOnDestroy() {
    this.subscription.unsubscribe();
  }
}
```

Or use async pipe (auto-unsubscribes):

```typescript
tasks$ = this.taskService.getTasks();
```

```html
<div *ngFor="let task of tasks$ | async">
  {{ task.title }}
</div>
```

## What We've Covered

You now know:

- What Angular is and when to use it
- Creating components
- Templates and data binding
- Directives and pipes
- Component communication
- Services and dependency injection
- HTTP client
- Routing
- Forms (template-driven and reactive)
- Lifecycle hooks
- RxJS basics

Angular is a comprehensive framework with much more to explore, but you now have the foundation to build real applications.

## What's Next

In the next chapter, we'll connect your Angular frontend to your Express/MongoDB backend. You'll learn how to make HTTP requests, handle authentication, manage state, and build a fully integrated MEAN stack application where the frontend and backend work together seamlessly.

---

*"Angular has a learning curve, but once you climb it, you can see for miles."* — Developer Who Persevered
