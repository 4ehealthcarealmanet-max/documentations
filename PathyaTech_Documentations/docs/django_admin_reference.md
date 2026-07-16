# PathyaTech Django Admin Panel Developer Reference

This document provides a comprehensive technical reference for the Django Admin portal (`/admin/`) configuration. The Django Admin acts as the primary administrative panel for database audits, record-level management, bulk processing, and verification checks.

---

## 1. User & Authentication Administration (`apps/auth_app/admin.py`)

### A. Model: `User`
- **List Columns**: `email`, `name`, `phone`, `role`, `status`, `is_staff`, `created_at`
- **List Filters**: `role`, `status`, `is_staff`, `is_superuser`, `is_active`
- **Search Fields**: `email`, `name`, `phone`
- **Default Sorting**: Newest users first (`-created_at`)
- **Fieldsets**: Grouped into credentials, personal info, permissions, and important timestamps.
- **Read-Only Fields**: `id`, `created_at`, `updated_at`, `last_login`
- **Embedded Inline Panels**:
  - `DoctorProfileInline`: View and edit doctor-specific configurations directly from the User detail view.
  - `PatientProfileInline`: Manage patient details.
  - `PatientMedicalProfileInline`: Manage chronic diseases, medications, and surgical history.
  - `PatientLifestyleProfileInline`: Manage food preferences, smoking, and alcohol habits.
  - `OrganizationMembershipInline`: Relate the user to affiliated hospitals, clinics, labs, or pharmacies.
  - `DoctorStaffInline`: Check receptionist staff assigned to this doctor.
  - `UserDoctorAssignmentsInline`: Track doctor links for staff roles.
- **Bulk Actions**:
  - `Mark selected users as Active`: Transitions user status to `ACTIVE`.
  - `Block selected users`: Transitions user status to `BLOCKED` (restricting token authentication).

### B. Model: `OrganizationMembership`
- **List Columns**: `user`, `organization`, `role`, `is_active`, `joined_at`
- **List Filters**: `role`, `is_active`
- **Search Fields**: `user__email`, `organization__name`

### C. Model: `DoctorAssignment`
- **List Columns**: `user`, `doctor`, `role`, `organization`, `is_active`, `created_at`
- **List Filters**: `role`, `is_active`
- **Search Fields**: `user__email`, `doctor__email`, `organization__name`

### D. Model: `OTP`
- **List Columns**: `user`, `otp`, `is_verified`, `created_at`
- **List Filters**: `is_verified`, `created_at`
- **Search Fields**: `user__email`, `user__name`, `otp`
- **Read-Only Enforcements**: All fields are read-only; the `add_permission` and `delete_permission` are disabled to prevent manual tampering.

---

## 2. Doctor Management Administration (`apps/doctors/admin.py`)

### A. Model: `DoctorProfile`
- **List Columns**: `name`, `specialization`, `experience_years`, `verification_status`, `reel_verification_status`, `rating_average`, `license_number`, `adhaar_number`
- **List Filters**: `verification_status`, `reel_verification_status`, `specialization`
- **Search Fields**: `name`, `user__email`, `specialization`, `license_number`, `adhaar_number`
- **Default Sorting**: Highest average rating first (`-rating_average`)
- **Fieldsets**:
  - *Core Info*: `user`, `specialization`, `license_number`, `adhaar_number`, `verification_status`, `rejection_reason`
  - *Professional Details*: `experience_years`, `bio`, `languages_spoken`, `rating_average`, `profile_photo`
  - *Introductory Reels*: `short_reel_url`, `reel_verification_status` with inline video player preview.
- **Embedded Inline Panels**:
  - `DoctorEducationInline`: Academic credentials, degrees, and graduation years.
  - `DoctorExperienceInline`: Past clinics, hospital appointments, and durations.
  - `DoctorOrganizationInline`: Associated healthcare centers.
  - `DoctorAvailabilityInline`: Setup templates for scheduling.
  - `DoctorKeywordInline`: Specialized keywords for platform search optimization.
  - `DoctorBoosterInline`: Clinical case study boosters.
  - `DoctorEntityDocumentInline`: Clickable verification documents with PDF/image preview thumbnails.
- **Bulk Actions**:
  - `Approve selected doctors`: Updates status to `APPROVED` and syncs search listings.
  - `Reject selected doctors`: Updates status to `REJECTED` and disables booking intake.
  - `Approve selected introduction reels`: Approves Cloudinary video reels for public rendering.
  - `Reject selected introduction reels`: Flags reels as `REJECTED` and hides them from profiles.

### B. Model: `DoctorReview`
- **List Columns**: `doctor`, `patient`, `rating`, `created_at`
- **List Filters**: `rating`, `created_at`
- **Search Fields**: `doctor__name`, `patient__name`, `comment`

