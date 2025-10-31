# Chapter 9: Authentication and Security

## Why Security Matters

Right now, your task manager has a fatal flaw: anyone can create, view, edit, or delete any task. There's no concept of users, no login, no permissions. It's like having a house with no locks—convenient until someone you don't like walks in.

This chapter fixes that. You'll learn:
- User registration and login
- Password hashing with bcrypt
- JWT (JSON Web Tokens) for authentication
- Protecting API routes
- Implementing authentication in Angular
- Security best practices

By the end, users will have accounts, passwords will be encrypted, and users can only manage their own tasks.

## Authentication vs. Authorization

**Authentication:** Who are you?
- Proving identity (login with email/password)
- Verifying credentials
- Issuing tokens

**Authorization:** What are you allowed to do?
- Checking permissions
- Role-based access control
- Resource ownership

You need both. Authentication says "You are Alice." Authorization says "Alice can only edit her own tasks."

## Backend: User Model

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
    avatar: String
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
  timestamps: true
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

// Don't return password in JSON
userSchema.methods.toJSON = function() {
  const obj = this.toObject();
  delete obj.password;
  delete obj.__v;
  return obj;
};

// Compare password
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};

const User = mongoose.model('User', userSchema);

export default User;
```

Install bcrypt:

```bash
npm install bcrypt
```

## JWT Authentication

Install JWT library:

```bash
npm install jsonwebtoken
```

Create `utils/auth.js`:

```javascript
import jwt from 'jsonwebtoken';

const JWT_SECRET = process.env.JWT_SECRET || 'your-secret-key-change-this-in-production';
const JWT_EXPIRES_IN = '7d';

export function generateToken(userId) {
  return jwt.sign({ userId }, JWT_SECRET, { expiresIn: JWT_EXPIRES_IN });
}

export function verifyToken(token) {
  try {
    return jwt.verify(token, JWT_SECRET);
  } catch (error) {
    return null;
  }
}
```

**IMPORTANT:** Never hardcode secrets. Use environment variables. Add to `.env`:

```
JWT_SECRET=your-super-secret-key-that-nobody-knows-make-it-long-and-random
```

## Authentication Middleware

Create `middleware/auth.js`:

```javascript
import { verifyToken } from '../utils/auth.js';
import User from '../models/User.js';

export async function requireAuth(req, res, next) {
  try {
    // Get token from header
    const authHeader = req.headers.authorization;

    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return res.status(401).json({ error: 'No token provided' });
    }

    const token = authHeader.substring(7); // Remove 'Bearer '

    // Verify token
    const decoded = verifyToken(token);

    if (!decoded) {
      return res.status(401).json({ error: 'Invalid token' });
    }

    // Get user
    const user = await User.findById(decoded.userId).select('-password');

    if (!user || !user.isActive) {
      return res.status(401).json({ error: 'User not found' });
    }

    // Attach user to request
    req.user = user;
    next();
  } catch (error) {
    res.status(500).json({ error: 'Authentication error' });
  }
}

export function requireAdmin(req, res, next) {
  if (req.user.role !== 'admin') {
    return res.status(403).json({ error: 'Admin access required' });
  }
  next();
}
```

## Authentication Routes

Create `routes/auth.js`:

```javascript
import express from 'express';
import User from '../models/User.js';
import { generateToken } from '../utils/auth.js';
import { requireAuth } from '../middleware/auth.js';

const router = express.Router();

// Register
router.post('/register', async (req, res) => {
  try {
    const { username, email, password } = req.body;

    // Check if user exists
    const existingUser = await User.findOne({
      $or: [{ email }, { username }]
    });

    if (existingUser) {
      if (existingUser.email === email) {
        return res.status(400).json({ error: 'Email already registered' });
      }
      return res.status(400).json({ error: 'Username already taken' });
    }

    // Create user
    const user = new User({
      username,
      email,
      password
    });

    await user.save();

    // Generate token
    const token = generateToken(user._id);

    res.status(201).json({
      message: 'User registered successfully',
      token,
      user
    });
  } catch (error) {
    if (error.name === 'ValidationError') {
      const messages = Object.values(error.errors).map(e => e.message);
      return res.status(400).json({ error: messages.join(', ') });
    }
    res.status(500).json({ error: error.message });
  }
});

