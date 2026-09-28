# AI Invoice Manager

> An intelligent, full-stack financial management and invoicing platform powered by React, Node.js, Express, PostgreSQL, and Google Gemini AI.

---

## Overview

**AI Invoice Manager** streamlines the billing lifecycle for freelancers and small businesses. By combining robust database tracking with generative AI capabilities, the platform automates receipt parsing, generates financial executive summaries, drafts tailored payment reminder emails, and assists with writing custom invoice notes and descriptions.

---

## Key Features

- **📷 AI-Powered Receipt Parsing:** Upload receipt images or PDFs to automatically extract vendor details, dates, totals, and line items.
- **📊 Business Insights & Summaries:** SQL-heavy aggregation combined with Gemini AI to generate natural-language financial executive summaries and track overdue accounts.
- **✉️ Context-Aware Payment Reminders:** Automatically draft payment reminder emails with adjustable tones (*friendly*, *firm*, or *final*) based on days overdue and client details.
- **📝 Smart Invoice Note Writer:** Draft descriptions, terms, and custom notes for invoices based on provided line items and prompts.
- **💼 Full Client & Catalog Management:** Manage clients, catalog items, invoices, expenses, and payments in one unified interface.

---

## Tech Stack

### **Backend**
- **Runtime:** Node.js (v25.6+)
- **Framework:** Express.js
- **Database:** PostgreSQL (`pg` pool)
- **Validation:** Zod
- **AI Integration:** `@google/genai` (Google Gemini)
- **Middleware:** `express-rate-limit`, `multer`, custom error & async handlers

### **Frontend**
- **Framework:** React 19 + Vite
- **Styling:** Tailwind CSS, `cva`, `clsx`
- **Data Fetching:** TanStack Query (React Query v5)
- **Charts & Motion:** Recharts, Framer Motion, Lucide React
- **Document Export:** `@react-pdf/renderer`

---

## Architecture & Database Schema

The database relies on a relational PostgreSQL schema:

- **`users` / `company_settings`**: Store user authentication credentials and business details (logo, address, tax rate, invoice prefixes).
- **`clients`**: Track client names, company details, contact emails, and addresses.
- **`catalog_items`**: Save reusable products/services with default rates and billing units (`hour`, `project`, `month`).
- **`invoices` & `invoice_items`**: Core ledger for tracking invoice states (`draft`, `sent`, `paid`), line items, taxes, and sub-totals.
- **`payments` & `expenses`**: Track incoming payments and categorized business expenses across custom date ranges.

---

## API Reference

### **AI Services (`/api/ai`)**

| Method | Endpoint | Description | Payload / Parameters |
| :--- | :--- | :--- | :--- |
| `POST` | `/receipt-parse` | Extract structured data from uploaded image/PDF receipt | `multipart/form-data` (`file`) |
| `POST` | `/business-summary` | Generate natural-language executive summary of monthly metrics | None (Uses user context) |
| `POST` | `/payment-reminder` | Draft a payment reminder email | `{ invoiceId: UUID, tone: "friendly" \| "firm" \| "final" }` |
| `POST` | `/write-note` | Generate invoice description or terms note | `{ kind: "description" \| "terms", prompt?: string, items?: [...] }` |

---

## Getting Started

### **Prerequisites**
- **Node.js**: v20+ installed
- **PostgreSQL**: Local instance or remote database URL (e.g., Neon PostgreSQL)
- **Gemini API Key**: Obtainable from Google AI Studio

---

### **1. Clone the Repository**

```bash
git clone https://github.com/YOUR_USERNAME/aiinvoicemanager.git
cd aiinvoicemanager
```

---

### **2. Backend Setup**

```bash
cd backend

# Install dependencies
npm install

# Configure environment variables
cp .env.example .env
```

Set up your `.env` file:

```env
PORT=5000
DATABASE_URL=postgresql://user:password@localhost:5432/aiinvoicemanager
JWT_SECRET=your_jwt_secret_key
GEMINI_API_KEY=your_google_gemini_api_key
```

Run database migrations and seed demo data:

```bash
npm run db:migrate
npm run db:seed
```

Start the backend development server:

```bash
npm run dev
```

---

### **3. Frontend Setup**

```bash
cd ../frontend/invoicer

# Install dependencies
npm install

# Start Vite dev server
npm run dev
```

The app will be accessible at `http://localhost:5173`.

---

## Security & Rate Limiting

- **Authentication:** JWT bearer tokens in Authorization headers.
- **Route Protection:** `requireAuth` middleware enforces token validation on protected endpoints.
- **Rate Limiting:** `aiLimiter` middleware throttles heavy Gemini API calls to prevent quota exhaustion and abuse.
- **Schema Validation:** Strict runtime type safety with Zod schemas on requests.

---

## License

This project is licensed under the [MIT License](LICENSE).

## Acknowledgements

- Inspired by and built following the tutorial by [Time To Program](https://youtu.be/FrRSqYYzBlk?si=en4tv048txWSNPqb).
