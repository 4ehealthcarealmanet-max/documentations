# 🎨 Frontend Developer Guide

The PathyaTech Lab Frontend is built on **Next.js 15 (App Router)** and utilizes **Tailwind CSS** for layout styling. It is configured to operate in `"use client"` (SPA-like) mode for real-time dashboards and interactive operational consoles.

---

## 🗂️ Project Directory Map

```
frontend/src/
├── app/
│   └── lab/
│       ├── layout.tsx         # Unified Navbar, Sidebar, and Auth Wrapper
│       ├── page.tsx           # Public Test Discovery & Booking Page
│       ├── login/             # Native User Login (Admin/Staff/Operator)
│       ├── register/          # Lab Organization Onboarding
│       ├── admin/             # Lab Admin Dashboard Panel
│       │   ├── page.tsx       # Core Metrics (Total Bookings, Revenue, Staff counts)
│       │   ├── bookings/      # Booking Pipeline & Dossier Views
│       │   ├── collaboration/ # Collaboration Chat & Post Panel
│       │   ├── tests/         # Lab Test Catalog (CRUD)
│       │   ├── schedule/      # Time Slot & Operating Hours Generator
│       │   └── staff/         # Staff Management Panel
│       └── operator/          # Mobile-First Phlebotomist Portal
│           ├── page.tsx       # Assigned Collections List
│           └── bookings/      # Step-by-step Sequential Status Tracker
├── context/
│   └── AuthContext.tsx        # JWT session storage and Profile sync
├── services/
│   ├── api.ts                 # Axios Base Config & Auto-Refresh Interceptor
│   ├── auth.api.ts            # Auth requests
│   ├── lab.api.ts             # Booking, Slot, Cart & Report transactions
│   └── collaboration.api.ts   # Chat, Posts & Collaboration Alerts
└── styles/
    └── globals.css            # Base Styles & Custom Tailwind Tokens
```

---

## 🔐 Session Management (`AuthContext`)

Authentication state is shared globally across the frontend application using a custom React Context.

* **File Location**: `src/context/AuthContext.tsx`
* **Tokens**: Stored in `localStorage` as `access` (short-lived API access) and `refresh` (long-lived rotation token).
* **Usage**:
  ```tsx
  import { useAuth } from "@/context/AuthContext";

  const MyComponent = () => {
    const { user, organization, logout } = useAuth();
    // user contains: id, email, role, full_name
    // organization contains: id, name, verification_status
  };
  ```

---

## 📡 API Request Pipeline (`Axios Interceptors`)

To avoid session expiration issues during active operation, the Axios client automatically intercepts 401 errors, requests a new access token using the refresh token, and retries the original request.

```
Request ──> [Axios Request Interceptor: Attach Bearer Token] ──> API Server
                                                                   │
                                                [401 Unauthorized] │
                                                                   ▼
Request <── [Axios Response Interceptor: Retry Request] <── [Call /token/refresh/]
```

* **Axios Instance**: Located in `src/services/api.ts`.
* **Important**: If the token refresh API fails or no refresh token exists, the interceptor clears local storage and redirects the user to the landing page `/lab/login` immediately.

---

## 🎨 Design System & Responsiveness

The application is styled with a premium, high-contrast Slate and Blue palette. It features subtle hover micro-animations (`transition-all duration-300 hover:scale-[1.02]`) and soft glassmorphism.

### 📱 Responsive Layout Conversions (Desktop Table $\leftrightarrow$ Mobile Cards)
Admin dashboards are heavily optimized for split views. Tables automatically morph into rich detail cards on smaller viewports.

**Implementation Example:**
```tsx
{/* Desktop Table View */}
<div className="hidden md:block overflow-x-auto">
  <table className="w-full text-left">
    <thead>
      <tr className="border-b border-slate-100">
        <th>Patient</th>
        <th>Date</th>
        <th>Status</th>
      </tr>
    </thead>
    <tbody>
      {bookings.map(b => (
        <tr key={b.id}>
          <td>{b.patient_name}</td>
          <td>{b.appointment_date}</td>
          <td>{b.status}</td>
        </tr>
      ))}
    </tbody>
  </table>
</div>

{/* Mobile Cards View */}
<div className="block md:hidden space-y-4">
  {bookings.map(b => (
    <div key={b.id} className="p-4 bg-white border border-slate-100 rounded-2xl shadow-sm">
      <div className="font-bold">{b.patient_name}</div>
      <div className="text-slate-500">{b.appointment_date}</div>
      <span className="badge">{b.status}</span>
    </div>
  ))}
</div>
```

---

## 🛠️ Common UI Best Practices

1. **Defensive Value Extraction**: Since relations from backend API endpoints can arrive either as raw UUID strings or fully nested objects depending on authorization scopes, always extract values defensively:
   ```ts
   const organizationId = typeof org === 'object' ? org.id : org;
   ```
2. **Tabbed Navigation States**: Dashboard pages (such as Scheduling or Collaboration) persist the selected tab inside URL hash tags (`#posts`, `#chat`) or search parameters (`?tab=conversations`) to support browser history navigation.
3. **Optimistic UI Updates**: During real-time operations like assigning a collector, the frontend updates local state immediately before the network request resolves, reverting back only if a failure occurs.
