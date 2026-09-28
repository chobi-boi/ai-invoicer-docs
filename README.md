# AI Invoice Manager

> A full-stack invoicing app with client management, invoices, payments, expenses, reports, and AI features (receipt scanning, summaries, reminders) powered by Google Gemini.

---

## Overview

**AI Invoice Manager** streamlines the billing lifecycle for freelancers and small businesses. By combining robust database tracking with generative AI capabilities, the platform automates receipt parsing, generates financial executive summaries, drafts tailored payment reminder emails, and assists with writing custom invoice notes and descriptions.

---
<img width="1883" height="848" alt="ai-invoice-manager" src="https://github.com/user-attachments/assets/ce3e629f-acba-4e4b-9f45-9a44f39fe851" />

## Key Features

- **📷 AI-Powered Receipt Parsing:** Upload receipt images or PDFs to automatically extract vendor details, dates, totals, and line items.
- **📊 Business Insights & Summaries:** SQL-heavy aggregation combined with Gemini AI to generate natural-language financial executive summaries and track overdue accounts.
- **✉️ Context-Aware Payment Reminders:** Automatically draft payment reminder emails with adjustable tones (*friendly*, *firm*, or *final*) based on days overdue and client details.
- **📝 Smart Invoice Note Writer:** Draft descriptions, terms, and custom notes for invoices based on provided line items and prompts.
- **💼 Full Client & Catalog Management:** Manage clients, catalog items, invoices, expenses, and payments in one unified interface.

---

- **Frontend:** React + Vite (`frontend/invoicer`)
- **Backend:** Node.js + Express, CommonJS (`backend`)
- **Database:** PostgreSQL (Neon)
- **Auth:** JWT stored in an HTTP-only cookie
- **AI:** Google Gemini API
- **Deployment:** Dokploy (Nixpacks), single service
## Project structure
 
```
.
├── package.json            # root workspace (build + start scripts)
├── backend/
│   ├── package.json
│   └── src/
│       ├── server.js       # Express app, API routes, serves the built frontend in production
│       ├── config/         # env + database setup
│       ├── middleware/
│       ├── models/
│       ├── routes/
│       └── services/
└── frontend/
    └── invoicer/           # Vite React app
```
 
In production, one Node process serves both the API (`/api/*`) and the built frontend (`frontend/invoicer/dist`), so there is no CORS setup and the frontend calls the API with relative paths (`/api/...`).
 
## Prerequisites
 
