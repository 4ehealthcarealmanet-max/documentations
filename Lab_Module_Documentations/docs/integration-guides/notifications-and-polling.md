# 🔔 Integration: Notifications & Polling

The PathyaTech Lab Module incorporates a multi-recipient notification system to keep patients, doctors, and lab operators synchronized throughout the booking lifecycle.

---

## 1. Patient Notification Polling (Pull Architecture)

To simplify cross-module communication without requiring incoming webhooks or web socket configurations on the main HealthCare backend, notifications are retrieved using an **outbound polling model**.

```
┌─────────────────────────┐               HTTP GET                ┌─────────────────────────┐
│   HealthCare Frontend   ├──────────────────────────────────────>│   Lab Module Backend    │
│  (Polls every 30-60s)   │   ?patient_id=uuid                    │  (Checks notification)  │
└───────────┬─────────────┘                                       └───────────┬─────────────┘
            │                                                                 │
            │ Display Alert Toast                                             │ Return Unread Alerts
            ▼                                                                 ▼
┌─────────────────────────┐                                       ┌─────────────────────────┐
│   HealthCare Frontend   ├──────────────────────────────────────>│   Lab Module Backend    │
│  (Immediate Acknowledge)│   POST /poll-mark-read/ {patient_id}  │  (Sets is_read = True)  │
└─────────────────────────┘                                       └─────────────────────────┘
```

### 1.1 Poll Endpoint
* **Method**: `GET`
* **Endpoint**: `/api/labs/notifications/poll/`
* **Parameters**: `patient_id` (UUID of the patient)
* **Response**: Returns a JSON array of unread notification objects.

### 1.2 Acknowledge (Mark as Read)
* **Method**: `POST`
* **Endpoint**: `/api/labs/notifications/poll-mark-read/`
* **Payload**:
  ```json
  {
    "patient_id": "f5548c2a-9cb8-42cb-b1b2-1082dcb52304"
  }
  ```

---

## 2. Notification Types & Definitions

The system defines four notification codes:

| Type | Target Audience | Trigger Event |
| :--- | :--- | :--- |
| `BOOKING_CONFIRMED` | Patient | Triggered when booking payment is successfully verified. |
| `COLLECTION_UPDATE` | Patient & Collector | Triggered when a collector changes the sample collection status (e.g. `PICKED`, `IN_TRANSIT`). |
| `REPORT_UPLOADED` | Patient & Doctor | Triggered when a lab operator or admin uploads the PDF results. |
| `BOOKING_CANCELLED` | Patient | Triggered when the booking is cancelled by administrative staff. |

---

## 3. Asynchronous Django Signals & Email Pipelines

Emails are dispatched asynchronously during model state changes using Django's signal framework (`backend/apps/labs/signals.py`).

### 3.1 Signal Listeners
* **`post_save` on `LabBooking`**: Checks if the status changed from `PENDING` to `BOOKED` (fires `BOOKING_CONFIRMED`) or from any state to `CANCELLED` (fires `BOOKING_CANCELLED`).
* **`post_save` on `LabSampleCollection`**: Listens for collector updates and fires email alerts to the patient containing the collector's name and contact information.
* **`post_save` on `LabReport`**: Listens for published PDF uploads and triggers notification emails.

### 3.2 SendGrid Config
SMTP configurations in `settings.py` route all outgoing signals through the SendGrid gateway:
```python
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.sendgrid.net'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = 'apikey'
EMAIL_HOST_PASSWORD = os.environ.get('SENDGRID_API_KEY')
```

---

## 4. SMS & WhatsApp Gateway Hooks (Future Readiness)

The database notification model includes field slots and callback methods to easily attach SMS/WhatsApp notification services (such as Twilio or Interakt).

```python
# Ready-to-use method hooks in apps/labs/models/lab_notification.py
def dispatch_sms(self):
    if not self.booking.patient_phone:
        return
    # TODO: Integrate Twilio API Client
    # client.messages.create(body=self.message, to=self.booking.patient_phone, ...)
    pass
```
To activate SMS alerts, install the `twilio` Python package, configure account credentials in the `.env` file, and call `dispatch_sms()` inside the signal handler functions.
