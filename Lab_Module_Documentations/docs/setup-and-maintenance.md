# 🛠️ Setup & Maintenance Guide

Follow this guide to configure, run, and maintain the PathyaTech Lab Module.

---

## 🐍 Backend Configuration & Deployment

### 1. Prerequisites
- **Python**: Version 3.11 or 3.12
- **Virtual Environment Tool**: `venv` or `conda`
- **Database**: PostgreSQL (Production) or SQLite (Local Development)

### 2. Quickstart Installation
Navigate to the `backend` folder and run the following:

```bash
# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # On Windows use `venv\Scripts\activate`

# Install required modules
pip install -r requirements.txt
```

### 3. Environment Configuration (`.env`)
Create a file named `.env` in the `backend/` directory. Refer to the table below for configuration details:

| Variable | Description | Example / Default |
| :--- | :--- | :--- |
| `ALLOWED_HOSTS` | Array of domains authorized to reach the backend | `*` |
| `CLOUDINARY_KEY` | Cloudinary API Key | `your_cloudinary_key` |
| `CLOUDINARY_NAME` | Cloudinary Cloud Name | `your_cloudinary_name` |
| `CLOUDINARY_SECRET` | Cloudinary API Secret | `your_cloudinary_secret` |
| `CLOUDINARY_URL` | Cloudinary Connection URL | `cloudinary://your_cloudinary_key:your_cloudinary_secret@your_cloudinary_name` |
| `CORS_ALLOWED_ORIGINS` | Allowed cross-origin sites | `https://your-main-frontend.vercel.app,http://localhost:3000` |
| `DB_HOST` | Database host | `your_db_host` |
| `DB_NAME` | Database name | `postgres` |
| `DB_USER` | Database user | `your_db_user` |
| `DB_PASSWORD` | Database password | `your_db_password` |
| `DB_PORT` | Database port | `5432` |
| `DEBUG` | Toggle debug verbose modes | `True` |
| `DEFAULT_FROM_EMAIL` | Default sender address | `notifications@yourdomain.com` |
| `EMAIL_HOST` | SMTP server host | `smtp.gmail.com` |
| `EMAIL_HOST_USER` | SMTP server user | `your_email_host_user@gmail.com` |
| `EMAIL_HOST_PASSWORD` | SMTP server password | `your_email_host_password` |
| `EMAIL_PORT` | SMTP port | `587` |
| `EMAIL_USE_TLS` | SMTP TLS toggle | `True` |
| `FRONTEND_URL` | URL of the frontend client | `http://localhost:3000` |
| `PLATFORM_RAZORPAY_KEY_ID` | Platform Razorpay Key ID | `rzp_test_yourkeyid` |
| `PLATFORM_RAZORPAY_KEY_SECRET` | Platform Razorpay Key Secret | `your_razorpay_key_secret` |
| `PLATFORM_RAZORPAY_WEBHOOK_SECRET` | Platform Razorpay Webhook Secret | `your_razorpay_webhook_secret` |
| `RAZORPAY_KEY_ID` | Lab Razorpay Key ID | `rzp_test_yourkeyid` |
| `RAZORPAY_KEY_SECRET` | Lab Razorpay Key Secret | `your_razorpay_key_secret` |
| `RAZORPAY_WEBHOOK_SECRET` | Lab Razorpay Webhook Secret | `your_razorpay_webhook_secret` |
| `SECRET_KEY` | Django cryptographic secret key | `your_django_secret_key` |
| `SENDGRID_API_KEY` | SendGrid SMTP key | `SG.your_sendgrid_api_key` |
| `ZOOM_ACCOUNT_ID` | Zoom Account ID | `your_zoom_account_id` |
| `ZOOM_CLIENT_ID` | Zoom Client ID | `your_zoom_client_id` |
| `ZOOM_CLIENT_SECRET` | Zoom Client Secret | `your_zoom_client_secret` |
| `ZOOM_SECRET_TOKEN` | Zoom Secret Token | `your_zoom_secret_token` |

---

### 4. Database Migrations & Administration

Apply database changes:
```bash
python manage.py makemigrations
python manage.py migrate
```

Verify migration status using the built-in checker script:
```bash
python check_migrations.py
```

Create a superuser account for the Django Administration panel (`/admin`):
```bash
python manage.py createsuperuser
```

### 5. Running the Service
```bash
# Run local development server
python manage.py runserver 0.0.0.0:8000
```

---

## ⚡ Frontend Configuration & Run

### 1. Prerequisites
- **Node.js**: Version 18.x or 20.x
- **Package Manager**: `npm` (included with Node)

### 2. Quickstart Installation
Navigate to the `frontend` folder and run:

```bash
# Install package dependencies
npm install
```

### 3. Environment Variables (`.env.local`)
Create a file named `.env.local` in the `frontend/` directory:

```env
# URL for HealthCare core/consultation backend
NEXT_PUBLIC_CONSULTATION_URL=https://your-main-platform.com

# URL for this Lab Backend Microservice
NEXT_PUBLIC_BACKEND_URL=https://your-lab-backend.com

# Payment Gateway (Razorpay)
NEXT_PUBLIC_RAZORPAY_KEY_ID=rzp_test_yourkeyid

# Zoom Meeting integration
NEXT_PUBLIC_ZOOM_CLIENT_SECRET=your_zoom_client_secret
```

> [!IMPORTANT]
> The HealthCare backend runs on port **9000**. Ensure the `NEXT_PUBLIC_CONSULTATION_URL` is set correctly; otherwise, the Collaboration dashboard will return network errors.

### 4. Running the App

```bash
# Start Next.js development server
npm run dev
```
Open [http://localhost:3000/lab](http://localhost:3000/lab) in your browser.

```bash
# Build production bundles
npm run build

# Start production server
npm run start
```

---

## 🧹 Maintenance & Troubleshooting

### Clearing Next.js Caches
If Next.js experiences compilation errors or caching conflicts, clear the cache folders:
```bash
rm -rf .next node_modules/.cache
npm run dev
```

### Razorpay Webhook Management
Ensure your server exposes an endpoint to capture events from Razorpay (e.g. `POST /api/payments/webhook/razorpay/`). For local testing, use a tunneling tool like **ngrok**:
```bash
ngrok http 8000
```
Update the webhook URL in the Razorpay dashboard with your ngrok URL.
