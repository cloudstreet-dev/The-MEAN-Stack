# Chapter 11: Deployment and Going Live

## From Localhost to the Internet

Your application works beautifully on your computer. Now it's time to share it with the world. Deployment is the process of making your application accessible on the internet, and it involves several important considerations:

- Choosing hosting providers
- Configuring production environments
- Setting up continuous deployment
- Monitoring and logging
- Performance optimization
- Security hardening

This chapter walks you through deploying your MEAN stack application step by step.

## Deployment Options

### Backend Hosting Options

**1. Render (Recommended for Beginners)**
- Free tier available
- Easy deployment from GitHub
- Automatic HTTPS
- Good documentation
- [render.com](https://render.com)

**2. Railway**
- Simple deployment
- Good free tier
- Built-in databases
- [railway.app](https://railway.app)

**3. Heroku**
- Classic PaaS
- Simple deployment
- Free tier removed (paid plans)
- [heroku.com](https://heroku.com)

**4. DigitalOcean App Platform**
- Simple deployment
- Good performance
- Reasonable pricing
- [digitalocean.com](https://www.digitalocean.com/products/app-platform)

**5. AWS/GCP/Azure**
- Enterprise-grade
- Complex but powerful
- Requires more expertise
- Most expensive

### Frontend Hosting Options

**1. Vercel (Recommended)**
- Excellent for Angular
- Free tier
- Automatic deployments
- Global CDN
- [vercel.com](https://vercel.com)

**2. Netlify**
- Similar to Vercel
- Great free tier
- Good documentation
- [netlify.com](https://netlify.com)

**3. Render (Static Sites)**
- Can host both frontend and backend
- Free tier for static sites
- [render.com](https://render.com)

**4. Firebase Hosting**
- Google's solution
- Fast global CDN
- Good integration with Firebase services
- [firebase.google.com](https://firebase.google.com)

### Database Hosting Options

**1. MongoDB Atlas (Recommended)**
- Official MongoDB cloud service
- Free M0 tier (512MB)
- Easy to use
- Global distribution
- [mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas)

**2. Railway**
- Built-in MongoDB
- Simple setup
- [railway.app](https://railway.app)

**3. DigitalOcean Managed MongoDB**
- Dedicated database
- More expensive
- [digitalocean.com](https://www.digitalocean.com/products/managed-databases-mongodb)

## Deploying the Backend

### Step 1: Prepare Your Backend for Production

#### Production Dependencies

Ensure these are in `dependencies`, not `devDependencies`:

```bash
npm install --save express mongoose cors dotenv jsonwebtoken bcrypt helmet express-rate-limit express-mongo-sanitize
```

#### Create Production Configuration

Update `src/app.js`:

```javascript
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';
import mongoSanitize from 'express-mongo-sanitize';
import 'dotenv/config';

import taskRoutes from './routes/tasks.js';
import authRoutes from './routes/auth.js';

const app = express();

// Security middleware
app.use(helmet());
app.use(mongoSanitize());

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100
});
app.use('/api/', limiter);

// CORS configuration
const allowedOrigins = process.env.FRONTEND_URL?.split(',') || ['http://localhost:4200'];

app.use(cors({
  origin: (origin, callback) => {
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true
}));

// Body parsing
app.use(express.json());

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

// Routes
app.use('/api/tasks', taskRoutes);
app.use('/api/auth', authRoutes);

// 404 handler
app.use((req, res) => {
  res.status(404).json({ error: 'Not found' });
});

// Error handler
app.use((err, req, res, next) => {
  console.error(err.stack);

  if (process.env.NODE_ENV === 'production') {
    res.status(500).json({ error: 'Internal server error' });
  } else {
    res.status(500).json({ error: err.message, stack: err.stack });
  }
});

export default app;
```

#### Create Start Script

Update `src/server.js`:

```javascript
import app from './app.js';
import connectDatabase from './database.js';

const PORT = process.env.PORT || 3000;

async function start() {
  try {
    await connectDatabase();
    app.listen(PORT, () => {
      console.log(`Server running on port ${PORT}`);
      console.log(`Environment: ${process.env.NODE_ENV}`);
    });
  } catch (error) {
    console.error('Failed to start server:', error);
    process.exit(1);
  }
}

start();
```

### Step 2: Deploy to Render

#### Create `render.yaml`:

```yaml
services:
  - type: web
    name: task-manager-api
    env: node
    plan: free
    buildCommand: npm install
    startCommand: npm start
    envVars:
      - key: NODE_ENV
        value: production
      - key: MONGODB_URL
        sync: false
      - key: JWT_SECRET
        sync: false
      - key: FRONTEND_URL
        sync: false
```

#### Deploy Steps:

1. Push code to GitHub
2. Go to [render.com](https://render.com)
3. Sign up/log in
4. Click "New +" → "Web Service"
5. Connect your GitHub repository
6. Configure:
   - **Name:** task-manager-api
   - **Environment:** Node
   - **Build Command:** `npm install`
   - **Start Command:** `npm start`
7. Add environment variables:
   - `NODE_ENV=production`
   - `MONGODB_URL=your-mongodb-atlas-url`
   - `JWT_SECRET=your-secret-key`
   - `FRONTEND_URL=https://your-frontend-url.vercel.app`
8. Click "Create Web Service"

Render will deploy your backend automatically! You'll get a URL like `https://task-manager-api.onrender.com`.

**Important:** Free tier services on Render spin down after inactivity. First request may be slow.

## Deploying the Frontend

### Step 1: Prepare Angular for Production

#### Update Environment Files

`src/environments/environment.ts`:

```typescript
export const environment = {
  production: true,
  apiUrl: 'https://task-manager-api.onrender.com/api'
};
```

`src/environments/environment.development.ts`:

```typescript
export const environment = {
  production: false,
  apiUrl: 'http://localhost:3000/api'
};
```

#### Build for Production

```bash
ng build --configuration=production
```

This creates optimized files in `dist/` directory.

### Step 2: Deploy to Vercel

#### Install Vercel CLI (Optional):

```bash
npm install -g vercel
```

#### Deploy via Web Interface:

1. Go to [vercel.com](https://vercel.com)
2. Sign up/log in with GitHub
3. Click "Add New" → "Project"
4. Import your repository
5. Configure:
   - **Framework Preset:** Angular
   - **Root Directory:** ./
   - **Build Command:** `ng build --configuration=production`
   - **Output Directory:** `dist/task-manager-frontend/browser`
6. Add environment variable:
   - Key: `API_URL`
   - Value: `https://task-manager-api.onrender.com/api`
7. Click "Deploy"

Vercel will build and deploy automatically! You'll get a URL like `https://task-manager.vercel.app`.

#### Deploy via CLI:

```bash
cd task-manager-frontend
vercel
```

Follow the prompts. Vercel will deploy your app.

### Step 3: Update Backend CORS

Update your backend's `FRONTEND_URL` environment variable on Render:

```
FRONTEND_URL=https://task-manager.vercel.app
```

## Setting Up MongoDB Atlas

### Create Database:

1. Go to [mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas)
2. Sign up/log in
3. Create a free M0 cluster
4. Choose a cloud provider and region (closest to your backend)
5. Name your cluster
6. Create cluster (takes a few minutes)

### Configure Database:

1. **Database Access:**
   - Go to "Database Access"
   - Add a database user
   - Username: `taskmanager`
   - Password: Generate secure password
   - Role: Read and write to any database

2. **Network Access:**
   - Go to "Network Access"
   - Add IP Address
   - Add `0.0.0.0/0` (allow from anywhere)
   - Note: For production, restrict to your server's IP

3. **Get Connection String:**
   - Go to "Clusters"
   - Click "Connect"
   - Choose "Connect your application"
   - Copy the connection string:
     ```
     mongodb+srv://taskmanager:<password>@cluster0.xxxxx.mongodb.net/taskmanager?retryWrites=true&w=majority
     ```
   - Replace `<password>` with your user password

4. **Add to Render:**
   - Go to your Render service
   - Environment → Add environment variable
   - `MONGODB_URL=your-connection-string`

## Continuous Deployment

### Automatic Deployments

Both Render and Vercel support automatic deployments:

**Render:**
- Automatically deploys when you push to your main branch
- Can configure different branches for staging/production

**Vercel:**
- Automatically deploys every push
- Creates preview URLs for pull requests
- Production deployment from main branch

### Deployment Workflow:

```bash
# 1. Make changes
git add .
git commit -m "Add new feature"

# 2. Push to GitHub
git push origin main

# 3. Render and Vercel automatically deploy
```

### Preview Environments

Create preview environments for testing:

1. Create a branch:
   ```bash
   git checkout -b feature/new-feature
   ```

2. Push branch:
   ```bash
   git push origin feature/new-feature
   ```

3. Vercel creates a preview URL automatically

4. Merge when ready:
   ```bash
   git checkout main
   git merge feature/new-feature
   git push origin main
   ```

## Monitoring and Logging

### Backend Logging

Install Winston for production logging:

```bash
npm install winston
```

Create `src/utils/logger.js`:

```javascript
import winston from 'winston';

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
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

export default logger;
```

Use in your app:

```javascript
import logger from './utils/logger.js';

logger.info('User logged in', { userId: user._id });
logger.error('Failed to create task', { error: error.message });
```

### Error Tracking

Use a service like Sentry for error tracking:

```bash
npm install @sentry/node
```

Configure in `src/app.js`:

```javascript
import * as Sentry from '@sentry/node';

if (process.env.NODE_ENV === 'production') {
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV
  });
}

// Error handler
app.use((err, req, res, next) => {
  if (process.env.NODE_ENV === 'production') {
    Sentry.captureException(err);
  }
  // Rest of error handling
});
```

### Uptime Monitoring

Use a free uptime monitoring service:

**UptimeRobot:**
1. Go to [uptimerobot.com](https://uptimerobot.com)
2. Add new monitor
3. Type: HTTP(s)
4. URL: `https://your-api.onrender.com/health`
5. Monitoring interval: 5 minutes
6. Get email alerts when down

## Performance Optimization

### Backend Optimization

**1. Enable Gzip Compression:**

```bash
npm install compression
```

```javascript
import compression from 'compression';

app.use(compression());
```

**2. Add Caching Headers:**

```javascript
app.use((req, res, next) => {
  res.set('Cache-Control', 'public, max-age=300'); // 5 minutes
  next();
});
```

**3. Database Indexing:**

Ensure your Mongoose schemas have indexes:

```javascript
taskSchema.index({ userId: 1, createdAt: -1 });
taskSchema.index({ userId: 1, completed: 1 });
```

**4. Connection Pooling:**

Mongoose handles this automatically, but you can configure:

```javascript
mongoose.connect(MONGODB_URL, {
  maxPoolSize: 10,
  minPoolSize: 5
});
```

### Frontend Optimization

**1. Lazy Loading:**

```typescript
// In app.routes.ts
export const routes: Routes = [
  {
    path: 'tasks',
    loadComponent: () => import('./components/tasks/task-list/task-list.component')
      .then(m => m.TaskListComponent),
    canActivate: [authGuard]
  }
];
```

**2. Image Optimization:**

- Use WebP format
- Compress images
- Use CDN for assets

**3. Bundle Analysis:**

```bash
ng build --stats-json
npx webpack-bundle-analyzer dist/task-manager-frontend/stats.json
```

**4. Service Worker (PWA):**

```bash
ng add @angular/pwa
```

This makes your app work offline and load faster.

## Security Checklist

- [ ] HTTPS enabled (Render and Vercel provide this)
- [ ] Environment variables set correctly
- [ ] JWT secret is strong and random
- [ ] MongoDB Atlas has authentication
- [ ] CORS configured properly
- [ ] Rate limiting enabled
- [ ] Input validation on backend
- [ ] Helmet.js configured
- [ ] No secrets in code (use environment variables)
- [ ] `.env` in `.gitignore`
- [ ] Updated dependencies (no vulnerabilities)

Check for vulnerabilities:

```bash
npm audit
npm audit fix
```

## Custom Domain (Optional)

### Backend Domain:

**On Render:**
1. Go to your service settings
2. "Custom Domain" section
3. Add your domain (e.g., `api.yourdomain.com`)
4. Update your domain's DNS:
   - Type: CNAME
   - Name: api
   - Value: your-app.onrender.com

### Frontend Domain:

**On Vercel:**
1. Go to project settings
2. "Domains" section
3. Add your domain (e.g., `yourdomain.com`)
4. Update DNS as instructed

## Backup Strategy

### Database Backups:

**MongoDB Atlas:**
- Automatic backups on paid tiers
- Free tier: Use `mongodump`:

```bash
mongodump --uri="your-mongodb-atlas-connection-string" --out=./backup
```

Schedule backups with cron or GitHub Actions.

### Code Backups:

Your code is on GitHub—that's your backup. But:
- Tag releases: `git tag v1.0.0`
- Use branches for versions
- Don't delete old branches immediately

## Troubleshooting Common Issues

### Backend Not Starting:

1. Check logs on Render
2. Verify environment variables
3. Test MongoDB connection
4. Check port configuration (use `process.env.PORT`)

### CORS Errors:

1. Verify `FRONTEND_URL` environment variable
2. Check if frontend URL is exact (with https://)
3. Include credentials in requests

### 502 Bad Gateway:

1. Backend might be crashed
2. Check logs
3. Verify all dependencies installed
4. Check for syntax errors

### Database Connection Issues:

1. Verify MongoDB Atlas IP whitelist
2. Check username/password in connection string
3. Ensure network access is configured
4. Test connection string locally first

## What We've Covered

You now know:

- How to choose hosting providers
- How to prepare apps for production
- How to deploy backend (Render)
- How to deploy frontend (Vercel)
- How to set up MongoDB Atlas
- How to configure continuous deployment
- How to monitor and log
- How to optimize performance
- Security best practices
- Troubleshooting common issues

Your MEAN stack application is now live and accessible to anyone on the internet!

## What's Next

You've built and deployed a full-stack application. But your journey doesn't end here. In the final chapter, we'll explore what comes next: advanced topics, alternative technologies, career paths, and resources for continued learning. The MEAN stack is just the beginning.

---

*"Deployment is not the end of development. It's the beginning of real-world feedback, iteration, and improvement."* — Experienced Developer
