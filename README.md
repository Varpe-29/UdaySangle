# 🏛️ Political Office Management System — v2.0

**Enterprise-grade, production-ready web application** for managing political office operations including complaints, appointments, notices, and letters — rebuilt from the ground up with security, scalability, and modern architecture.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Security Features](#security-features)
- [Project Structure](#project-structure)
- [Local Development](#local-development)
- [Deployment](#deployment)
  - [Railway (Backend + DB)](#railway-backend--db)
  - [Vercel (Frontend)](#vercel-frontend)
- [Environment Variables](#environment-variables)
- [Default Credentials](#default-credentials)
- [API Documentation](#api-documentation)
- [What Changed from v1](#what-changed-from-v1)

---

## Overview

| Feature | Description |
|---|---|
| Complaint Management | Citizens submit complaints publicly; admins manage, update status, add notes |
| Appointment Booking | Public booking form; admin confirms/cancels with calendar view |
| Notice Board | Admins post notices with file attachments; public view available |
| Letter Management | Internal letter creation and tracking |
| Admin Dashboard | Analytics, charts, activity logs, user management |
| RBAC | Three roles: Super Admin, Main Admin, Staff Admin |

---

## Tech Stack

### Backend
- **Runtime:** Node.js 20 + TypeScript 5
- **Framework:** NestJS 10 (modular, enterprise-grade)
- **Database:** PostgreSQL 16 via Prisma ORM
- **Auth:** JWT (15min) + Refresh Token Rotation (7 days, HTTP-only cookies)
- **File Storage:** Cloudinary (replaces local disk)
- **Logging:** Winston structured logger
- **Docs:** Swagger/OpenAPI

### Frontend
- **Framework:** Next.js 15 (App Router)
- **Language:** TypeScript (strict mode)
- **Styling:** Tailwind CSS + custom design tokens
- **State:** Zustand (auth) + TanStack Query (server state)
- **Forms:** React Hook Form + Zod validation
- **Charts:** Recharts
- **Animations:** Framer Motion

### Infrastructure
- **Backend Hosting:** Railway
- **Frontend Hosting:** Vercel
- **Database:** Railway PostgreSQL
- **CI/CD:** GitHub Actions
- **Containers:** Docker + Docker Compose (local dev)

---

## Security Features

| Feature | Implementation |
|---|---|
| Password Hashing | bcrypt (12 rounds) |
| JWT Access Tokens | 15-minute expiry, signed with HS256 |
| Refresh Token Rotation | SHA-256 hashed, stored in DB, 7-day expiry |
| HTTP-Only Cookies | Refresh token never accessible via JS |
| Brute-Force Protection | Account locked after 5 failed attempts (30 min) |
| Rate Limiting | 60 req/min global; 5 login attempts/min |
| Security Headers | Helmet (CSP, HSTS, X-Frame-Options, etc.) |
| CORS | Strict whitelist — only frontend URL allowed |
| Input Validation | DTO validation on all endpoints (class-validator) |
| XSS Sanitization | HTML entity encoding on all string inputs |
| SQL Injection | Prisma parameterized queries — no raw SQL |
| Soft Deletes | Records never physically deleted (audit trail) |
| RBAC | Role-based guards on every protected endpoint |
| Audit Logging | Every mutation logged with user + IP |
| Session Tracking | Active sessions table with device info |
| Secure File Uploads | Type whitelist, size limits, Cloudinary only |

---

## Project Structure

```
production-app/
├── backend/
│   ├── prisma/
│   │   ├── schema.prisma        # Full DB schema
│   │   └── seed.ts              # Initial admin users
│   ├── src/
│   │   ├── main.ts              # Bootstrap with security
│   │   ├── app.module.ts        # Root module
│   │   ├── prisma/              # DB service
│   │   ├── modules/
│   │   │   ├── auth/            # JWT + refresh token auth
│   │   │   ├── users/           # User service
│   │   │   ├── complaints/      # Complaint CRUD + public form
│   │   │   ├── appointments/    # Appointment CRUD + public form
│   │   │   ├── notices/         # Notice board
│   │   │   ├── letters/         # Letter management
│   │   │   ├── admin/           # Analytics + user management
│   │   │   └── health/          # Health check endpoint
│   │   ├── common/
│   │   │   ├── guards/          # JWT + Roles guards
│   │   │   ├── interceptors/    # Transform + Logging
│   │   │   ├── filters/         # Global exception filter
│   │   │   ├── middleware/      # Security + Request logger
│   │   │   └── decorators/      # @CurrentUser, @Roles, @Public
│   │   └── config/              # Env validation, Cloudinary, Multer, Winston
│   └── Dockerfile
│
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── layout.tsx       # Root layout
│   │   │   ├── page.tsx         # Redirects to /dashboard
│   │   │   ├── login/           # Login page
│   │   │   ├── dashboard/       # Analytics dashboard
│   │   │   ├── complaints/      # Complaints management
│   │   │   ├── appointments/    # Appointments management
│   │   │   ├── notices/         # Notice board
│   │   │   ├── letters/         # Letters
│   │   │   └── admin/           # Admin panel (user mgmt + logs)
│   │   ├── components/
│   │   │   └── providers.tsx    # QueryClient provider
│   │   ├── lib/
│   │   │   └── api.ts           # Axios + auto token refresh
│   │   ├── services/            # API service functions
│   │   ├── store/
│   │   │   └── auth.store.ts    # Zustand auth store
│   │   ├── validations/         # Zod schemas
│   │   ├── middleware.ts         # Next.js route protection
│   │   └── styles/
│   │       └── globals.css      # CSS variables, dark mode
│   └── Dockerfile
│
├── .github/workflows/
│   └── ci-cd.yml                # GitHub Actions CI/CD
├── docker-compose.yml           # Local dev environment
└── README.md
```

---

## Local Development

### Prerequisites
- Node.js 20+
- Docker & Docker Compose
- A Cloudinary account (free tier works)

### Quick Start (Docker)

```bash
# 1. Clone the repo
git clone https://github.com/YOUR_USERNAME/office-manage.git
cd office-manage

# 2. Set Cloudinary credentials
cp backend/.env.example backend/.env
# Edit backend/.env with your Cloudinary keys

# 3. Start everything
docker-compose up --build

# App will be available at:
# Frontend: http://localhost:3000
# Backend:  http://localhost:5000
# Swagger:  http://localhost:5000/api/docs
```

### Manual Setup (without Docker)

```bash
# Start PostgreSQL locally (or use Docker just for DB)
docker run -d --name pg -e POSTGRES_PASSWORD=postgres -p 5432:5432 postgres:16-alpine

# Backend
cd backend
cp .env.example .env       # Fill in all values
npm install
npx prisma migrate dev --name init
npx prisma db seed
npm run start:dev

# Frontend (new terminal)
cd frontend
cp .env.example .env.local  # Fill in API URL
npm install
npm run dev
```

---

## Deployment

### Railway (Backend + DB)

1. **Create Railway account** → [railway.app](https://railway.app)

2. **New Project → Deploy from GitHub** → select your repo → select `backend` folder

3. **Add PostgreSQL** → Railway dashboard → `+ New` → `Database` → `PostgreSQL`

4. **Link DB to backend service** → Railway auto-injects `DATABASE_URL`

5. **Add environment variables** in Railway dashboard:
   ```
   JWT_SECRET=<generate: openssl rand -hex 64>
   JWT_REFRESH_SECRET=<generate: openssl rand -hex 64>
   COOKIE_SECRET=<generate: openssl rand -hex 32>
   FRONTEND_URL=https://your-app.vercel.app
   CLOUDINARY_CLOUD_NAME=your_name
   CLOUDINARY_API_KEY=your_key
   CLOUDINARY_API_SECRET=your_secret
   NODE_ENV=production
   ENABLE_SWAGGER=false
   ```

6. **Deploy** → Railway will run: `npm install && npx prisma generate && npm run build`
   Then start: `npx prisma migrate deploy && node dist/main`

7. **Seed the database** (one time):
   ```bash
   # In Railway shell or via railway CLI:
   cd backend && npx prisma db seed
   ```

### Vercel (Frontend)

1. **Create Vercel account** → [vercel.com](https://vercel.com)

2. **Import repository** → Set Root Directory to `frontend`

3. **Add environment variables**:
   ```
   NEXT_PUBLIC_API_URL=https://your-backend.railway.app/api/v1
   NEXT_PUBLIC_APP_NAME=Office Management System
   ```

4. **Deploy** → Vercel builds and deploys automatically

5. **Update CORS** → Add your Vercel URL to Railway's `FRONTEND_URL` env var

### GitHub Actions Setup

Add these secrets to your GitHub repo (`Settings → Secrets → Actions`):

```
DATABASE_URL          → Your Railway PostgreSQL URL
JWT_SECRET            → Same as Railway
JWT_REFRESH_SECRET    → Same as Railway
COOKIE_SECRET         → Same as Railway
NEXT_PUBLIC_API_URL   → Your Railway backend URL
RAILWAY_TOKEN         → Railway CLI token (railway.app/account/tokens)
VERCEL_TOKEN          → Vercel CLI token
VERCEL_ORG_ID         → From Vercel project settings
VERCEL_PROJECT_ID     → From Vercel project settings
```

---

## Environment Variables

### Backend (`backend/.env`)

| Variable | Required | Description |
|---|---|---|
| `DATABASE_URL` | ✅ | PostgreSQL connection string |
| `JWT_SECRET` | ✅ | Min 64 chars random string |
| `JWT_REFRESH_SECRET` | ✅ | Min 64 chars random string |
| `COOKIE_SECRET` | ✅ | Min 32 chars random string |
| `FRONTEND_URL` | ✅ | Frontend URL for CORS |
| `CLOUDINARY_CLOUD_NAME` | ✅ | Cloudinary cloud name |
| `CLOUDINARY_API_KEY` | ✅ | Cloudinary API key |
| `CLOUDINARY_API_SECRET` | ✅ | Cloudinary API secret |
| `PORT` | ❌ | Default: 5000 |
| `NODE_ENV` | ❌ | Default: development |
| `ENABLE_SWAGGER` | ❌ | Show API docs (disable in prod) |

### Frontend (`frontend/.env.local`)

| Variable | Required | Description |
|---|---|---|
| `NEXT_PUBLIC_API_URL` | ✅ | Backend API base URL |
| `NEXT_PUBLIC_APP_NAME` | ❌ | App display name |

---

## Default Credentials

> ⚠️ **Change these immediately after first login!**

| Role | Username | Default Password |
|---|---|---|
| Super Admin | `superadmin` | `SuperAdmin@2024!` |
| Main Admin | `admin` | `MainAdmin@2024!` |
| Staff Admin | `staff` | `Staff@2024!` |

### Role Permissions

| Action | Staff | Main Admin | Super Admin |
|---|---|---|---|
| View complaints/appointments | ✅ | ✅ | ✅ |
| Add complaint notes | ✅ | ✅ | ✅ |
| Post notices & letters | ✅ | ✅ | ✅ |
| Update complaint/appt status | ❌ | ✅ | ✅ |
| Delete records | ❌ | ✅ | ✅ |
| View admin panel | ❌ | ✅ | ✅ |
| Lock/unlock users | ❌ | ✅ | ✅ |
| Create/delete users | ❌ | ❌ | ✅ |

---

## API Documentation

When `ENABLE_SWAGGER=true`, visit: `http://localhost:5000/api/docs`

### Key Endpoints

```
POST   /api/v1/auth/login          # Login → returns access token + sets cookie
POST   /api/v1/auth/refresh        # Refresh access token via cookie
POST   /api/v1/auth/logout         # Logout current session
DELETE /api/v1/auth/logout-all     # Logout all devices
GET    /api/v1/auth/me             # Current user profile

GET    /api/v1/complaints          # All complaints (admin, paginated)
POST   /api/v1/complaints          # Submit complaint (PUBLIC, multipart)
GET    /api/v1/complaints/public   # Public approved complaints
PATCH  /api/v1/complaints/:id/status  # Update status (main admin)
POST   /api/v1/complaints/:id/entries # Add note
DELETE /api/v1/complaints/:id      # Soft delete (main admin)

GET    /api/v1/appointments        # All appointments (admin, paginated)
POST   /api/v1/appointments        # Book appointment (PUBLIC)
PATCH  /api/v1/appointments/:id/status  # Update status

GET    /api/v1/notices             # All notices (public)
POST   /api/v1/notices             # Post notice (admin, multipart)
DELETE /api/v1/notices/:id         # Delete (main admin)

GET    /api/v1/letters             # All letters (admin)
POST   /api/v1/letters             # Create letter (admin, multipart)

GET    /api/v1/admin/analytics     # Dashboard analytics
GET    /api/v1/admin/users         # User list
POST   /api/v1/admin/users         # Create user (super admin)
GET    /api/v1/admin/activity-logs # Activity logs

GET    /health                     # Health check (public)
```

---

## What Changed from v1

| Issue | v1 (Old) | v2 (New) |
|---|---|---|
| Auth | Hardcoded credentials in frontend JS | JWT + bcrypt + DB auth |
| Password storage | **Plain text in DB** | bcrypt hash (12 rounds) |
| Sessions | localStorage JWT (no expiry) | HTTP-only cookie refresh token |
| Brute force | No protection | Lock after 5 attempts, 30 min |
| Rate limiting | None | 60 req/min global, 5 login/min |
| Database | MySQL + raw queries | PostgreSQL + Prisma ORM |
| File storage | Local disk (lost on redeploy) | Cloudinary CDN |
| Framework | Express.js (bare) | NestJS (modular, typed) |
| Frontend | Vite + React (CSR only) | Next.js 15 App Router (SSR) |
| RBAC | Simple `role === 'admin'` check | Guards + decorators system |
| Audit trail | None | Full activity + audit logs |
| Error handling | Console.log | Global exception filter + Winston |
| API docs | None | Swagger/OpenAPI |
| Deployment | Manual | GitHub Actions CI/CD |

---

## License

Private — All rights reserved. Unauthorized use prohibited.
