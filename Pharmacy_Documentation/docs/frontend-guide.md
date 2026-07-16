# Pharmacy Module - Frontend Developer Guide

This document details the frontend architecture, page routing, and key components of the Pharmacy Next.js application.

## 1. Project Directory Structure

The frontend is a modern Next.js project using TypeScript and Tailwind CSS.

```
frontend/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── layout.tsx          # Root HTML layout and provider wrappers
│   │   ├── page.tsx            # Main landing/search page
│   │   └── pharmacy/           # Pharmacy application namespace
│   │       ├── layout.tsx      # Pharmacy main layout (navigation, notifications)
│   │       ├── page.tsx        # Pharmacy general workspace
│   │       ├── login/          # Login page
│   │       ├── register/       # Registration flow
│   │       ├── settings/       # Organization and profile settings
│   │       ├── admin/          # Admin portal
│   │       ├── operator/       # Operator portal
│   │       └── staff/          # Staff portal
│   ├── context/                # Global React Contexts
│   │   ├── AuthContext.tsx     # Session management and user state
│   │   └── AuthGuard.tsx       # Route protection middleware
│   ├── services/               # HTTP client wrappers
│   │   ├── api.ts              # Core Pharmacy Axios client with auto-refresh interceptors
│   │   └── collaboration.api.ts# Consultation cross-module API services
│   └── styles/                 # Global styles
```

---

## 2. Authentication and Route Security

### 2.1 State Management (`AuthContext.tsx`)
Authentication state is managed globally through a custom React Context.
* **`user`**: Stores parsed JSON representation of the logged-in user profile, including their `role`, `email`, `name`, and linked `organization`.
* **`login(token, userData)`**: Stores JWT access token and user metadata into `localStorage`, sets local state, and refreshes the router.
* **`logout()`**: Clears all items in `localStorage` and redirects to the landing page `/pharmacy`.

### 2.2 Route Guards (`AuthGuard.tsx`)
Component-level middleware that inspects user roles and active sessions:
* Restricts access to `/pharmacy/admin`, `/pharmacy/operator`, and `/pharmacy/staff` based on the logged-in user's role mapping.
* Redirects unauthenticated users to `/pharmacy/login`.

---

## 3. Dashboard Roles & Routes

The application has custom workspaces for each operational user role:

### 3.1 Admin Portal (`/pharmacy/admin`)
For Pharmacy Owners and Administrators.
* **Bookings**: Manage incoming pharmacy orders, approve bookings, and update sub-order statuses.
* **Staff**: Manage personnel, add staff members, and assign roles (`PHARMACY_STAFF` or `PHARMACY_OPERATOR`).
* **Schedule**: Setup daily store timings and configure time slots for patient collection.
* **Payments**: Monitor revenue, manage connected payouts accounts, and track status of transactions/refunds.
* **Collaboration**: Coordination panel to resolve cross-module activities with referring Doctors.

### 3.2 Operator Portal (`/pharmacy/operator`)
For counter staff or intake officers.
* **Bookings Queue**: Review, filter, and modify active orders.
* **Patient Support**: Log inquiries and coordinate with patients on collection statuses.

### 3.3 Staff Portal (`/pharmacy/staff`)
For pharmacy technicians or support agents.
* Work list for picking medicines, uploading verified prescription records, and coordinating deliveries.

---

## 4. API Client Integration

### 4.1 Axios Interceptors (`api.ts`)
The core HTTP client handles background authorization header injection and silent token rotation:
* **Request Interceptor**: Automatically attaches the current `access` token as a `Bearer` header for non-public API endpoints.
* **Response Interceptor**: Listens for `401 Unauthorized` responses with a code of `token_not_valid`. When encountered, it stalls the request, calls `/token/refresh/` using the stored `refresh` token, updates local tokens, and replays the original failed API request.
