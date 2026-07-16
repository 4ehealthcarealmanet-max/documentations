# Lab Module & Diagnostics API Reference

This document serves as the developer guide and API reference manual for the **Standalone Lab Diagnostics Module**. It outlines public test discovery, slot booking rules, cart orchestrations, payment checkouts, phlebotomist logistics, and doctor report review/AI analysis integration.

---

## 1. Authentication & Base URL
- **Base URL:** `http://localhost:8000` (or production domain)
- **Authentication:** All client requests are authenticated using a **JWT access token issued by the main PathyaTech platform**.
- **Auth Header:**
  ```http
  Authorization: Bearer <token>
  ```
- **Note:** These endpoints do *not* use any separate credentials; they authorize using the unified JWT token.

---

## 2. Patient API Endpoint Directory

### A. Public Discovery (Unauthenticated)

#### 1. Browse Lab Tests
* **Endpoint:** `GET /api/labs/public-tests/`
* **Description:** Search and filter tests across verified laboratories.
* **Query Parameters:**
  - `search` (string): Search by test name, lab name, or category. E.g., `?search=Glucose`.
  - `lab__city` (string): Filter tests by lab location city. E.g., `?lab__city=Mumbai`.
  - `category` (string): Filter by category. E.g., `?category=Blood Test`.
  - `home_collection_available` (bool): Filter by home sample collection. E.g., `?home_collection_available=true`.
  - `ordering` (string): Sort fields (`price`, `-price`, `created_at`). E.g., `?ordering=price`.
  - `page` (int): Navigation page number. E.g., `?page=2`.

##### Response Schema:
```json
{
  "count": 45,
  "next": "...",
  "results": [
    {
      "id": "7a3e-uuid",
      "name": "Full Body Checkup",
      "category": "Comprehensive",
      "price": "2500.00",
      "final_price": 2250.00,
      "home_collection_available": true,
      "turnaround_time": "24 Hours",
      "lab": {
        "id": "9b1c-uuid",
        "name": "Metropolis Diagnostic Center",
        "city": "Delhi",
        "verification_status": "VERIFIED"
      }
    }
  ]
}
```

#### 2. Get Single Test
* **Endpoint:** `GET /api/labs/public-tests/{id}/`
* **Description:** Returns complete details for a test, including the full address of the laboratory.

#### 3. View Available slots for a Lab
* **Endpoint:** `GET /api/labs/public-slots/?lab={lab_uuid}&date={YYYY-MM-DD}`
* **Description:** Returns time slots for patients opting for `LAB_VISIT` collection.

##### Response Schema:
```json
[
  {
    "id": "uuid-of-slot",
    "date": "2026-04-10",
    "start_time": "09:00:00",
    "end_time": "09:30:00",
    "max_capacity": 5,
    "current_bookings": 2,
    "is_full": false
  }
]
```

---

### B. Lab Shopping Cart (JWT Required)

#### 4. View Cart
* **Endpoint:** `GET /api/labs/cart/`
* **Description:** Retrieves all lab tests currently in the patient's active cart.

#### 5. Add Test to Cart
* **Endpoint:** `POST /api/labs/cart/`
* **Payload:** `{ "lab_test": "uuid-of-the-test" }`
* **Constraint:** Duplicates are prevented at the database level using unique constraint checks.

#### 6. Remove Test from Cart
* **Endpoint:** `DELETE /api/labs/cart/{id}/`

---

### C. Booking & Payment Verification (JWT Required)

#### 7. Checkout Cart
* **Endpoint:** `POST /api/labs/user-bookings/checkout/`
* **Description:** Converts cart items from a specific lab into a pending `LabBooking` transaction, returns a Razorpay order ID, and clears checked-out cart items.

##### Payload Example:
```json
{
  "lab": "uuid-of-laboratory",
  "collection_type": "HOME",
  "preferred_time": "8-10 AM",
  "appointment_date": "2026-04-10",
  "patient_name": "John Doe",
  "patient_age": 30,
  "patient_gender": "Male",
  "referred_by_doctor": "Dr. Anjali Mehta",
  "appointment_id": "uuid-of-consultation",
  "recommendation_id": "uuid-of-recommendation"
}
```

##### Response Schema:
```json
{
  "booking_id": "uuid-of-booking",
  "razorpay_order_id": "order_XXXXXX",
  "amount": 1500.00,
  "currency": "INR"
}
```

#### 8. Direct Booking (Buy Now)
* **Endpoint:** `POST /api/labs/user-bookings/direct-book/`
* **Description:** Bypasses the cart to book one or multiple tests from a specific lab instantly. Same payload and response schema as Checkout.

#### 9. Verify Razorpay Payment
* **Endpoint:** `POST /api/labs/user-bookings/{booking_id}/verify-payment/`
* **Description:** Receives Razorpay checkout parameters, verifies the cryptographic signature on the server, and transitions the booking from `PENDING` to `BOOKED`.
* **Payload:**
  ```json
  {
    "razorpay_payment_id": "pay_XXXXXX",
    "razorpay_order_id": "order_XXXXXX",
    "razorpay_signature": "signature_hash"
  }
  ```
* **Integration Action:** If `recommendation_id` is linked, the system automatically PATCHes the doctor's recommended prescription test record as `is_booked = True`.

---

## 3. Phlebotomist Tracking & Logistics

For patients booking `HOME` collection, the system assigns a phlebotomist and logs live status updates. Patients can retrieve these in `GET /api/labs/user-bookings/`:

- **Logistics Status Flow:** `SCHEDULED` $\rightarrow$ `PICKED` $\rightarrow$ `IN_TRANSIT`.
- **Response Details:** Includes the assigned collector's name and the scheduled collection slot:
  ```json
  "collection_details": [
    {
      "id": "uuid-of-ticket",
      "status": "IN_TRANSIT",
      "collector_name": "Rajesh Kumar",
      "scheduled_at": "2026-04-10T08:30:00Z"
    }
  ]
  ```

---

## 4. Doctor Referral & Report AI Analysis (JWT Required)

> [!NOTE]
> These endpoints check that the user's role is `DOCTOR`. Doctors can only access reports for patients they referred (meaning `referred_by_doctor` matches their profile name).

### 1. List Patient Reports
* **Endpoint:** `GET /api/labs/doctor-reports/`
* **Query Parameters:** `?patient_id={uuid}` (Filter to a specific patient).
* **Response:** Returns list of `PUBLISHED` reports, providing PDF URLs.

### 2. Run Clinical AI Analysis
* **Endpoint:** `POST /api/labs/doctor-reports/{id}/ai-analysis/`
* **Description:** Triggers Google Gemini AI to analyze biomarkers in the report PDF. Returns a clinical summary and triage urgency rating.

##### Response Schema:
```json
{
  "interpretation": "Hemoglobin level is slightly low (11.8 g/dL), indicating mild anemia.",
  "suggestions": [
    "Recommend iron-rich dietary changes.",
    "Order follow-up Complete Blood Count (CBC) in 4 weeks."
  ],
  "urgency": "Low"
}
```
