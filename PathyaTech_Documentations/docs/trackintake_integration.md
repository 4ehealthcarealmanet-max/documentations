# TrackIntake Diet & Biomarker Integration Guide

This document describes how the PathyaTech Healthcare Platform integrates with the external **TrackIntake** system (diet, nutrition, and biomarker tracking service). 

All data created or modified through these APIs is stored in the shared TrackIntake database, allowing patients to view synced health profiles, diet plans, and biomarker reports on the main TrackIntake application.

> [!IMPORTANT]
> **Base URL Configuration:** All endpoints in this guide must be requested against the **TrackIntake Backend Base URL** (e.g., `http://127.0.0.1:8000` or the production TrackIntake server domain), NOT the PathyaTech project's own backend API server URL.

---

## 1. Authentication & Token Flow

All integration endpoints (except **Registration** and **Plan Listing**) require JWT Bearer Authentication. 

```http
Authorization: Bearer <YOUR_ACCESS_TOKEN>
```

Your client application can obtain the access token in two ways:
1. **Upon Signup:** The register endpoint returns `access` and `refresh` tokens directly in the HTTP response on successful creation.
2. **Standard Login:** For returning users, request new tokens via:
   - **Endpoint:** `POST /api/login/`
   - **Payload:** `{ "email": "user@example.com", "password": "securepassword123" }`

---

## 2. API Reference Map

### A. Direct Signup (User Registration)
* **Endpoint:** `POST /api/integration/register/`
* **Access Control:** Public
* **Description:** Registers a new patient without requiring email/phone OTP verification. It initializes a `UserProfile` and links them to the default **Free Plan**.

#### Payload Example:
```json
{
  "email": "patient@example.com",
  "full_name": "John Doe",
  "password": "securepassword123",
  "role": "user",
  "gender": "male",
  "date_of_birth": "1995-08-15",
  "country": "India",
  "city": "Mumbai",
  "mobile_number": "9876543210",
  "height_cm": 175.5,
  "weight_kg": 72.0,
  "activity_level": "Moderately Active",
  "goal": "Lose Weight",
  "diet_type": "Vegetarian",
  "allergies": "Peanuts",
  "is_diabetic": false,
  "is_hypertensive": false,
  "has_heart_condition": false,
  "has_thyroid_disorder": false,
  "has_arthritis": false,
  "has_gastric_issues": false,
  "other_chronic_condition": "None",
  "family_history": "Grandfather had type-2 diabetes"
}
```

#### Response (`201 Created`):
```json
{
  "message": "User registered successfully.",
  "tokens": {
    "refresh": "<JWT_REFRESH_TOKEN>",
    "access": "<JWT_ACCESS_TOKEN>"
  },
  "user": {
    "id": 42,
    "email": "patient@example.com",
    "full_name": "John Doe",
    "role": "user"
  }
}
```

---

### B. List Subscription Plans
* **Endpoint:** `GET /api/integration/plans/`
* **Access Control:** Public
* **Description:** Retrieves all active subscription tiers available for purchase.

#### Response (`200 OK`):
```json
[
  {
    "id": 2,
    "name": "Standard Plan",
    "price": 299.00,
    "duration_days": 30,
    "plan_type": "patient",
    "features": {
      "weight_tracker_allowed": true,
      "nutrition_search_allowed": true,
      "custom_reminder_allowed": true,
      "chat_allowed": true,
      "ai_diet_allowed": true,
      "water_intake_allowed": true
    }
  }
]
```

---

### C. Create Razorpay Payment Order
* **Endpoint:** `POST /api/integration/create-order/`
* **Access Control:** Authenticated (JWT)
* **Description:** Initiates a plan purchase. It generates a Razorpay Order and registers a pending transaction.

#### Payload:
```json
{
  "plan_id": 2
}
```

#### Response (`201 Created`):
```json
{
  "order_id": "order_OkG391klD91sH",
  "amount": 29900,
  "currency": "INR",
  "key": "rzp_test_YourKey",
  "plan": {
    "id": 2,
    "name": "Standard Plan",
    "price": 299
  }
}
```

---

### D. Verify Razorpay Payment Signature
* **Endpoint:** `POST /api/integration/verify-payment/`
* **Access Control:** Authenticated (JWT)
* **Description:** Validates Razorpay's cryptographic response parameters on the server using the API key secret, transitioning the subscription to active status.

