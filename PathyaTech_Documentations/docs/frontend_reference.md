# PathyaTech Frontend Reference Manual

This document details the frontend codebase, application structure, state management, routes, and API communication services for the Next.js client application.

---

## 1. Directory Structure (Next.js App Router)

The frontend is built with Next.js 14 (App Router) and is structured inside `frontend/src/` as follows:

```
frontend/
├── tsconfig.json
├── tailwind.config.js       # Configuration for CSS utility bounds (if needed)
├── package.json
└── src/
    ├── app/                 # Application routing entry point
    │   ├── auth/            # Onboarding routes: login, signup, OTP validation
    │   ├── patient/         # Patient dashboard, medical vault, diet planner, lab booking
    │   ├── doctor/          # Doctor dashboard, slot manager, prescriptions, AI consultation panel
    │   ├── clinic/          # Receptionist dashboard, walk-in register, content editor
    │   ├── platform/        # Admin control console, licensing verification lists
    │   ├── layout.tsx       # Root layout defining html/body headers
    │   └── page.tsx         # Root redirect routing node
    ├── components/          # Reusable shared UI nodes (buttons, inputs, cards, layouts)
    ├── context/             # Global Context API definitions
    ├── services/            # Backend REST API wrappers and HTTP connection clients
    └── styles/              # Global stylesheet index (`globals.css`) and modular overrides
```

---

## 2. Page Routing Matrix

Here are the primary paths mapped out under `src/app/`:

| URL Path | Filesystem Endpoint | Purpose | Access Control |
|---|---|---|---|
| `/` | `src/app/page.tsx` | Main platform landing/redirect gate | Public |
| `/auth/login` | `src/app/auth/login/page.tsx` | Account password login page | Public |
| `/auth/signup` | `src/app/auth/signup/page.tsx` | Mobile/Email profile sign-up and OTP trigger | Public |
| `/patient` | `src/app/patient/page.tsx` | Patient landing page (Vitals, Diet Tracker) | Patient Only |
| `/patient/vault` | `src/app/patient/vault/page.tsx` | Patient personal medical file vault | Patient Only |
| `/patient/labs` | `src/app/patient/labs/page.tsx` | Diagnostics test booking checkout cart | Patient Only |
| `/doctor` | `src/app/doctor/page.tsx` | Doctor consultations overview dashboard | Verified Doctor |
| `/doctor/slots` | `src/app/doctor/slots/page.tsx` | Doctor operating calendar slot configurator | Verified Doctor |
| `/clinic` | `src/app/clinic/page.tsx` | Receptionist patient reception dashboard | Active Receptionist |
| `/clinic/content` | `src/app/clinic/content/page.tsx` | Clinic case studies & promotional reels manager | Active Receptionist |
| `/platform` | `src/app/platform/page.tsx` | Admin account dashboard and verification queue | Administrator |

---

## 3. Global State Management (Context API)

State configurations are located under `src/context/`:
- **`AuthContext`**:
  - Manages active user details (`id`, `name`, `email`, `role`, `status`).
  - Stores the active JWT access token in memory and local session headers.
  - Automatically invokes token refresh requests if credentials expire.
  - Handles login, signup redirect callbacks, and logout clears.
- **`DashboardContext`**:
  - Manages dashboard menu tabs, active sidebar states, and search queries.

---

## 4. API & Integration Service Layer

All server communication is abstracted under `src/services/` to keep UI components decoupled from networking:
- **`api.ts` (Axios Client)**:
  - Initialized with `NEXT_PUBLIC_API_BASE_URL` (typically `http://127.0.0.1:8000/api`).
  - Sets up request interceptors to append the `Authorization: Bearer <token>` header dynamically.
  - Sets up response interceptors to catch `401 Unauthorized` errors and trigger token rotation before repeating requests.
- **Service Modules**:
  - `auth.ts`: Signs up, logs in, triggers/verifies mobile/email OTPs.
  - `patient.ts`: Manages profiles, updates diet planners, retrieves vault files.
  - `doctor.ts`: Configures slots, posts prescriptions, and triggers Gemini AI clinical assistance nodes.
  - `clinic.ts`: Books walk-in registrations, logs offline payments, submits reels.
  - `admin.ts`: Audits profiles, updates verification flags, modifies payout details.
