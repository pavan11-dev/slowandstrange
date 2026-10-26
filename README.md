# Slow & Strange Shop

E-commerce website for the @slowandstrange Instagram shop, which sells live isopods, springtails and tarantulas. Customers browse the catalog, add animals to a cart, log in, and pay online. The shop owner manages products and orders.

> Status: in development. See [Project status](#project-status).

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | React (Vite), Redux Toolkit + RTK Query, React Router, Tailwind CSS, React Hook Form + Zod |
| Backend | Node.js, Express, Mongoose, Zod |
| Database | MongoDB Atlas |
| Images | Cloudinary |
| Auth | JWT access token + rotating refresh token in httpOnly cookies |
| Payments | <Razorpay or Stripe, decided in SS-53> |
| Hosting | Frontend: <Vercel/Netlify>. Backend: <Render/Railway> |

## Folder map

```
slowandstrange/
  client/     React app (Vite)
  server/     Express API
  docs/       decisions.md and other project notes
  .github/    CI workflows
```

Inside each app:

- `server/src/`: `config`, `models`, `routes`, `controllers`, `services`, `middleware`, `validators`, `jobs`, `utils`
- `client/src/`: `app`, `api`, `features`, `pages`, `layouts`, `components`, `routes`, `hooks`, `utils`

## Prerequisites

- Node.js 20 LTS or newer
- npm
- A MongoDB Atlas cluster (free tier is fine for development)
- A Cloudinary account
- Payment gateway test-mode keys (once payments are built)

## Getting started

### 1. Clone and install

```bash
git clone <repo-url>
cd slowandstrange

cd server && npm install
cd ../client && npm install
```

### 2. Configure environment variables

```bash
cp server/.env.example server/.env
cp client/.env.example client/.env
```

Fill in the values. Never commit `.env` files.

**server/.env**

| Variable | Description |
|---|---|
| `NODE_ENV` | `development` or `production` |
| `PORT` | API port (default `5000`) |
| `MONGO_URI` | MongoDB Atlas connection string |
| `JWT_ACCESS_SECRET` | Random string, at least 32 characters |
| `JWT_REFRESH_SECRET` | A different random string, at least 32 characters |
| `CLIENT_URL` | Frontend origin, for CORS (`http://localhost:5173` locally) |
| `CLOUDINARY_CLOUD_NAME` | From the Cloudinary dashboard |
| `CLOUDINARY_API_KEY` | From the Cloudinary dashboard |
| `CLOUDINARY_API_SECRET` | From the Cloudinary dashboard |

More variables (payment gateway keys, webhook secret, email provider) are added by the payment and email tickets. Keep `.env.example` up to date.

**client/.env**

| Variable | Description |
|---|---|
| `VITE_API_URL` | API base URL (`http://localhost:5000/api` locally) |

### 3. Seed the database

```bash
cd server
npm run seed
```

This creates categories and sample products. It is safe to run more than once.

### 4. Run the apps

In two terminals:

```bash
# Terminal 1: API on http://localhost:5000
cd server && npm run dev

# Terminal 2: frontend on http://localhost:5173
cd client && npm run dev
```

Check that the API is up at `http://localhost:5000/api/health`.

## Scripts

**server/**

| Command | What it does |
|---|---|
| `npm run dev` | Start the API with auto-reload (nodemon) |
| `npm start` | Start the API (production) |
| `npm run seed` | Seed categories and sample products |
| `npm test` | Run API tests |
| `npm run lint` | Lint the code |

**client/**

| Command | What it does |
|---|---|
| `npm run dev` | Start the Vite dev server |
| `npm run build` | Production build into `dist/` |
| `npm run preview` | Preview the production build |
| `npm test` | Run frontend tests |
| `npm run lint` | Lint the code |

## Main user flow

```
Home > Shop > Product > Cart > Checkout
                                  |
                  logged in? -- yes --> Payment
                      |
                      no --> Login --> back to Checkout
```

## API overview

Base path: `/api`

| Area | Endpoints |
|---|---|
| Auth | `POST /auth/register`, `POST /auth/login`, `POST /auth/refresh`, `POST /auth/logout`, `GET /auth/me` |
| Catalog | `GET /products`, `GET /products/:slug`, `GET /products/:slug/related`, `GET /categories` |
| Orders | `POST /orders`, `GET /orders/mine`, `GET /orders/:id`, `POST /orders/:id/retry-payment` |
| Webhooks | `POST /webhooks/payment` |
| Contact | `POST /contact` |

Admin endpoints live under `/api/admin/*` and require an admin account.

## Key design decisions

- Money is stored as integers in the smallest unit (paise or cents), never as floats.
- The server recomputes all prices and totals. The client is never trusted.
- Stock is reserved atomically for 15 minutes at checkout, so two customers cannot buy the same one-of-a-kind animal.
- Only the payment webhook marks an order as paid.
- Auth tokens live in httpOnly cookies, never in localStorage.

More detail and the reasoning are in [docs/decisions.md](docs/decisions.md).

## Deployment

The frontend and backend deploy as two separate services from this one repository.

| Service | Root directory | Build | Start |
|---|---|---|---|
| Frontend | `client` | `npm run build` (output `dist`) | static hosting |
| Backend | `server` | `npm ci` | `npm start` |

Use a shared parent domain (for example `www.example.com` and `api.example.com`) so cookies work. Register the payment webhook URL in the gateway dashboard after the backend is live.

## Project status

Updated as tickets complete.

- [ ] Epic 1: Project setup (SS-1 to SS-5)
- [ ] Epic 2: Backend foundation (SS-6 to SS-10)
- [ ] Epic 3: Authentication (SS-11 to SS-18)
- [ ] Epic 4: Product catalog API (SS-19 to SS-26)
- [ ] Epic 5: Frontend foundation (SS-27 to SS-33)
- [ ] Epic 6: Public pages (SS-34 to SS-42)
- [ ] Epic 7: Cart (SS-43 to SS-47)
- [ ] Epic 8: Auth UI and protected checkout (SS-48 to SS-52)
- [ ] Epic 9: Orders and payments (SS-53 to SS-64)
- [ ] Epic 10: Static pages (SS-65 to SS-68)
- [ ] Epic 11: Quality and launch (SS-69 to SS-77)
- [ ] Epic 12: Admin panel (SS-78 to SS-84)

## Contributing workflow

- One ticket, one branch, one pull request into `main`.
- Branch name: `feature/SS-12-register-api`. Commit message: `SS-12: add register endpoint`.
- A ticket is done when its acceptance criteria are met, input is validated, `.env.example` is updated if needed, and this README is updated if setup changed.

