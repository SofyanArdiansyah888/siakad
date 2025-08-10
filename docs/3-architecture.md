# Architecture

## Frontend
- React (TypeScript)
- Vite
- Tanstack Router
- Tanstack Table
- React Hook Form
- Zod
- React Icons
- tRPC Client
- Tailwind CSS + ShadCN UI
- TanStack Query
- JWT + Refresh Token Auth

## Backend
- NestJS (TypeScript)
- tRPC Server
- Prisma ORM
- PostgreSQL

## Monorepo Structure
apps/
  frontend/        # Next.js + tRPC Client
  backend/         # NestJS + tRPC Server + Prisma

packages/
  types/           # Shared TypeScript types (frontend & backend)
  ui/              # Shared UI components (React, Tailwind, ShadCN)
  utils/           # Shared helper functions (date formatting, validation)
  config/          # Shared config constants/env schema (Zod, dotenv)
  prisma/          # Prisma schema + migrations (generate client di packages)

backend-modules/
  auth/            # Modul Auth (JWT, refresh, RBAC)
  student/         # Modul Mahasiswa
  lecturer/        # Modul Dosen
  course/          # Modul Mata Kuliah
  krs/             # Modul KRS
  khs/             # Modul KHS
  payment/         # Modul Keuangan
  report/          # Modul Laporan Akademik
  notification/    # Modul Notifikasi (WebSocket, Email, Push)




