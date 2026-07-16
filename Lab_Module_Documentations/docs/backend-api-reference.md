# 🔌 Backend API Reference

This document serves as the complete, authoritative API reference for the PathyaTech Lab Backend.

* **Base URL**: `http://localhost:8000` (development)
* **Default Content-Type**: `application/json`

---

## 🔐 Authorization Headers

1. **Lab Native Users (Admin, Staff, Collector)**:
   ```http
   Authorization: Bearer <lab_auth_token>
   ```
2. **External Users (Patient, Doctor)**:
   ```http
   Authorization: Bearer <healthcare_platform_token>
   ```

---

## 1. Authentication & Staff (`/api/auth/`)

Handles registration, session management, password recovery, and staff invitations.

### 1.1 Sign Up
* **Method**: `POST`
* **Endpoint**: `/api/auth/signup/`
* **Access**: Public
* **Payload**:
  ```json
  {
    "email": "admin@globaldiagnostic.com",
    "password": "SecurePassword123",
    "full_name": "Dr. Ramesh Sharma",
    "phone_number": "9876543210",
    "role": "LAB_ADMIN"
  }
  ```
  *(Valid Roles: `LAB_ADMIN`, `LAB_STAFF`, `LAB_OPERATOR`)*

### 1.2 Login
* **Method**: `POST`
* **Endpoint**: `/api/auth/login/`
* **Access**: Public
* **Response (200 OK)**:
  ```json
  {
    "refresh": "eyJhbGciOi...",
    "access": "eyJhbGciOi...",
    "user": {
      "id": "e44d32a0-bc61-419b-a010-097d9e4a30e1",
      "email": "admin@globaldiagnostic.com",
      "full_name": "Dr. Ramesh Sharma",
      "role": "LAB_ADMIN"
    }
  }
  ```

### 1.3 Get Current User Profile
* **Method**: `GET`
* **Endpoint**: `/api/auth/me/`
* **Access**: Authenticated (JWT)
* **Response (200 OK)**:
  ```json
  {
    "id": "e44d32a0-bc61-419b-a010-097d9e4a30e1",
    "email": "admin@globaldiagnostic.com",
    "full_name": "Dr. Ramesh Sharma",
    "role": "LAB_ADMIN",
    "phone_number": "9876543210",
    "organization": "395d852a-921a-4d32-9cb8-e8cb5be24391"
  }
  ```

### 1.4 Invite Staff
* **Method**: `POST`
* **Endpoint**: `/api/auth/staff/`
* **Access**: Lab Admin
* **Payload**:
  ```json
  {
    "email": "collector.rajesh@globaldiagnostic.com",
    "full_name": "Rajesh Kumar",
    "role": "LAB_OPERATOR",
    "phone_number": "9123456789"
  }
  ```

---

## 2. Lab Profiles (`/api/organizations/`)

Manages laboratory settings, addresses, and licensing metadata.

### 2.1 Get My Organizations
* **Method**: `GET`
* **Endpoint**: `/api/organizations/my-organizations/`
* **Access**: Lab Admin
* **Response (200 OK)**:
  ```json
  [
    {
      "id": "395d852a-921a-4d32-9cb8-e8cb5be24391",
      "name": "Global Diagnostic Labs",
      "type": "LAB",
      "registration_number": "REG-839210",
      "license_number": "LIC-940212",
      "address_line1": "123 Medical Avenue",
      "address_line2": "Opposite General Hospital",
      "city": "Mumbai",
      "state": "Maharashtra",
      "postal_code": "400001",
      "country": "India",
      "contact_email": "info@globaldiagnostic.com",
      "contact_phone": "9876543210",
      "verification_status": "VERIFIED"
    }
  ]
  ```

### 2.2 Update Lab Profile
* **Method**: `PATCH`
* **Endpoint**: `/api/organizations/{org_id}/update/`
* **Access**: Lab Admin
* **Payload**: Any organization fields to update.

---

## 3. Lab Test Catalog (`/api/labs/tests/`)

Manages tests, pricing, prep requirements, and turnaround times.

