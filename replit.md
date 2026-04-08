# Workspace

## Overview

pnpm workspace monorepo using TypeScript. Each package manages its own dependencies.

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **API framework**: Express 5
- **Database**: PostgreSQL + Drizzle ORM
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild (CJS bundle)

## Key Commands

- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- `pnpm --filter @workspace/api-server run dev` — run API server locally

See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details.

## Project: Hajj Permit QR Verification System

### Purpose
A system for verifying Hajj pilgrimage permits via QR codes. Matches the official hch.sa design with RTL/LTR multi-language support.

### Pages
- `/verify?id={permitId}` — Public QR verification page (Arabic/English, permit card, personal info card, Makkah watermark)
- `/license` — License page with DGA "Registered on" badge and grey footer
- `/admin` — Admin portal (requires login, redirects to `/admin/login`)
- `/admin/login` — Password-only login form

### Admin Portal Tabs
1. **Records** — Search/filter users, approve/reject status, QR code display, edit (slide-out panel), delete
2. **Add User** — Form to create a new permit holder including photo upload
3. **License Details** — Manage license numbers and expiry dates

### API Routes (`artifacts/api-server` — port 8080)
**Public:**
- `GET /api/healthz`
- `GET /api/user/:permitId` — returns 404 (Invalid Permit) or 403 (Permit Disabled) or user data
- `GET /api/generate-qr/:permitId`
- `GET /api/licenses`

**Admin (require session cookie):**
- `POST /api/admin/login`
- `POST /api/admin/logout`
- `GET /api/admin/session`
- `GET /api/admin/users?search=&page=&limit=`
- `POST /api/admin/users` — create new user (photo_url accepted as base64 data URL)
- `PUT /api/admin/users/:id` — full record update
- `PATCH /api/admin/users/:id/status` — toggle active/inactive
- `DELETE /api/admin/users/:id`
- `GET /api/admin/licenses`
- `POST /api/admin/licenses`
- `PUT /api/admin/licenses/:id`
- `DELETE /api/admin/licenses/:id`

### Authentication
- Password-only via `ADMIN_PASSWORD` + `ADMIN_USERNAME` env secrets
- Sessions stored in PostgreSQL `session` table, 7-day expiry
- `adminFetch()` in admin.tsx sends `credentials: "include"` for all admin API calls

### Database Schema (`lib/db`)
- `users` table: id, name, id_number, nationality, gender, date_of_birth, photo_url (text, stores base64 data URLs), permit_id (unique), permit_type, authority, service_provider, company_id, service_group_makkah, blood_type, status (active/inactive)
- `licenses` table: id, license_number, expiry_date
- `session` table: for express-session (PostgreSQL store)

### Design Notes
- **Font**: `'Tajawal', 'IBM Plex Sans Arabic', 'Noto Sans Arabic', sans-serif` — DO NOT change without explicit user approval
- **DGA "Registered on" box**: ONLY in `license.tsx` footer — never on verify.tsx
- **Viewport**: `initial-scale=1.2`
- **API body limit**: 10mb (for base64 photo uploads)
- **Test permit ID**: `446787342487223`
- **Default license number**: `20250519725`, expiry `19-05-2026`

### Photo Upload
- `PhotoUpload` component in admin.tsx resizes images to max 400×400 via canvas
- Stores as JPEG base64 data URL in `photo_url` field
- Has `img.onerror`, `reader.onerror`, and try/catch for robust error handling
