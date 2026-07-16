# PathyaTech Backend Reference Manual

This document details the backend architecture, application modules, database schemas, and REST APIs that run the PathyaTech Healthcare Platform.

---

## 1. Django Application Directory Layout

The Django backend is modularized into several focused application modules under `backend/apps/`:

```
backend/
├── manage.py
├── core/                    # Core project configurations (settings, routing, JWT config)
└── apps/
    ├── auth_app/            # Custom User model, Organization memberships, OTPs, staff assignments
    ├── patients/            # Patient physical profiles, medical & lifestyle history
    ├── doctors/             # Doctor profile, credentials, availability slots, experiences, keywords
    ├── appointments/        # Consultation slots, reservation logs, meeting integration
    ├── prescriptions/       # Medical prescriptions, items, diagnostic recommendations
    ├── payments/            # Transaction records, refunds, and payouts
    ├── content/             # Article categories, case studies, video reels (Cloudinary hosted)
    ├── subscriptions/       # Pricing models, active packages, usage tracker logs
    ├── documents/           # Secure Medical and Entity document vault with audit trails
    ├── communication/       # Chats, messaging, reminder system
    └── utils/               # Shared validators and helper libraries
```

---

## 2. Core Database Schema & Relational Models

Here is a summary of the database models across the application apps:

### A. Authentication & Users (`apps/auth_app`)
- **`User` (Table: `consultation_auth_user`)**:
  - `id` (UUID): Primary Key.
  - `email` (String, unique): Login ID.
  - `phone` (String, unique): Verified contact.
  - `name` (String): User's display name.
  - `role` (Choices: `PATIENT`, `DOCTOR`, `DOCTOR_RECEPTIONIST`, `ADMIN`).
  - `status` (Choices: `ACTIVE`, `INACTIVE`, `BLOCKED`).
  - `is_staff`, `is_active` (Booleans).
- **`OrganizationMembership`**:
  - Relates a `User` (Staff/Operator) to an `Organization` (Hospital/Clinic/Lab/Pharmacy).
  - Fields: `user` (FK), `organization` (FK), `role` (ADMIN, STAFF, OPERATOR), `is_active` (Boolean).
- **`DoctorAssignment`**:
  - Bridges receptionists or assistant nurses to a specific doctor.
  - Fields: `user` (Receptionist, FK), `doctor` (Doctor, FK), `organization` (FK, optional), `role` (RECEPTIONIST, NURSE), `is_active` (Boolean).
- **`OTP` (Table: `consultation_otp`)**:
  - Tracks registration/login OTPs.
  - Fields: `user` (FK), `otp` (6-digit string), `is_verified` (Boolean), `created_at` (expires in 10 minutes).

### B. Patients (`apps/patients`)
- **`PatientProfile`**:
  - Holds demographics: `full_name`, `gender`, `date_of_birth`, `city`, `state`, `country`, `blood_group`.
- **`PatientMedicalProfile`**:
  - Relates to patient profile. Tracks `chronic_conditions`, `allergies`, `current_medications`, `surgical_history`.
- **`PatientLifestyleProfile`**:
  - Tracks `food_preference` (Veg, Non-Veg, Vegan), `smoking_habits`, `alcohol_consumption`.
- **`LockSlot`**:
  - Holds temporarily reserved appointment slots before payment. Resets if unpaid.

### C. Doctors (`apps/doctors`)
- **`DoctorProfile`**:
  - Specialization, bio, ratings, license number, Aadhaar number, verification status (`PENDING`, `APPROVED`, `REJECTED`), and introduction reel URL.
- **`DoctorAvailability`**:
  - Effective date ranges, mode of consultation (ONLINE, OFFLINE), and active status.
- **`DoctorEducation` & `DoctorExperience`**:
  - Detailed professional history (degrees, institutions, past clinics).
- **`DoctorPaymentAccount`**:
  - Holds bank credentials and payout configurations.

### D. Appointments & Slots (`apps/appointments`)
- **`Slot`**:
  - Core slot entries. Fields: `doctor` (FK), `date`, `start_time`, `status` (`AVAILABLE`, `LOCKED`, `BOOKED`), `mode` (`ONLINE`, `OFFLINE`), `fee` (Decimal).
- **`Appointment`**:
  - Tracks active consultations. Fields: `slot` (FK), `patient` (FK), `doctor` (FK), `status` (`CONFIRMED`, `PENDING_PAYMENT`, `RESCHEDULED`, `CANCELLED`, `COMPLETED`), `meeting_link` (Zoom).

### E. Prescriptions & Recommendations (`apps/prescriptions`)
- **`Prescription`**:
  - Relates to Appointment. Tracks diagnosis details, doctor advice, and status (`DRAFT`, `FINAL`).
- **`PrescriptionItem`**:
  - Specific medicines, quantities, dosages, and intake frequencies.
- **`PrescribedLabTest`**:
  - Doctor recommended lab tests. Tracks `test_name`, `is_booked` (flagged `True` when patient checks out relevant test in their cart).

---

## 3. Custom Permission Classes & RBAC

The API uses custom permission gates to enforce role boundaries:
- **`IsDoctor`**: Checks that the user's role is `DOCTOR` and their profile verification status is `APPROVED`.
- **`IsReceptionist`**: Confirms the user's role is `DOCTOR_RECEPTIONIST` and that they have an active `DoctorAssignment` record.
- **`IsAdmin`**: Validates the user's role is `ADMIN` or that `is_staff` is `True`.
- **`IsPatientOwnerOrDoctorReadOnly`**: Restricts document vault access. Patients can modify/view their own records. Doctors can only view records if `shared_with_doctor` is set to `True`.

---

## 4. Primary REST API Endpoint Mapping

### Auth & Onboarding
- `POST /api/auth/signup/` - Registers a new user.
- `POST /api/auth/send-otp/` - Dispatches OTP to verified phone number.
- `POST /api/auth/verify-otp/` - Verifies OTP and issues JWT tokens.
- `POST /api/auth/login/` - Standard password login.
- `POST /api/auth/reset-password/` - Direct profile password reset.

### Patient Dashboard APIs
- `GET/PUT/PATCH /api/patients/profile/` - Accesses personal lifestyle and health vital profiles.
- `POST /api/patients/appointments/book/` - Books an appointment slot (generates Razorpay order).
- `PATCH /api/patients/appointments/<id>/reschedule/` - Reschedules a pending booking.
- `GET /api/patients/vault/` - Accesses personal document vault.

### Doctor Workflows
- `GET /api/doctors/appointments/` - Lists scheduled consultations.
- `POST /api/doctors/appointments/<id>/ai-diagnosis/` - Feeds details to Gemini AI for potential condition flags.
- `POST /api/doctors/lab-reports/<id>/ai-analysis/` - Parses report PDF values using Gemini.
- `POST /api/doctors/appointments/<id>/ai-custom-suggestion/` - Synthesizes doctor's custom query.
- `POST /api/prescriptions/` - Issues digital medical recipes.

### Receptionist Actions
- `POST /api/clinic/walk-in/` - Registers walk-in patients and records cash payments.
- `POST /api/clinic/reminders/` - Dispatches SMS/WhatsApp alerts for upcoming slots.
- `POST /api/clinic/marketing/case-studies/` - Creates case study boosters for moderation.