### 3.1 Create Lab Test
* **Method**: `POST`
* **Endpoint**: `/api/labs/tests/`
* **Access**: Lab Admin
* **Payload**:
  ```json
  {
    "name": "Complete Blood Count (CBC)",
    "category": "Hematology",
    "description": "Measures red blood cells, white blood cells, platelets, and hemoglobin.",
    "price": "500.00",
    "discount_percentage": 10.0,
    "home_collection_available": true,
    "preparation_instructions": "Fasting not required.",
    "turnaround_time": "12 Hours"
  }
  ```

### 3.2 Public Search (Patient discovery)
* **Method**: `GET`
* **Endpoint**: `/api/labs/public-tests/`
* **Access**: Public
* **Query Parameters**:
  * `search`: Matches test/lab names
  * `lab__city`: Filter by city name
  * `category`: Filter by category (exact match)
  * `home_collection_available`: `true` | `false`
  * `ordering`: `price` | `-price` | `created_at`
* **Response (200 OK)**: Paginated results including nested `lab` objects (see `README.md`).

---

## 4. Scheduling & Availability (`/api/labs/slots/`)

Manages operating hours and generates slots.

### 4.1 Auto-Generate Slots
* **Method**: `POST`
* **Endpoint**: `/api/labs/slots/generate-from-availability/`
* **Access**: Lab Admin
* **Payload**:
  ```json
  {
    "start_date": "2026-06-01",
    "end_date": "2026-06-07",
    "interval_minutes": 30,
    "capacity": 5
  }
  ```

### 4.2 Fetch Available Slots (Patient discovery)
* **Method**: `GET`
* **Endpoint**: `/api/labs/public-slots/`
* **Access**: Public
* **Query Parameters**:
  * `lab` (Required): UUID of the lab
  * `date` (Required): YYYY-MM-DD
* **Response (200 OK)**: Returns list of available slot IDs, hours, capacity, and current booking counts.

---

## 5. Cart Management (`/api/labs/cart/`)

A temporary booking staging area for patients.

### 5.1 View Cart Items
* **Method**: `GET`
* **Endpoint**: `/api/labs/cart/`
* **Access**: Patient (Cross-Module JWT)

### 5.2 Add to Cart
* **Method**: `POST`
* **Endpoint**: `/api/labs/cart/`
* **Access**: Patient (Cross-Module JWT)
* **Payload**:
  ```json
  {
    "lab_test": "304b7ac8-3b91-4cf1-a083-059adbf1b142"
  }
  ```

### 5.3 Update Item Schedule
* **Method**: `PATCH`
* **Endpoint**: `/api/labs/cart/{item_id}/`
* **Access**: Patient (Cross-Module JWT)
* **Payload**:
  ```json
  {
    "collection_type": "HOME",
    "appointment_date": "2026-06-03",
    "custom_home_slot": "92da104b-3cb8-4e89-9a28-ee1bcde24021"
  }
  ```

---

## 6. Bookings & Checkout (`/api/labs/user-bookings/`)

Processes checkout, verifies payments, and changes logistics statuses.

### 6.1 Cart Checkout
* **Method**: `POST`
* **Endpoint**: `/api/labs/user-bookings/checkout/`
* **Access**: Patient (Cross-Module JWT)
* **Payload**:
  ```json
  {
    "patient_name": "Shivam Likhar",
    "patient_phone": "9876543210",
    "patient_email": "shivam@example.com",
    "patient_age": 25,
    "patient_gender": "Male",
    "patient_address": "Flat 302, Green Meadows, Andheri West, Mumbai"
  }
  ```
* **Response (201 Created)**:
  ```json
  {
    "booking_id": "f5548c2a-9cb8-42cb-b1b2-1082dcb52304",
    "razorpay_order_id": "order_OkW19aHlPa92la",
    "amount": 450.00,
    "currency": "INR"
  }
  ```

### 6.2 Verify Razorpay Payment
* **Method**: `POST`
* **Endpoint**: `/api/labs/user-bookings/{booking_id}/verify-payment/`
* **Access**: Patient (Cross-Module JWT)
* **Payload**:
  ```json
  {
    "razorpay_payment_id": "pay_OkW8la923asPkl",
    "razorpay_order_id": "order_OkW19aHlPa92la",
    "razorpay_signature": "810a9cb842da34e9cf85289cb79e0a0d9b4b72648ec9..."
  }
  ```
