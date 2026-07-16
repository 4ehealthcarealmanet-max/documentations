# Pharmacy Module - Setup and Maintenance Guide

This document outlines the step-by-step setup procedure and ongoing maintenance operations for the Pharmacy Module.

## 1. Local Prerequisites

Ensure the following runtimes are installed on your machine:
* **Python**: Version 3.10 or higher
* **Node.js**: Version 18.x or higher (with `npm`)
* **PostgreSQL**: Version 14 or higher (running and accessible)

---

## 2. Backend Setup (Django)

### Step 2.1: Clone & Establish Environment
Navigate to the `backend/` directory:
```bash
cd backend
```

Create a python virtual environment and activate it:
```bash
python3 -m venv venv
source venv/bin/activate
```

Install all the required backend dependencies:
```bash
pip install -r requirements.txt
```

### Step 2.2: Setup Environment Variables
Copy `.env.example` to a new file named `.env`:
```bash
cp .env.example .env
```
Open `.env` and fill out your local parameters (especially your database connection credentials and API keys).

### Step 2.3: Database Bootstrapping (CRITICAL)
Before running standard Django migrations, you must run the module's custom database bootstrap script. This script establishes the required isolated tables cloned from the main platform's tables, overrides standard Django schemas with redirecting SQL Views, and inserts fake initial migrations to align Django's tracking state:
```bash
python scripts/bootstrap_db.py
```

### Step 2.4: Apply Migrations & Launch
Apply the remainder of Django's local migration stack:
```bash
python manage.py migrate
```

Start the backend development server:
```bash
python manage.py runserver
```
The API is now available at `http://localhost:8000`.

---

## 3. Frontend Setup (Next.js)

### Step 3.1: Install Dependencies
Navigate to the `frontend/` directory:
```bash
cd ../frontend
```

Install the NPM package dependencies:
```bash
npm install
```

### Step 3.2: Configure Environment Variables
Copy the frontend example variables file to `.env.local`:
```bash
cp .env.example .env.local
```
Update the target URL parameters as necessary (e.g. `NEXT_PUBLIC_BACKEND_URL=http://localhost:8000`).

### Step 3.3: Run Development Server
Start the Next.js development server:
```bash
npm run dev
```
Open [http://localhost:3000](http://localhost:3000) in your web browser to access the dashboard portal.

---

## 4. Maintenance Operations

### 4.1 Schema Migrations with SQL Views
Since the module relies on SQL Views (`django_session`, `auth_group`, `django_content_type`) mapping to prefix tables (`pharmacy_session`, etc.), you must exercise caution when adding or updating tables:
* If a model changes, verify that any SQL Views targeting it are updated to avoid schema-out-of-sync database errors.
* Re-run the view definition queries if SQL views get deleted by a migration.

### 4.2 Verifying Key Credentials
* **Cloudinary**: If uploads fail with `401/403` credentials errors, check your `CLOUDINARY_NAME`, `CLOUDINARY_KEY`, and `CLOUDINARY_SECRET` variables.
* **Razorpay**: To test payment flows locally, make sure you configure test credentials in `.env` (`rzp_test_...`).
