# Nuuma — Lingerie e-commerce

Full-stack demo: **Next.js (App Router)** + **Express** + **PostgreSQL (Prisma)**. Cash on delivery only; cart persisted in `localStorage`.

## Project layout

- `client/` — Next.js 14, React, Tailwind CSS, Axios
- `server/` — Express REST API, JWT auth, bcrypt, admin-protected product mutations

## Prerequisites

- Node.js 18+
- PostgreSQL 14+ (local or hosted; connection string as `DATABASE_URL`)

## 1. Backend setup

```bash
cd server
cp .env.example .env
# Edit .env: DATABASE_URL, JWT_SECRET, CLIENT_ORIGIN
npm install
npx prisma migrate deploy
# Or during development: npx prisma migrate dev
# Optional: npm run seed (connects only; add products in Admin)
npm run create-admin
npm run dev
```

The API listens on **http://localhost:5000** by default.

- Health check: `GET http://localhost:5000/api/health`
- Default admin (if you used `create-admin` without env overrides): `admin@nuuma.local` / `admin123456` — change in production.

### API summary

| Method | Path | Auth |
|--------|------|------|
| POST | `/api/auth/register` | — |
| POST | `/api/auth/login` | — |
| GET | `/api/products` | — (query: category, minPrice, maxPrice, size, search, limit) |
| GET | `/api/products/:id` | — (UUID) |
| POST | `/api/products` | Admin JWT |
| PUT | `/api/products/:id` | Admin JWT |
| DELETE | `/api/products/:id` | Admin JWT |
| POST | `/api/orders` | — (checkout body) |
| GET | `/api/orders` | Admin JWT |
| PUT | `/api/orders/:id/status` | Admin JWT |

Admin requests: `Authorization: Bearer <token>` from login.

## 2. Frontend setup

```bash
cd client
cp .env.local.example .env.local
# Ensure NEXT_PUBLIC_API_URL=http://localhost:5000 (or your API URL)
npm install
npm run dev
```

Open **http://localhost:3000**.

## 3. Connect frontend ↔ backend

- `client/.env.local` sets `NEXT_PUBLIC_API_URL` to the Express base URL (no trailing slash).
- `server/.env` sets `CLIENT_ORIGIN` to the Next.js origin for CORS (default `http://localhost:3000`).

Restart both processes after changing env files.

## Google Cloud Storage (optional uploads)

By default, product images are stored under `server/uploads` and served from `/uploads` on the API. To use **Google Cloud Storage** instead:

1. Create a GCS bucket in the same Google Cloud project as a service account that can write objects (for example `roles/storage.objectAdmin` on that bucket).
2. Allow **public read** for storefront URLs: with uniform bucket-level access, add bucket IAM so anonymous clients can read objects (for example principal `allUsers` with role **Storage Object Viewer**), or use another pattern your security team approves. Without public read, image URLs returned by the API will return 403 in the browser.
3. In `server/.env`, set `GCS_BUCKET_NAME` and authentication using either `GCS_KEY_FILE` (absolute path to the service account JSON key) or `GOOGLE_APPLICATION_CREDENTIALS` with the same path. Optional: `GCS_UPLOAD_PREFIX`, `GCS_PUBLIC_BASE_URL` (CDN or custom domain base URL, no trailing slash), `GCS_PROJECT_ID`.
4. Restart the API. The Next.js app must allow the image host: `client/next.config.mjs` includes `storage.googleapis.com` for the default public URL shape. If you set `GCS_PUBLIC_BASE_URL` to a CDN or custom domain, set **`NEXT_PUBLIC_GCS_IMAGE_HOST`** in `client/.env.local` to that hostname (see `client/.env.local.example`) so `next/image` allows it.

Do not commit service account JSON files; keep keys out of git (see `.gitignore` patterns).

## Production notes

- Use strong `JWT_SECRET` and HTTPS.
- Restrict CORS `CLIENT_ORIGIN` to your real domain.
- Run `npm run build && npm start` in `client/`; use `npm start` in `server/` with `NODE_ENV=production`.
- Run `npx prisma migrate deploy` against your production database before starting the API.