* **Response (200 OK)**:
  ```json
  {
    "status": "Payment verified and booking confirmed",
    "booking_status": "BOOKED"
  }
  ```

### 6.3 Assign Collector (Admin Operation)
* **Method**: `PATCH`
* **Endpoint**: `/api/labs/admin-bookings/{booking_id}/update-collection/`
* **Access**: Lab Admin
* **Payload**:
  ```json
  {
    "collector": "d98ab34c-12bc-4401-9a99-b1dca21d0191",
    "status": "SCHEDULED",
    "scheduled_at": "2026-06-03T08:30:00Z"
  }
  ```

### 6.4 Upload Diagnostic Report (PDF)
* **Method**: `POST`
* **Endpoint**: `/api/labs/admin-bookings/{booking_id}/upload-report/`
* **Access**: Lab Admin / Staff
* **Headers**: `Content-Type: multipart/form-data`
* **Multipart Fields**:
  * `report_file`: Binary PDF / Image
  * `lab_test_id` (Optional): UUID of a specific test inside the booking.

---

## 7. Doctor Portal APIs (`/api/labs/doctor-reports/`)

Allows referred doctors to view reports and request AI-powered analysis.

### 7.1 List Shared Reports
* **Method**: `GET`
* **Endpoint**: `/api/labs/doctor-reports/`
* **Access**: Doctor (Cross-Module JWT)
* **Response (200 OK)**:
  ```json
  [
    {
      "id": "78da12b9-3cba-4efb-a012-decf9108b3c1",
      "test_name": "Complete Blood Count (CBC)",
      "patient_name": "Shivam Likhar",
      "report_url": "https://res.cloudinary.com/pathya/image/upload/v12345/reports/cbc.pdf",
      "status": "PUBLISHED",
      "uploaded_at": "2026-06-03T15:20:00Z"
    }
  ]
  ```

### 7.2 Request AI Clinical Interpretation
* **Method**: `POST`
* **Endpoint**: `/api/labs/doctor-reports/{report_id}/ai-analysis/`
* **Access**: Doctor (Cross-Module JWT)
* **Response (200 OK)**:
  ```json
  {
    "interpretation": "Hemoglobin counts (14.2 g/dL) and white blood cell levels are within normal physiological bounds. No acute infections detected.",
    "suggestions": [
      "Maintain active hydration",
      "Follow up checkup in 6 months"
    ],
    "urgency": "LOW"
  }
  ```

---

## 8. Healthcare Collaboration (`/api/collaboration/`)

* **Base URL**: `NEXT_PUBLIC_CONSULTATION_URL` (HealthCare core backend, Port **9000**).
* **Access**: Lab Admin (JWT) / Doctor (Cross-Module JWT).

### 8.1 Create Collaboration Post
* **Method**: `POST`
* **Endpoint**: `/api/collaboration/posts/`
* **Payload**:
  ```json
  {
    "organization_id": "395d852a-921a-4d32-9cb8-e8cb5be24391",
    "organization_type": "LAB",
    "organization_name": "Global Diagnostic Labs",
    "title": "Onboarding Cardiologist for Cardiac Camp",
    "description": "Seeking a verified cardiologist for diagnostic interpretation during our upcoming weekend camp.",
    "specialization_required": "Cardiology",
    "consultation_mode": "OFFLINE",
    "location": "Sector 4, Mumbai",
    "urgency_level": "HIGH",
    "start_datetime": "2026-06-15T09:00:00Z",
    "end_datetime": "2026-06-16T18:00:00Z"
  }
  ```

### 8.2 Send Message
* **Method**: `POST`
* **Endpoint**: `/api/collaboration/posts/{post_id}/message/`
* **Payload**:
  ```json
  {
    "doctor_id": "19b8cbda-4ca1-409c-ba89-ea21bebc901a",
    "message": "Hello Dr. Mehta, we reviewed your profile and would love to collaborate."
  }
  ```

### 8.3 Get Messages
* **Method**: `GET`
* **Endpoint**: `/api/collaboration/messages/`
* **Query Parameters**:
  * `conversation_id`: UUID
