# AICTE Institution Complaint Monitoring Portal

A working prototype of the AICTE complaint workflow: institutions submit and track complaints, AICTE cells review, route, transfer and resolve them, and officials monitor deadlines and analytics.

## Requirements
- Node.js 22 or newer
- npm (comes with Node.js)
- Windows, macOS, or Linux

## Quick start (development)
1. From the project root, install everything:
   ```
   npm run install:all
   ```
2. Set up the database and sample data, in one step:
   ```
   npm run setup
   ```
   This creates `server/.env` if missing, generates the database client, creates or updates the tables, and loads sample data. It is safe to run again. If it asks whether to reset the database, answer `n` to keep your data. Answer `y` only on a fresh database.
3. Start both servers:
   ```
   npm run dev
   ```
4. Open http://localhost:5173

To start over with fresh sample complaints, run `npm run db:reset` in the `server` folder. This deletes all data.

Keep one project folder. Extracting a new zip over it keeps your database and `server/.env`, so you don't need to set up again.

## Environment variables (`server/.env`)
| Variable | Purpose | Default |
|---|---|---|
| `PORT` | API port | `4000` |
| `CLIENT_ORIGIN` | Address allowed to call the API | `http://localhost:5173` |
| `NODE_ENV` | `development` or `production` | `development` |
| `DATABASE_URL` | SQLite database file | required |
| `JWT_SECRET` | Signs login tokens | required (production refuses the example value) |
| `JWT_EXPIRES_IN` | How long a login lasts | `8h` |
| `ANTHROPIC_API_KEY` | Enables AI priority analysis | blank, which uses keyword rules |
| `ANTHROPIC_MODEL` | Model used for priority analysis | `claude-haiku-4-5` |

## Tuning the rules
Every threshold is in `server/src/lib/config.js` and can be changed in `server/.env` without editing code:
- `READ_ONLY_AFTER_HANDOFF`: `true` (default) keeps a transferring cell in view-only access. `false` removes its access.
- `DNA_WINDOW_DAYS`, `DNA_DUPLICATE_SCORE`, `DNA_RELATED_SCORE`: how similar complaints are found and labelled.
- `SLA_HIGH_DAYS`, `SLA_MEDIUM_DAYS`, `SLA_LOW_DAYS`: expected disposal times.
- `DUE_SOON_DAYS`, `HANDOFF_STALE_HOURS`: when alerts appear.
- `DEFAULT_CELL_CAPACITY`: the open-case capacity used by the what-if simulator.
The category-to-cell routing and the priority keywords are in the same file.

## Demo accounts
All demo accounts use the password `Password@123`.

| Account | Role |
|---|---|
| `abc@college.test` (or institution ID `1-1000000001`) | Institution |
| `xyz@college.test` (or `1-1000000002`) | Institution |
| `rb.officer@aicte.test` | Officer, Regulation Bureau |
| `pgrc.officer@aicte.test` | Officer, Public Grievance Redressal Cell |
| `asc.officer@aicte.test` | Officer, Academic Standards Cell |
| `adm.officer@aicte.test` | Officer, Administration cell |
| `admin@aicte.test` | Admin, all cells |

Institution `1-1000000003` has no account. Register it from the Create an account page.

## What the system does
- **Institutions:** submit complaints with documents, see a priority assessment and similar cases, track status, and reply when more information is requested.
- **Priority:** an AI check reads the description (when an API key is set) and falls back to keyword rules. The reason is recorded in the history.
- **Similarity (DNA):** complaints from the same institution in the last 180 days are compared. Duplicates, related cases, and overlaps with other cells are flagged.
- **Routing:** each category has a lead cell. Cells holding linked complaints appear as supporting cells.
- **Officer workflow:** request more information, transfer to another cell (which must acknowledge), or resolve. Each step is recorded in the history.
- **Monitoring:** deadlines, alerts, workload by cell, and search filters.
- **Analytics:** bureau and region reports, ageing, monthly trend, and a what-if workload forecast.

## Scripts
Run from the project root unless noted.

| Command | What it does |
|---|---|
| `npm run install:all` | Installs root, server, and client packages |
| `npm run dev` | Starts the API and the web app in development |
| `npm run build` | Builds the web app into `client/dist` |
| `npm start` | Starts the server, serving the built app in production mode |
| `npm run setup` | Creates the database and sample data, or updates them safely |
| `cd server` then `npm test` | Runs the automated tests (29 tests) |
| `cd server` then `npm run smoke` | Checks the running app: logins, access rules, and officer views. Start the app first |
| `cd server` then `npm run db:reset` | Deletes all data and reloads the sample data |

## Running in production (Windows)
1. Set `JWT_SECRET` in `server/.env` to a long random value. Set `NODE_ENV=production`.
2. From the project root:
   ```
   npm run install:all
   npm run build
   cd server
   npx prisma migrate deploy
   cd ..
   set NODE_ENV=production
   npm start
   ```
3. Open http://localhost:4000. The API and the app are served from the same address.

## Troubleshooting
- **"Request failed" or "Internal server error" on login:** check the server window for the first red error. Usually the database is missing, or `server/.env` has no `DATABASE_URL`.
- **`npm run dev` says `concurrently` is not recognized:** run `npm run install:all` from the project root.
- **Prisma says the client did not initialise:** run `npx prisma generate` in the `server` folder.
- **Too many login attempts:** wait 15 minutes, or restart the server during testing.
- **Server restarts in a loop:** make sure the project is not inside OneDrive. Move it to a folder such as `C:\dev`.

## Security notes
- Passwords are hashed with bcrypt. Login tokens expire after `JWT_EXPIRES_IN`.
- Login and registration are rate-limited. Other API routes have a generous global limit.
- Uploads are limited to PDF, JPG and PNG files of 5 MB each, with up to 5 per complaint.
- Security headers are set with Helmet.
- Before real use: change every demo password, use a strong `JWT_SECRET`, serve the app over HTTPS, and review the rules for priority and routing with AICTE.

## Known limits (prototype)
- Storage is SQLite and uploads are kept on local disk. Move to a shared database and object storage for multi-user production.
- Priority and routing use simple rules, and the forecast is a trend estimate.
- Monitoring shows all cells. Per-cell scoping for monitoring is not yet implemented.
- Results lists show the first 100 matches.

## Public tracking and accessibility
- **Track a complaint:** anyone can open `/track`, enter the complaint ID and the email the institution registered, and see the status, the plain-language explanation, the expected date, and the update history. The complaint text and internal notes are never shown. Lookups are limited to 30 attempts per 15 minutes.
- **Display settings:** every page has a footer with "Larger text" and "High contrast" options. Settings are remembered on that computer.
- **Keyboard and screen readers:** a "Skip to main content" link appears at the top of each page, focus is always visible, and motion is reduced when the computer asks for it.
"# sih" 