// Login
router.post('/login', async (req, res) => {
  try {
    const { email, password } = req.body;

    // Find user
    const user = await User.findOne({ email: email.toLowerCase() });

    if (!user) {
      return res.status(401).json({ error: 'Invalid email or password' });
    }

    // Check password
    const isMatch = await user.comparePassword(password);

    if (!isMatch) {
      return res.status(401).json({ error: 'Invalid email or password' });
    }

    // Check if active
    if (!user.isActive) {
      return res.status(401).json({ error: 'Account is inactive' });
    }

    // Generate token
    const token = generateToken(user._id);

    // Remove password from user object
    const userObj = user.toJSON();

    res.json({
      message: 'Login successful',
      token,
      user: userObj
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Get current user
router.get('/me', requireAuth, async (req, res) => {
  res.json({ user: req.user });
});

// Update profile
router.put('/profile', requireAuth, async (req, res) => {
  try {
    const { firstName, lastName, avatar } = req.body;

    req.user.profile = {
      firstName,
      lastName,
      avatar
    };

    await req.user.save();

    res.json({ user: req.user });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Change password
router.put('/password', requireAuth, async (req, res) => {
  try {
    const { currentPassword, newPassword } = req.body;

    // Get user with password
    const user = await User.findById(req.user._id);

    // Verify current password
    const isMatch = await user.comparePassword(currentPassword);

    if (!isMatch) {
      return res.status(401).json({ error: 'Current password is incorrect' });
    }

    // Set new password
    user.password = newPassword;
    await user.save();

    res.json({ message: 'Password updated successfully' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

export default router;
```

## Protecting Task Routes

Update `routes/tasks.js` to require authentication:

```javascript
import express from 'express';
import Task from '../models/Task.js';
import { requireAuth } from '../middleware/auth.js';

const router = express.Router();

// All task routes require authentication
router.use(requireAuth);

// Get user's tasks
router.get('/', async (req, res) => {
  try {
    const tasks = await Task.find({ userId: req.user._id })
      .sort({ createdAt: -1 })
      .limit(100);

    res.json(tasks);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Get single task (must own it)
router.get('/:id', async (req, res) => {
  try {
    const task = await Task.findOne({
      _id: req.params.id,
      userId: req.user._id
    });

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
      userId: req.user._id
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

// Update task (must own it)
router.put('/:id', async (req, res) => {
  try {
    const task = await Task.findOneAndUpdate(
      { _id: req.params.id, userId: req.user._id },
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

// Delete task (must own it)
router.delete('/:id', async (req, res) => {
  try {
    const task = await Task.findOneAndDelete({
      _id: req.params.id,
      userId: req.user._id
    });

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

## Frontend: Auth Service

Create `src/app/services/auth.service.ts`:

```typescript
import { Injectable, inject, signal } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, tap } from 'rxjs';
import { Router } from '@angular/router';

export interface User {
  _id: string;
  username: string;
  email: string;
  profile?: {
    firstName?: string;
    lastName?: string;
    avatar?: string;
  };
  role: string;
}

interface AuthResponse {
  token: string;
  user: User;
  message: string;
}

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  private http = inject(HttpClient);
  private router = inject(Router);

  private apiUrl = 'http://localhost:3000/api/auth';

  currentUser = signal<User | null>(null);
  isAuthenticated = signal<boolean>(false);

  constructor() {
    // Check for existing token on init
    this.checkAuthStatus();
  }

  register(username: string, email: string, password: string): Observable<AuthResponse> {
    return this.http.post<AuthResponse>(`${this.apiUrl}/register`, {
      username,
      email,
      password
    }).pipe(
      tap(response => this.handleAuthSuccess(response))
    );
  }

  login(email: string, password: string): Observable<AuthResponse> {
    return this.http.post<AuthResponse>(`${this.apiUrl}/login`, {
      email,
      password
    }).pipe(
      tap(response => this.handleAuthSuccess(response))
    );
  }

  logout() {
    localStorage.removeItem('authToken');
    this.currentUser.set(null);
    this.isAuthenticated.set(false);
    this.router.navigate(['/login']);
  }

  getCurrentUser(): Observable<{ user: User }> {
    return this.http.get<{ user: User }>(`${this.apiUrl}/me`).pipe(
      tap(response => {
        this.currentUser.set(response.user);
        this.isAuthenticated.set(true);
      })
    );
  }

  getToken(): string | null {
    return localStorage.getItem('authToken');
  }

  private handleAuthSuccess(response: AuthResponse) {
    localStorage.setItem('authToken', response.token);
    this.currentUser.set(response.user);
    this.isAuthenticated.set(true);
  }

  private checkAuthStatus() {
    const token = this.getToken();
    if (token) {
      this.getCurrentUser().subscribe({
        error: () => {
          // Token invalid, clear it
          this.logout();
        }
      });
    }
  }
}
```

## Frontend: Auth Interceptor

Update `src/app/interceptors/auth.interceptor.ts`:

```typescript
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { AuthService } from '../services/auth.service';
import { catchError, throwError } from 'rxjs';
import { Router } from '@angular/router';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const router = inject(Router);
  const token = authService.getToken();

  // Clone request and add token
  let authReq = req;
  if (token) {
    authReq = req.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`
      }
    });
  }

  return next(authReq).pipe(
    catchError(error => {
      // Handle 401 Unauthorized
      if (error.status === 401) {
        authService.logout();
      }
      return throwError(() => error);
    })
  );
};
```

## Frontend: Login Component

```bash
ng g c components/auth/login
```

Edit `login.component.ts`:

```typescript
import { Component, inject } from '@angular/core';
import { FormBuilder, FormGroup, Validators, ReactiveFormsModule } from '@angular/forms';
import { Router, RouterLink } from '@angular/router';
import { CommonModule } from '@angular/common';
import { AuthService } from '../../../services/auth.service';

@Component({
  selector: 'app-login',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule, RouterLink],
  templateUrl: './login.component.html',
  styleUrl: './login.component.css'
})
export class LoginComponent {
  private fb = inject(FormBuilder);
  private authService = inject(AuthService);
  private router = inject(Router);

  loginForm: FormGroup = this.fb.group({
    email: ['', [Validators.required, Validators.email]],
    password: ['', [Validators.required, Validators.minLength(8)]]
  });

  submitting = false;
  error = '';

  onSubmit() {
    if (this.loginForm.invalid) {
      this.loginForm.markAllAsTouched();
      return;
    }

    this.submitting = true;
    this.error = '';

    const { email, password } = this.loginForm.value;

    this.authService.login(email, password).subscribe({
      next: () => {
        this.router.navigate(['/tasks']);
      },
      error: (err) => {
        this.error = err.error?.error || 'Login failed';
        this.submitting = false;
      }
    });
  }
}
```

Edit `login.component.html`:

```html
<div class="login-container">
  <div class="login-card">
    <h2>Login</h2>

    <div *ngIf="error" class="error">
      {{ error }}
    </div>

    <form [formGroup]="loginForm" (ngSubmit)="onSubmit()">
      <div class="form-group">
        <label for="email">Email</label>
        <input
          id="email"
          type="email"
          formControlName="email"
          placeholder="you@example.com"
        />
        <div *ngIf="loginForm.get('email')?.invalid && loginForm.get('email')?.touched" class="field-error">
          <div *ngIf="loginForm.get('email')?.errors?.['required']">Email is required</div>
          <div *ngIf="loginForm.get('email')?.errors?.['email']">Invalid email format</div>
        </div>
      </div>

      <div class="form-group">
        <label for="password">Password</label>
        <input
          id="password"
          type="password"
          formControlName="password"
          placeholder="••••••••"
        />
        <div *ngIf="loginForm.get('password')?.invalid && loginForm.get('password')?.touched" class="field-error">
          <div *ngIf="loginForm.get('password')?.errors?.['required']">Password is required</div>
          <div *ngIf="loginForm.get('password')?.errors?.['minlength']">Password must be at least 8 characters</div>
        </div>
      </div>

      <button
        type="submit"
        [disabled]="loginForm.invalid || submitting"
        class="submit-btn"
      >
        {{ submitting ? 'Logging in...' : 'Login' }}
      </button>
    </form>

    <p class="footer-text">
      Don't have an account? <a routerLink="/register">Register here</a>
    </p>
  </div>
</div>
```

## Frontend: Register Component

```bash
ng g c components/auth/register
```

Similar to login component, but with username field and password confirmation.

## Frontend: Auth Guard

Create `src/app/guards/auth.guard.ts`:

```typescript
import { inject } from '@angular/core';
import { Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

export const authGuard = () => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return true;
  }

  return router.createUrlTree(['/login']);
};
```

Use in routes:

```typescript
import { Routes } from '@angular/router';
import { authGuard } from './guards/auth.guard';

export const routes: Routes = [
  { path: 'login', component: LoginComponent },
  { path: 'register', component: RegisterComponent },
  {
    path: 'tasks',
    component: TaskListComponent,
    canActivate: [authGuard]
  },
  { path: '', redirectTo: '/tasks', pathMatch: 'full' }
];
```

## Security Best Practices

### 1. Never Store Passwords in Plain Text

Always hash with bcrypt:

```javascript
const hashedPassword = await bcrypt.hash(password, 10);
```

### 2. Use HTTPS in Production

HTTP sends data in plain text. HTTPS encrypts it.

### 3. Validate Input

Never trust user input. Validate on both frontend and backend.

### 4. Rate Limiting

Prevent brute force attacks:

```bash
npm install express-rate-limit
```

```javascript
import rateLimit from 'express-rate-limit';

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts
  message: 'Too many login attempts, please try again later'
});

router.post('/login', loginLimiter, async (req, res) => {
  // Login logic
});
```

### 5. Helmet for Security Headers

```bash
npm install helmet
```

```javascript
import helmet from 'helmet';

app.use(helmet());
```

### 6. Sanitize Input

Prevent NoSQL injection:

```bash
npm install express-mongo-sanitize
```

```javascript
import mongoSanitize from 'express-mongo-sanitize';

app.use(mongoSanitize());
```

### 7. Environment Variables

Never commit secrets to git:

```.gitignore
.env
node_modules/
```

### 8. Token Expiration

JWTs should expire:

```javascript
const token = jwt.sign({ userId }, JWT_SECRET, { expiresIn: '7d' });
```

### 9. Secure Cookies

For token storage in cookies:

```javascript
res.cookie('token', token, {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'strict',
  maxAge: 7 * 24 * 60 * 60 * 1000
});
```

### 10. CORS Configuration

Be specific with allowed origins:

```javascript
app.use(cors({
  origin: process.env.FRONTEND_URL,
  credentials: true
}));
```

## What We've Covered

You now know:

- How to implement user authentication
- Password hashing with bcrypt
- JWT token generation and verification
- Protecting API routes with middleware
- Implementing authentication in Angular
- Auth guards for protecting routes
- Security best practices

Your application is now secure. Users must register and log in, passwords are encrypted, and users can only access their own data.

## What's Next

In the next chapter, we'll put everything together and build the complete task manager application from start to finish. You'll create a polished, production-ready app that demonstrates all the MEAN stack concepts you've learned.

---

*"Security isn't a feature you add at the end. It's a mindset you adopt from the beginning."* — Security-Conscious Developer
