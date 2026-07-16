# 🛒 Integration: Cart & Checkout

Diagnostic tests differ from retail products: they require physical appointments, preparation instructions (e.g., fasting), and slot scheduling. This document outlines the logic behind the multi-appointment checkout pipeline.

---

## 1. Scheduling at the Cart

Every item added to the cart (`/api/labs/cart/`) must have its collection type and timing configured before a checkout can be initiated.

```
       [ Add Test to Cart ] ────> Default: LAB_VISIT (No Date)
                 │
                 ▼
     [ User Configures Item ] ──> Selects: HOME or LAB_VISIT
                 │
                 ├───────────────────────────────┐
                 ▼                               ▼
      [ If HOME Collection ]           [ If LAB_VISIT ]
      Select Appointment Date          Select Appointment Date
      Select Custom Home Slot          Select Lab Slot (TimeSlot)
                 │                               │
                 └───────────────┬───────────────┘
                                 ▼
                     [ PATCH /api/labs/cart/{id}/ ]
```

### Required Fields for Staging
* **`collection_type`**: `HOME` or `LAB_VISIT`
* **`appointment_date`**: YYYY-MM-DD
* **`slot`**: UUID (Only if `collection_type` is `LAB_VISIT`)
* **`custom_home_slot`**: UUID (Only if `collection_type` is `HOME`)

---

## 2. Order Grouping & Splitting Logic

When checking out, the backend processes all items in the patient's active cart. Since a patient may book different tests from various laboratories or configure different schedule timings, the system performs **sub-ordering**.

### Splitting Criteria
Cart items are grouped into a single **`LabBooking` (Sub-Order)** if and only if they share:
1. The same **`LabOrganization`** (Laboratory)
2. The same **`collection_type`** (`HOME` vs `LAB_VISIT`)
3. The same **`appointment_date`**
4. The same **`slot`** or **`custom_home_slot`**

### Concrete Example Scenario
A user adds three tests to their cart:
1. **CBC Test** — Lab Alpha, `HOME` Collection, Date: `2026-06-03`, Time: `8-10 AM`
2. **Kidney Profile** — Lab Alpha, `HOME` Collection, Date: `2026-06-03`, Time: `8-10 AM`
3. **Chest X-Ray** — Lab Alpha, `LAB_VISIT` Collection, Date: `2026-06-04`, Time: `11:00 AM Slot`

#### Backend Execution on Checkout:
- **`LabMasterOrder`**: Created for the sum of all tests.
- **`LabBooking A`**: Linked to `LabMasterOrder`. Contains **CBC Test** & **Kidney Profile** (Scheduled for Home collection on June 3rd).
- **`LabBooking B`**: Linked to `LabMasterOrder`. Contains **Chest X-Ray** (Scheduled for Lab Visit on June 4th).

---

## 3. The Razorpay Checkout Pipeline

```
   Frontend                      Lab Backend                     Razorpay API
      │                               │                                │
      │ 1. POST /checkout/            │                                │
      ├──────────────────────────────>│                                │
      │                               │ 2. Create Order & Sub-Bookings │
      │                               │ 3. POST /orders                │
      │                               ├───────────────────────────────>│
      │                               │<───────────────────────────────┤
      │                               │    [ Returns razorpay_order_id ]
      │ 4. Send Order Info            │                                │
      │<──────────────────────────────┤                                │
      │                               │                                │
      │ 5. Render Payment UI Overlay  │                                │
      │ 6. Complete Card Payment      │                                │
      ├───────────────────────────────┼───────────────────────────────>│
      │<──────────────────────────────┼────────────────────────────────┤
      │    [ Returns Payment Credentials: ID + Signature ]             │
      │                               │                                │
      │ 7. POST /verify-payment/      │                                │
      ├──────────────────────────────>│                                │
      │                               │ 8. Cryptographically Validate  │
      │                               │ 9. Transition State to PAID    │
      │ 10. Success Response          │                                │
      │<──────────────────────────────┤                                │
```

### Verification Cryptography
To prevent client-side payment bypasses, verification is computed on the backend using the SHA-256 HMAC algorithm:

$$\text{Signature} = \text{HMAC-SHA256}(\text{razorpay\_order\_id} + "|" + \text{razorpay\_payment\_id}, \text{RAZORPAY\_KEY\_SECRET})$$

If the computed signature matches the header `razorpay_signature` sent by the client, the transaction is marked as complete.
