# PathyaTech Local Setup & Installation Guide

This document provides step-by-step instructions for configuring and running the PathyaTech backend and frontend applications locally.

---

## 1. Backend Setup & Run Guide

### A. Prerequisites
- **Python**: Version 3.10 or higher.
- **Pip**: Python package installer.
- **Virtualenv**: Python virtual environment manager.

### B. Installation Steps
1. Navigate to the backend directory:
   ```bash
   cd backend
   ```
2. Create and activate a virtual environment:
   ```bash
   # Create environment
   python -m venv venv

   # Activate on Linux/macOS
   source venv/bin/activate

   # Activate on Windows
   venv\Scripts\activate
   ```
3. Install backend dependencies:
   ```bash
   pip install -r requirements.txt
   ```

### C. Environment Variables
Create a file named `.env` in the `backend/` directory:
```env
# =========================
# CORE / DJANGO SETTINGS
# =========================
DEBUG=True
SECRET_KEY=your-django-secret-key-here
FRONTEND_URL=http://localhost:3000
ALLOWED_HOSTS=localhost,127.0.0.1
CORS_ALLOWED_ORIGINS=http://localhost:3000,http://127.0.0.1:3000

# =========================
# DATABASE (SUPABASE POSTGRES)
# =========================
DB_NAME=postgres
DB_USER=your-db-user
DB_PASSWORD=your-db-password
DB_HOST=your-db-host
DB_PORT=6543

# =========================
# PAYMENTS (RAZORPAY)
# =========================
RAZORPAY_KEY_ID=your-razorpay-key-id
RAZORPAY_KEY_SECRET=your-razorpay-key-secret
RAZORPAY_WEBHOOK_SECRET=your-razorpay-webhook-secret

# =========================
# CLOUD STORAGE (CLOUDINARY)
# =========================
CLOUDINARY_NAME=your-cloudinary-name
CLOUDINARY_KEY=your-cloudinary-key
CLOUDINARY_SECRET=your-cloudinary-secret
CLOUDINARY_URL=your-cloudinary-url

# =========================
# COMMUNICATION (ZOOM)
# =========================
ZOOM_ACCOUNT_ID=your-zoom-account-id
ZOOM_CLIENT_ID=your-zoom-client-id
ZOOM_CLIENT_SECRET=your-zoom-client-secret
ZOOM_SECRET_TOKEN=your-zoom-secret-token

# =========================
# AI SERVICES (GEMINI)
# =========================
GEMINI_KEY=your-gemini-api-key

# =========================
# EMAIL CONFIGURATION (SMTP)
# =========================
DEFAULT_FROM_EMAIL=your-email@gmail.com
EMAIL_BACKEND=django.core.mail.backends.smtp.EmailBackend
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USE_TLS=True
EMAIL_HOST_USER=your-email@gmail.com
EMAIL_HOST_PASSWORD=your-gmail-app-password
```

### D. Database Migrations & Run
1. Run migrations to initialize the Supabase database:
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```
2. Create a superuser account to log into the Django Admin dashboard:
   ```bash
   python manage.py createsuperuser
   ```
3. Launch the development server:
   ```bash
   python manage.py runserver
   ```
   The backend API will now be active at `http://127.0.0.1:8000/`. You can view the API documentation pages directly at:
   - **Swagger UI**: `http://127.0.0.1:8000/api/docs/`
   - **ReDoc**: `http://127.0.0.1:8000/api/redoc/`

---

## 2. Frontend Setup & Run Guide

### A. Prerequisites
- **Node.js**: Version 18.0.0 or higher.
- **npm**: Node package manager.

### B. Installation Steps
1. Navigate to the frontend directory:
   ```bash
   cd frontend
   ```
2. Install npm package dependencies:
   ```bash
   npm install
   ```

### C. Environment Variables
Create a file named `.env.local` in the `frontend/` directory:
```env
# URL of the main Django backend
NEXT_PUBLIC_BACKEND_URL=http://localhost:9000

# URL of the Lab module backend (if separate, otherwise same as backend)
NEXT_PUBLIC_LAB_BACKEND_URL=http://localhost:8000

# URL of the TrackIntake integration service
NEXT_PUBLIC_TRACKINTAKE_BACKEND_URL=http://localhost:7000

# Razorpay Key ID for client-side checkout
NEXT_PUBLIC_RAZORPAY_KEY_ID=rzp_test_your_key_id
```

### D. Launching the App
1. Start the Next.js development server:
   ```bash
   npm run dev
   ```
2. Open the application in your browser:
   `http://localhost:3000`

---

## 3. Integration & Third-Party APIs

### A. Razorpay Setup
Ensure you configure your Razorpay Key ID and Secret inside Django settings. For local sandbox testing, toggle the gateway mode to `test`.

### B. Zoom API
Configure Zoom OAuth credentials (Client ID, Client Secret, and Account ID) inside the settings dashboard to support automated virtual consultation link generation.

---

## 4. Troubleshooting Common Setup Issues

- **Database Connection Failures**: Check that the `DATABASE_URL` matches the Supabase URL schema and that your IP address is allowed in the Supabase network security whitelist settings.
- **OTP Delivery Failures**: Confirm that `EMAIL_HOST_PASSWORD` and port parameters match your SMTP service credentials.
- **JWT Errors**: Check that the token refresh loop is active. If requests return a `401 Unauthorized` status, verify the SimpleJWT expiry times in `backend/core/settings.py`.
- **Node Modules Build Errors**: Delete the `.next` and `node_modules` folders, clear the npm cache (`npm cache clean --force`), and run `npm install` again.
