# SmartSpend — Smart Expense Tracker

A local personal finance app with an Express API, Prisma/SQLite persistence, cookie authentication, and a responsive React dashboard. Multiple accounts are isolated by user ID. The sample database seed creates `demo@example.com` with password `DemoPass123`.

## Requirements

- Node.js 20 or newer and npm
- VS Code (optional; recommended extensions are listed in `.vscode/extensions.json`)

## Setup

From the repository root in PowerShell:

```powershell
Copy-Item .env.example .env
# Edit server/.env and replace JWT_SECRET with a random value of at least 32 characters.
npm install
npx prisma migrate dev --name init
npm run seed
npm run dev
```

Open <http://localhost:5173>. The API runs at <http://localhost:4000/api/v1>. Vite proxies browser requests through the configured API URL (`VITE_API_URL`, default `http://localhost:4000/api/v1`). For local HTTP, cookies are `httpOnly`, `SameSite=Lax`; production uses the secure cookie flag.

The `DATABASE_URL` in `.env` is `file:./dev.db`; Prisma resolves this relative to `server/prisma/schema.prisma`, so the database file is `server/prisma/dev.db`. Keep `.env` private. The server loads this root environment file when run from its package folder.

## Commands

```powershell
npm run dev       # Vite and API together
npm test          # Jest/Supertest and Vitest/RTL
npm run lint      # ESLint on server and client
npm run build     # production client build
npm run seed      # idempotent categories and 60 days of sample activity
npm run smoke     # API smoke check against a running server
```

Tests use `server/prisma/test.db` rather than the development database. To run smoke, start the API, seed a user (or create any first user with categories), then run `npm run smoke` from another terminal.

## API overview

All endpoints use `/api/v1`. Register and login set a signed JWT in an `httpOnly` cookie. Except for auth endpoints, requests require that cookie. Validation errors and other failures use `{ "error": { "code", "message", "details" } }`.

| Area          | Routes                                                                                    |
| ------------- | ----------------------------------------------------------------------------------------- |
| Auth          | `POST /auth/register`, `/auth/login`, `/auth/logout`; `GET /auth/me`                      |
| Transactions  | `GET/POST /transactions`, `GET/PUT/DELETE /transactions/:id`                              |
| Categories    | `GET/POST /categories`, `PUT/DELETE /categories/:id`                                      |
| Budgets       | `GET/POST /budgets`, `GET /budgets/status?month=YYYY-MM`, `PUT/DELETE /budgets/:id`       |
| Reports       | `GET /reports/summary`, `/reports/by-category`, `/reports/trend`, `/reports/top-expenses` |
| Smart         | `POST /smart/categorize`, `GET /smart/insights`                                           |
| Savings goals | `GET/POST /goals`, `PUT/DELETE /goals/:id`, `POST /goals/:id/contributions`               |
| Import/export | `GET /export/csv`, `POST /export/import/csv`; `PATCH /settings`                           |

REST Client examples are in `http/`. The JSON export collection import is supported in Thunder Client from the bundled VS Code extension; use the same endpoint examples and cookies.

## Back up your data

Stop the server and copy `server/prisma/dev.db` to a safe location. Keep a copy of `server/.env` separately so you can restore the signing secret. Restart the server after copying.

## Troubleshooting

- **Prisma cannot find `DATABASE_URL`:** ensure the root `.env` exists (copy `.env.example`) and run Prisma commands from the repository root as shown.
- **Port already in use:** set `PORT` in `server/.env`; Vite uses port 5173.
- **Browser reports a network or CORS error:** keep `CLIENT_ORIGIN=http://localhost:5173`, and check the API is listening on port 4000.
- **Prisma client missing:** run `npm --prefix server exec prisma -- generate`.
- **Blank database:** run `npm run seed`; migrations must exist first.
- **SQLite database locked:** stop any second API process and close SQLite viewer tabs before restarting.

## Notes

Amounts are stored as integer paise in SQLite and returned as rupee values by the API, so calculations avoid floating point storage. New accounts get starter categories; the seed adds an expanded category set and 60 days of demo activity. The local product supports merchant keyword categorization, user-learned rules after category corrections, forecast and budget insight calculations based on user transactions, savings goals with contributions, and recurring-rule execution through `/recurring`. Budget alert thresholds are evaluated when an expense is saved and included in budget status. A schema migration adds savings-goal storage. This app is intended for local personal finance tracking; configure HTTPS, secure secrets, and an operational database before public deployment.