---

## 3. Patient Management Administration (`apps/patients/admin.py`)

### A. Model: `PatientProfile`
- **List Columns**: `full_name`, `email`, `verification_status`, `gender`, `date_of_birth`, `city`
- **List Filters**: `verification_status`, `gender`, `city`, `blood_group`
- **Search Fields**: `full_name`, `user__email`, `contact_number`
- **Embedded Inline Panels**:
  - `PatientMedicalProfileInline`: Chronic health history.
  - `PatientLifestyleProfileInline`: Dietary and habits history.
- **Bulk Actions**:
  - `Approve selected patients`: Marks verification status as `APPROVED`.
  - `Reject selected patients`: Marks verification status as `REJECTED`.

### B. Model: `LockSlot`
- **List Columns**: `patient`, `availability`, `status`, `created_at`
- **List Filters**: `status`, `created_at`
- **Search Fields**: `patient__email`, `availability__doctor__name`

---

## 4. Appointments & Slots Administration (`apps/appointments/admin.py`)

### A. Model: `Slot`
- **List Columns**: `doctor`, `date`, `start_time`, `status`, `mode`, `fee`
- **List Filters**: `status`, `mode`, `date`, `is_emergency_slot`
- **Search Fields**: `doctor__name`, `date`
- **Date Hierarchy**: `date` for drill-down searches.

### B. Model: `Appointment`
- **List Columns**: `patient_name`, `doctor_name`, `appointment_date`, `status`, `mode`, `created_at`
- **List Filters**: `status`, `mode`, `created_at`
- **Search Fields**: `patient__full_name`, `doctor__name`, `id`
- **Bulk Actions**:
  - `Mark selected as Completed`: Updates appointment status to `COMPLETED` and locks started/finished timestamps.
  - `Cancel selected appointments`: Cancels bookings and sets corresponding slots back to `AVAILABLE`.

---

## 5. Payments & Financial Ledgers (`apps/payments/admin.py`)

### A. Model: `Payment_Table` (Consultations and Labs Ledger)
- **List Columns**: `transaction_id`, `gateway_payment_id`, `payer_email`, `reference_type`, `amount`, `status`, `created_at`
- **List Filters**: `status`, `reference_type`, `payment_method`, `created_at`
- **Search Fields**: `transaction_id`, `gateway_payment_id`, `payer__email`, `reference_id`
- **Permissions**: Read-Only. `has_add_permission` and `has_delete_permission` are disabled to maintain immutable audit ledgers.

### B. Model: `Refund`
- **List Columns**: `appointment`, `refund_amount`, `initiated_by`, `refund_status`, `created_at`
- **List Filters**: `refund_status`, `initiated_by`, `created_at`
- **Bulk Actions**:
  - `Mark selected refunds as SUCCEEDED`: Triggers gateway callback confirmation, transitioning refund status to `SUCCEEDED` and the linked booking to `REFUNDED`.
  - `Mark selected refunds as FAILED`: Marks pending refunds as `FAILED`.

---

## 6. Document Audit Trail (`apps/documents/admin.py`)

### A. Model: `EntityDocument`
- **List Columns**: `file_name`, `document_type`, `verification_status`, `version`, `is_active`, `uploaded_by`, `uploaded_at`
- **List Filters**: `verification_status`, `document_type`, `is_active`
- **Clickable File Previews**: Renders inline clickable `📄 View File` links for PDF attachments and direct image thumbnails (max-height 120px) for image files.
- **Verification Cascade Logic**:
  - When an admin marks an `EntityDocument` as `VERIFIED`, the system checks all verification documents linked to that entity. If all are verified, the parent doctor/organization status is updated to `APPROVED` or `VERIFIED`.
  - If any document is marked `REJECTED`, the parent entity is immediately set to `REJECTED`.
- **Bulk Actions**:
  - `Mark selected as Verified`: Approves the credentials.
  - `Mark selected as Rejected`: Rejects credentials, locking the profile.

### B. Model: `DocumentAuditLog`
- **List Columns**: `document`, `action`, `performed_by`, `performed_at`
- **List Filters**: `action`, `performed_at`
- **Read-Only Enforcement**: Entirely immutable. Manual edits, insertions, or deletions are blocked on this model.

---

## 7. Subscriptions & Patient Plans (`apps/subscriptions/admin.py`)

### A. Model: `PatientSubscriptionPlan`
- **Special Feature**: Contains a custom admin form override that dynamically queries active plans from the TrackIntake API (`/api/integration/plans/?type=patient`) via HTTP. It populates a drop-down field with the remote plan identifiers for mapping, falling back to static options (`Free Plan`, `Standard Plan`, `Premium Plan`) if the API is offline.
