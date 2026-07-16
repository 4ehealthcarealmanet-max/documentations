# PathyaTech Actor Lifecycles & Workflow Operations

This document charts the lifecycle states, transition triggers, business rules, and technical data pipelines for all four core actors: Patients, Doctors, Receptionists, and Admins.

---

## 1. Actor Lifecycles & States

### A. Patient Lifecycle
```
[Discovery] ──► [OTP Verified] ──► [Active Account] ──► [Slot Booking] ──► [Consultation] ──► [Lab Tests] ──► [Diet Plans]
```
1. **Discovery & Onboarding**: Patient registers via frontend or receptionist walk-in. Account status begins at `PENDING_VERIFICATION`. Verification of OTP changes status to `ACTIVE`.
2. **Profile Setups**: Patient logs physical parameters (`PatientProfile`), medical conditions (`PatientMedicalProfile`), and dietary habits (`PatientLifestyleProfile`).
3. **Medical Vault Uploads**: Patient uploads documents. Privacy control `shared_with_doctor` defaults to `True` (viewable by doctors during booking) but can be toggled to `False` (making the file private).
4. **Consultation Flow**: Patient selects doctor, locks slot, completes Razorpay, and receives a Zoom link.
5. **Lab Diagnostics**: Patient receives test suggestions, places order (HOME or LAB_VISIT), phlebotomist tracks progress (`SCHEDULED` -> `PICKED` -> `IN_TRANSIT`), and lab publishes PDF results.
6. **Continuous Management**: Patient logs biomarkers, accesses nutritionist-approved 7-day diet plans, and schedules follow-up appointments.

### B. Doctor Lifecycle
```
[Register] ──► [Upload Licenses] ──► [Admin Verification] ──► [Active Profiles] ──► [Manage Slots] ──► [Clinical Care] ──► [Payouts]
```
1. **Onboarding**: Doctor registers, creating a profile with status `PENDING`. Uploads license/degrees to `EntityDocument` and an intro video reel.
2. **Admin Audit**: Admin reviews files. If approved, profile verification status changes to `APPROVED`, publishing doctor details to search lists.
3. **Scheduling**: Doctor configures working calendar using `DoctorAvailability`, generating open database `Slot` models.
4. **Consultations**: Doctor reviews patient's shared vault documents, consults online/offline, drafts prescriptions, and creates lab recommendations.
5. **AI Interpretation**: Doctor runs AI report analysis on completed tests to view briefings and urgency flags (Low/Medium/High).
6. **Boosters & Payouts**: Doctor logs bank credentials to trigger transaction payouts and submits case study boosters to increase search ranking.

### C. Receptionist Lifecycle
1. **Onboarding**: Created by admin and linked to a clinic doctor via `DoctorAssignment`.
2. **Walk-in Registrations**: Registers walk-in patients directly, skipping OTP flows, and locks available slots.
3. **Billing**: Records offline CASH payments, immediately confirming bookings without opening payment gateways.
4. **Clinic Queue Operations**: Monitors patient attendance (`PENDING` -> `ARRIVED` -> `IN_CONSULTATION`), updating waitlists.
5. **Marketing Support**: Drafts case studies and records clinic reels, uploading media files to Cloudinary for admin verification.

### D. Platform Administrator Lifecycle
1. **Onboarding**: Granted `is_staff = True` or `role = ADMIN` access.
2. **Entity Verification**: Approves or rejects doctor profiles and clinic organization licenses.
3. **Financial Oversight**: Reviews transaction ledgers, coordinates refunds, and manages platform payout accounts.
4. **Content Moderation**: Approves or rejects case study boosters and marketing reels before publication.

---

## 2. Key Workflow Operations

### A. Appointment Booking & Slot Locking
1. **Lock Slot**: Patient selects a slot. System inserts `LockSlot` with status `LOCKED`, reserving the database `Slot` for 10 minutes.
2. **Paid Checkout**: For paid slots, system creates a pending entry in `Payment_Table` and returns a Razorpay order.
3. **Verify Payment**:
   - Razorpay signature is verified server-side using HMAC SHA256 hashing.
   - If verified, `Payment_Table` status is set to `SUCCESS`, the `Slot` becomes `BOOKED`, the appointment moves to `CONFIRMED`, and a Zoom link is created.
   - If signature verification fails or the 10-minute lock expires, the slot is released back to `AVAILABLE`.

### B. Rescheduling Rules
- **Access Limits**: Max 2 reschedules per appointment chain.
- **Validity Gate**: Allowed only if current status is `CONFIRMED` or `PENDING_PAYMENT` (cannot reschedule `CANCELLED`, `COMPLETED`, or `NO_SHOW` appointments).
- **Time Restriction**:
  - **Patients**: Must submit rescheduling request at least 3 hours before the slot's start time.
  - **Doctors**: Can reschedule any time before the appointment starts.
- **Data Execution**: Execution is wrapped in an atomic transaction:
  - Old slot status is set to `AVAILABLE` and old appointment status is set to `RESCHEDULED`.
  - New slot status is set to `BOOKED` and new appointment is created.
  - Payment credit is mapped to the new appointment ID.
  - Fresh Zoom links are generated for online consultations.

### C. Gemini AI Clinical Analysis Pipeline
Doctors can trigger three Gemini-powered clinical analysis endpoints:
1. **AI Diagnosis Assist (`/ai-diagnosis/`)**:
   - **Input**: Patient demographics, lifestyle profile (sleep, diet, habits), medical history (allergies, medications, chronic conditions), and vault files.
   - **Gemini Engine**: Synthesizes details to return a JSON payload listing potential conditions, confidence levels, recommended tests, and critical risks.
2. **Lab Report Analysis (`/ai-analysis/`)**:
   - **Input**: Text parsed from a specific patient lab report PDF.
   - **Gemini Engine**: Returns a clinical interpretation briefing, actionable next steps, and a triage urgency rating (`Low`, `Medium`, or `High`).
3. **AI Custom Suggestion (`/ai-custom-suggestion/`)**:
   - **Input**: A custom clinical question submitted by the doctor, along with the patient's full medical file context.
   - **Gemini Engine**: Generates a tailored clinical recommendation.

### D. Admin Entity Document Verification Cascade
- Orgs and doctors upload verification files as `EntityDocument` models.
- Admin reviews each document, flagging them as `VERIFIED` or `REJECTED`.
- **Approve Cascade**: If all associated credentials for a doctor/org are marked `VERIFIED`, the entity's status is automatically set to `APPROVED`.
- **Reject Cascade**: If any single document is marked `REJECTED`, the entity status is immediately set to `REJECTED`, locking the profile.
