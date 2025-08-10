# Frontend Deployment Guide

## 🚀 Deployment Strategy

### Build Process
- **Development**: Vite dev server dengan hot reload
- **Staging**: Production build dengan environment variables
- **Production**: Optimized build dengan CDN dan caching

### Deployment Targets
- **Staging**: Vercel/Netlify preview deployments
- **Production**: Vercel/Netlify production deployments
- **CDN**: Cloudflare untuk static assets

## 🛠️ Build Configuration

### Vite Configuration
```ts
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          ui: ['@radix-ui/react-dialog', '@radix-ui/react-select'],
          router: ['@tanstack/react-router'],
          query: ['@tanstack/react-query'],
        },
      },
    },
    chunkSizeWarningLimit: 1000,
  },
  define: {
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
  },
});
```

### Environment Configuration
```bash
# .env.development
VITE_API_URL=http://localhost:3001
VITE_APP_NAME=SIAKAD Dev
VITE_ENVIRONMENT=development

# .env.staging
VITE_API_URL=https://api-staging.siakad-kampus.com
VITE_APP_NAME=SIAKAD Staging
VITE_ENVIRONMENT=staging

# .env.production
VITE_API_URL=https://api.siakad-kampus.com
VITE_APP_NAME=SIAKAD
VITE_ENVIRONMENT=production
```

### Package.json Scripts
```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "build:staging": "tsc && vite build --mode staging",
    "build:production": "tsc && vite build --mode production",
    "preview": "vite preview",
    "type-check": "tsc --noEmit",
    "lint": "eslint src --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "lint:fix": "eslint src --ext ts,tsx --fix"
  }
}
```

## 🌐 Vercel Deployment

### Vercel Configuration
```json
// vercel.json
{
  "buildCommand": "npm run build:production",
  "outputDirectory": "dist",
  "framework": "vite",
  "rewrites": [
    {
      "source": "/(.*)",
      "destination": "/index.html"
    }
  ],
  "headers": [
    {
      "source": "/assets/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }
      ]
    },
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "Referrer-Policy",
          "value": "strict-origin-when-cross-origin"
        }
      ]
    }
  ],
  "env": {
    "VITE_API_URL": "@vite_api_url",
    "VITE_APP_NAME": "@vite_app_name"
  }
}
```

### Environment Variables Setup
```bash
# Vercel CLI setup
npm i -g vercel

# Login to Vercel
vercel login

# Add environment variables
vercel env add VITE_API_URL
vercel env add VITE_APP_NAME

# Deploy
vercel --prod
```

### Branch Deployments
- **main branch** → Production deployment
- **staging branch** → Staging deployment
- **feature branches** → Preview deployments

## 🚀 Netlify Deployment

### Netlify Configuration
```toml
# netlify.toml
[build]
  command = "npm run build:production"
  publish = "dist"

[build.environment]
  NODE_VERSION = "18"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "/assets/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "strict-origin-when-cross-origin"
```

### Netlify CLI Setup
```bash
# Install Netlify CLI
npm install -g netlify-cli

# Login to Netlify
netlify login

# Initialize project
netlify init

# Deploy
netlify deploy --prod
```

## 🔧 Docker Deployment

### Dockerfile
```dockerfile
# Dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Install dependencies based on the preferred package manager
COPY package.json package-lock.json* ./
RUN npm ci --only=production

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Build the application
RUN npm run build:production

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/public ./public

USER nextjs

EXPOSE 3000

ENV PORT 3000

# Start the application
CMD ["npx", "serve", "-s", "dist", "-l", "3000"]
```

### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'

services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    restart: unless-stopped
    depends_on:
      - backend

  backend:
    build:
      context: ../backend
      dockerfile: Dockerfile
    ports:
      - "3001:3001"
    environment:
      - DATABASE_URL=${DATABASE_URL}
    restart: unless-stopped
```

## 📊 Performance Optimization

### Bundle Analysis
```bash
# Install bundle analyzer
npm install -D rollup-plugin-visualizer

# Add to vite config
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    react(),
    visualizer({
      filename: 'dist/stats.html',
      open: true,
    }),
  ],
});
```

### Code Splitting
```tsx
// Lazy load components
import { lazy, Suspense } from 'react';

const StudentList = lazy(() => import('./components/StudentList'));
const CourseList = lazy(() => import('./components/CourseList'));

// Route-based code splitting
const routes = [
  {
    path: '/students',
    component: lazy(() => import('./pages/Students')),
  },
  {
    path: '/courses',
    component: lazy(() => import('./pages/Courses')),
  },
];
```

### Image Optimization
```tsx
// Use optimized images
import { Image } from './components/ui/image';

// Responsive images
<Image
  src="/images/student-avatar.jpg"
  alt="Student Avatar"
  width={200}
  height={200}
  sizes="(max-width: 768px) 100vw, 200px"
  loading="lazy"
/>
```

## 🔒 Security Configuration

### Content Security Policy
```html
<!-- index.html -->
<meta http-equiv="Content-Security-Policy" content="
  default-src 'self';
  script-src 'self' 'unsafe-inline' 'unsafe-eval';
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  font-src 'self' data:;
  connect-src 'self' https://api.siakad-kampus.com;
">
```

### Security Headers
```ts
// Security middleware
app.use((req, res, next) => {
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
  res.setHeader('Permissions-Policy', 'camera=(), microphone=(), geolocation=()');
  next();
});
```

## 📈 Monitoring & Analytics

### Error Tracking
```tsx
// Error boundary with reporting
import { ErrorBoundary } from 'react-error-boundary';

const ErrorFallback = ({ error, resetErrorBoundary }) => {
  // Report error to monitoring service
  reportError(error);
  
  return (
    <div role="alert">
      <h2>Something went wrong:</h2>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
};

// Wrap app with error boundary
<ErrorBoundary FallbackComponent={ErrorFallback}>
  <App />
</ErrorBoundary>
```

### Performance Monitoring
```tsx
// Performance monitoring
import { useEffect } from 'react';

const usePerformanceMonitoring = () => {
  useEffect(() => {
    // Monitor Core Web Vitals
    if ('web-vital' in window) {
      import('web-vitals').then(({ getCLS, getFID, getFCP, getLCP, getTTFB }) => {
        getCLS(console.log);
        getFID(console.log);
        getFCP(console.log);
        getLCP(console.log);
        getTTFB(console.log);
      });
    }
  }, []);
};
```

## 🔄 CI/CD Pipeline

### GitHub Actions
```yaml
# .github/workflows/deploy.yml
name: Deploy Frontend

on:
  push:
    branches: [main, staging]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run lint
      - run: npm run test:coverage
      - run: npm run build:production

  deploy-staging:
    needs: test
    if: github.ref == 'refs/heads/staging'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run build:staging
      - uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
          vercel-args: '--prod'

  deploy-production:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run build:production
      - uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
          vercel-args: '--prod'
```

## 📋 Deployment Checklist

### Pre-Deployment
- [ ] All tests passing
- [ ] Code review completed
- [ ] Environment variables configured
- [ ] Build successful locally
- [ ] Performance budget met
- [ ] Security scan passed

### Post-Deployment
- [ ] Application accessible
- [ ] API integration working
- [ ] Authentication working
- [ ] Error monitoring active
- [ ] Performance monitoring active
- [ ] Rollback plan ready

### Monitoring
- [ ] Uptime monitoring
- [ ] Error tracking
- [ ] Performance metrics
- [ ] User analytics
- [ ] Security alerts