- Node.js 18 or newer
- A [Neon](https://neon.tech) PostgreSQL database (or any PostgreSQL instance)
- A [Google AI Studio](https://aistudio.google.com/apikey) API key for Gemini
## Local development
 
### 1. Clone and install
 
```bash
git clone https://github.com/chobi-boi/ai-invoicer.git
cd ai-invoicer
npm install          # workspaces install backend and frontend together
```
 
### 2. Configure the backend
 
Create `backend/.env` (this file is git-ignored, never commit it):
 
```env
NODE_ENV=development
PORT=8000
DATABASE_URL=postgresql://USER:PASSWORD@HOST/DBNAME?sslmode=require
JWT_SECRET=replace-with-a-long-random-string
JWT_EXPIRES=7d
COOKIE_NAME=aivm_token
CLIENT_ORIGIN=http://localhost:5173
GEMINI_API_KEY=your-gemini-key
GEMINI_MODEL=gemini-2.5-flash
```
 
`PORT=8000` matters locally: the Vite dev server proxies `/api` to `http://localhost:8000` (see `frontend/invoicer/vite.config.js`).
 
Generate a strong secret with:
 
```bash
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```
 
### 3. Run both apps
 
In two terminals:
 
```bash
# Terminal 1: backend (API on http://localhost:8000)
npm --prefix backend run dev
 
# Terminal 2: frontend (http://localhost:5173)
npm --prefix frontend/invoicer run dev
```
 
Open http://localhost:5173. The API health check is at http://localhost:8000/api/health.
 
> Verify the backend dev script name in `backend/package.json`. If it has no `dev` script, use `npm --prefix backend start`.
 
## Environment variables
 
| Variable | Required | Description |
|---|---|---|
| `NODE_ENV` | yes | `development` locally, `production` when deployed |
| `PORT` | yes | Port the server listens on (`8000` locally, `3000` on Dokploy) |
| `DATABASE_URL` | yes | PostgreSQL connection string (Neon) |
| `JWT_SECRET` | yes | Long random string used to sign auth tokens |
| `JWT_EXPIRES` | no | Token lifetime, e.g. `7d` |
| `COOKIE_NAME` | no | Auth cookie name, e.g. `aivm_token` |
| `CLIENT_ORIGIN` | yes | Public site URL, no trailing slash, e.g. `https://aiinvoicer.example.com` |
| `GEMINI_API_KEY` | yes | Google Gemini API key |
| `GEMINI_MODEL` | no | Model name, e.g. `gemini-2.5-flash` |
 
If the Neon connection fails with an authentication error, remove `&channel_binding=require` from `DATABASE_URL` and keep `sslmode=require`.
 
## Production build (local test)
 
```bash
npm run build      # installs dependencies and builds frontend/invoicer/dist
NODE_ENV=production npm start
```
 
Then open the app at `http://localhost:<PORT>`. The root `package.json` scripts are:
 
```json
"build": "npm install --include=dev && npm run build --workspace frontend/invoicer",
"start": "npm --prefix backend start"
```
 
`--include=dev` is required because Vite is a devDependency and `NODE_ENV=production` would otherwise skip it.
 
## Deploying on Dokploy
 
1. **Create an Application** and connect this GitHub repository (branch `main`).
2. **Build type:** Nixpacks. **Root / base directory:** `/` (the repository root, not `backend`).
3. **Environment tab:** add every variable from the table above. Use production values:
   - `NODE_ENV=production`
   - `PORT=3000`
   - `CLIENT_ORIGIN=https://your-domain.com` (exact origin, no trailing slash)
   - a newly generated `JWT_SECRET`
4. **Domains:** add your domain, set the container port to `3000`, and enable HTTPS (Let's Encrypt).
5. **Health check (optional):** path `/api/health`.
6. **Deploy.** Nixpacks runs `npm run build` then `npm start`.
Verify after deploy:
 
- `https://your-domain.com/` loads the app
- `https://your-domain.com/api/health` returns `status: ok` and `db: connected`
- Register, log in, create an invoice, and try an AI feature
### Auto-deploy
 
Enable auto-deploy in Dokploy so pushes to `main` redeploy automatically.
 
## Security notes
 
- Never commit `.env` files. They are covered by `.gitignore`; Dokploy holds production secrets in its Environment tab.
- If a secret was ever committed, rotating it is the only real fix, because Git history keeps old values. Rotate `JWT_SECRET`, the database password, and the Gemini key.
- Keep `.env.example` (placeholders only) in the repo as a template.
## Troubleshooting
 
| Symptom | Likely cause |
|---|---|
| `Nixpacks was unable to generate a build plan` | Root directory is set to a subfolder, or the root `package.json` is missing |
| `vite: not found` during build | Build script is missing `--include=dev` |
| Blank page or `Cannot GET /` | `frontend/invoicer/dist` was not built, or the static block in `server.js` sits after `notFound` |
| Frontend folder empty after clone | `frontend/invoicer` was committed as a nested Git repo; remove its inner `.git` and re-add it |
| Requests go to `localhost` in production | Frontend API base URL is not the relative `/api` |
| Login works but the session is lost | `CLIENT_ORIGIN` has a trailing slash or does not match the site URL |
| Database connection errors | Wrong `DATABASE_URL`, or drop `channel_binding=require` |
 
## Acknowledgements

- Inspired by and built following the tutorial by [Time To Program](https://youtu.be/FrRSqYYzBlk?si=en4tv048txWSNPqb).
