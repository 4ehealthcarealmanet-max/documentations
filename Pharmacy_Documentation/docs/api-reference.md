# Pharmacy Module - API Reference

This document maps all the API endpoints available in the Pharmacy module.

## 1. Authentication APIs

Base path: `/api/auth/`

### 1.1 Signup
* **Endpoint**: `POST /api/auth/signup/`
* **Access**: Public
* **Payload**:
  ```json
  {
    "email": "admin@example.com",
    "password": "SecurePassword123",
    "name": "John Doe",
    "phone": "+1234567890",
    "role": "PHARMACY_ADMIN"
  }
  ```
* **Responses**:
  * `201 Created`: Signup successful. OTP sent.
  * `200 OK`: Account already exists but inactive. OTP resent.
  * `400 Bad Request`: Validation errors.

### 1.2 Verify OTP (Account Activation)
* **Endpoint**: `POST /api/auth/verify-otp/`
* **Access**: Public
* **Payload**:
  ```json
  {
    "email": "admin@example.com",
    "otp": "123456"
  }
  ```
* **Responses**:
  * `200 OK`: OTP verified successfully. If the role is `PHARMACY_ADMIN`, a new organization is bootstrapped automatically and linked.
  * `400 Bad Request`: Invalid or expired OTP.

### 1.3 Login
* **Endpoint**: `POST /api/auth/login/`
* **Access**: Public
* **Payload**:
  ```json
  {
    "email": "admin@example.com",
    "password": "SecurePassword123",
    "role": "PHARMACY_ADMIN"
  }
  ```
* **Responses**:
  * `200 OK`: Returns JWT tokens and active membership.
    ```json
    {
      "access": "eyJhbG...",
      "refresh": "eyJhbG...",
      "id": "uuid-string",
      "role": "PHARMACY_ADMIN",
      "email": "admin@example.com",
      "name": "John Doe",
      "organization": {
        "id": "org-uuid-string",
        "name": "John Doe's Pharmacy",
        "type": "PHARMACY"
      }
    }
    ```
  * `401 Unauthorized`: Invalid credentials.
  * `403 Forbidden`: Account not verified or role mismatch.

### 1.4 Get/Update Current User Info
* **Endpoint**: `GET` / `PATCH` `/api/auth/me/`
* **Access**: Authenticated (JWT Bearer)
* **PATCH Payload**:
  ```json
  {
    "name": "John Updated",
    "phone": "+9876543210"
  }
  ```
* **Responses**:
  * `200 OK`: Returns user profile data and membership details.

### 1.5 Password Recovery
* **Request Reset OTP**: `POST /api/auth/send-otp/`
  * Payload: `{"email": "admin@example.com"}`
* **Reset Password**: `POST /api/auth/reset-password/`
  * Payload: `{"email": "user@example.com", "otp": "123456", "new_password": "NewSecurePassword123"}`

### 1.6 Logout
* **Endpoint**: `POST /api/auth/logout/`
* **Access**: Authenticated
* **Payload**: `{"refresh": "refresh_token_here"}`

---

## 2. Organization APIs

Base path: `/api/organizations/`

### 2.1 List Organizations
* **Endpoint**: `GET /api/organizations/`
* **Access**: Public / Authenticated
* **Query Parameters**: `city`, `state`, `type`, `verification_status`
* **Response**: List of organizations.

### 2.2 User Organizations
* **Endpoint**: `GET /api/organizations/my-organizations/`
* **Access**: Authenticated
* **Response**: List of organizations the active user is a member of.

### 2.3 Update Organization
* **Endpoint**: `PATCH /api/organizations/<uuid:org_id>/update/`
* **Access**: Authenticated (Admin role membership required)
* **Payload**:
  ```json
  {
    "name": "Healthy Pharmacy",
    "address_line1": "123 Main St",
    "city": "Boston",
    "state": "MA",
    "postal_code": "02108"
  }
  ```

---

## 3. Documents & Media APIs

Base path: `/api/documents/`

### 3.1 Upload Entity Document
* **Endpoint**: `POST /api/documents/entity/upload/`
* **Access**: Authenticated
* **Format**: `multipart/form-data`
* **Payload**:
  * `file`: (Binary File)
  * `document_type`: `LICENSE` | `REGISTRATION` | `GALLERY` | `COMPLIANCE`
  * `app_label`: `organizations` | `pharmacys`
  * `model`: `pharmacyorganization` | `pharmacymedicine`
  * `object_id`: Target object's UUID
* **Response**: `201 Created` with document details.

### 3.2 Verify Document
* **Endpoint**: `PATCH /api/documents/entity/<uuid:pk>/verify/`
* **Access**: Authenticated (Platform Admin Only)
* **Payload**:
  ```json
  {
    "verification_status": "VERIFIED",
    "rejection_reason": ""
  }
  ```

---

## 4. Catalog & Booking APIs (Pharmacys)

Base path: `/api/pharmacys/`

### 4.1 Medicines Catalog
* **Manage Catalog (Internal)**: `/api/pharmacys/medicines/` (GET, POST, PUT, DELETE)
* **Public Search (Patients)**: `/api/pharmacys/public-medicines/` (GET)
  * Query parameters: `search`, `category`, `pharmacy_id`, `ordering`

### 4.2 Booking and Orders
* **Cart Operations**: `/api/pharmacys/cart/` (GET, POST, DELETE)
  * Create Payload:
    ```json
    {
      "pharmacy_medicine": "medicine-uuid",
      "collection_type": "HOME",
      "appointment_date": "2026-07-20",
      "custom_home_slot": "slot-uuid"
    }
    ```
* **Patient Sub-Orders**: `/api/pharmacys/user-orders/`
  * View booked sub-orders and status.
* **Admin Sub-Orders**: `/api/pharmacys/admin-orders/`
  * Update sub-order status (`BOOKED`, `COMPLETED`, `CANCELLED`).

---

## 5. Payments APIs

Base path: `/api/payments/`

### 5.1 Initiate Payment
* **Endpoint**: `POST /api/payments/initiate/`
* **Access**: Authenticated
* **Payload**:
  ```json
  {
    "reference_type": "PHARMACY_ORDER",
    "reference_id": "master-order-uuid",
    "amount": 1500.00,
    "currency": "INR"
  }
  ```
* **Response**: `201 Created` returning Razorpay order object details.

### 5.2 Verify Payment
* **Endpoint**: `POST /api/payments/verify/`
* **Access**: Authenticated
* **Payload**:
  ```json
  {
    "razorpay_order_id": "order_xyz123",
    "razorpay_payment_id": "pay_abc789",
    "razorpay_signature": "signature_hash_here"
  }
  ```
* **Response**: `200 OK` on successful validation.