#### Payload:
```json
{
  "razorpay_order_id": "order_OkG391klD91sH",
  "razorpay_payment_id": "pay_OkG4n29D29aK",
  "razorpay_signature": "9d81d2938ac...f82810a91f38"
}
```

#### Response (`200 OK`):
```json
{
  "message": "Payment verified and plan activated successfully.",
  "subscription": {
    "id": 15,
    "plan_name": "Standard Plan",
    "start_date": "2026-05-24",
    "end_date": "2026-06-23",
    "is_active": true
  }
}
```

---

### E. Check Active Subscription Plan
* **Endpoint:** `GET /api/integration/check-subscription/`
* **Access Control:** Authenticated (JWT)
* **Description:** Queries if the patient has an active paid subscription plan. Used to gate access to premium diet views.

#### Response (`200 OK`):
```json
{
  "has_active_plan": true,
  "plan": {
    "id": 2,
    "name": "Standard Plan",
    "price": 299,
    "expires_at": "2026-06-23"
  }
}
```

---

### F. Get/Update User Health Profile
* **Endpoints:** 
  - `GET /api/integration/profile/`
  - `PUT /api/integration/profile/` (Full update)
  - `PATCH /api/integration/profile/` (Partial update)
* **Access Control:** Authenticated (JWT)
* **Description:** Manages physical vitals and health indicators (allergies, height, weight, activity level, diabetic/hypertensive status).

---

### G. Chronological Biomarker & Lab Reports
* **Endpoints:**
  - `GET /api/integration/lab-reports/` (List all historical reports)
  - `POST /api/integration/lab-reports/` (Add new report)
  - `GET/PUT/PATCH/DELETE /api/integration/lab-reports/<id>/` (Manage specific report entry)
* **Access Control:** Authenticated (JWT)
* **Description:** Tracks chronological biomarker trends (blood sugar, lipids, HbA1c, TSH, uric acid, vitamins). PDF files are submitted via `multipart/form-data` and uploaded to Cloudinary.

#### Biomarker JSON Schema:
```json
{
  "report_date": "2026-05-24",
  "weight_kg": 72.0,
  "height_cm": 175.5,
  "waist_circumference_cm": 88.0,
  "blood_pressure_systolic": 120,
  "blood_pressure_diastolic": 80,
  "fasting_blood_sugar": 95.0,
  "postprandial_sugar": 135.0,
  "hba1c": 5.6,
  "ldl_cholesterol": 100.0,
  "hdl_cholesterol": 50.0,
  "triglycerides": 140.0,
  "uric_acid": 5.5,
  "creatinine": 0.9,
  "vitamin_d3": 35.0,
  "vitamin_b12": 450.0,
  "tsh": 2.1
}
```

---

### H. Get Certified Active Diet Plan
* **Endpoint:** `GET /api/integration/diet/`
* **Access Control:** Authenticated (JWT)
* **Description:** Fetches the active 7-day personalized diet plan. Requires an active subscription plan.

#### Plan JSON Structure:
```json
{
  "status_code": "APPROVED",
  "message": "Diet plan status: Approved.",
  "plan_data": {
    "id": 105,
    "user_full_name": "John Doe",
    "for_week_starting": "2026-05-24",
    "meals": {
      "Day 1": {
        "Early-Morning": {
          "food_name": "Warm water with lemon",
          "quantity": "250 ml",
          "Gram_Equivalent": 250,
          "Calories": 8,
          "Protein": 0.2,
          "Carbs": 2.5,
          "Fats": 0,
          "Sugar": 0.5,
          "Fiber": 0.3
        },
        "Breakfast": {
          "food_name": "Vegetable Idli with Mint Chutney",
          "quantity": "2 idlis",
          "Gram_Equivalent": 120,
          "Calories": 160,
          "Protein": 4.5,
          "Carbs": 32,
          "Fats": 1.2,
          "Sugar": 1.1,
          "Fiber": 2.4
        },
        "Lunch": { ... },
        "Dinner": { ... }
      }
    },
    "status": "approved",
    "nutritionist_comment": "Focus on high-fiber lunches."
  }
}
```
