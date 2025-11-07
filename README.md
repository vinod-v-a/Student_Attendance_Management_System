
---

# 🎓 Student Attendance Management System (FastAPI)

A modern, scalable **Student Attendance Management System** built with **FastAPI**, designed for training institutions to monitor and manage student attendance efficiently.

The system automates **attendance tracking**, **email notifications**, and **Excel-based data imports**, while providing secure **role-based access** for Admins and Students.

---

## 🧭 Table of Contents

* [📖 Overview](#-overview)
* [✨ Features](#-features)
* [🏗️ Project Architecture](#️-project-architecture)
* [⚙️ Tech Stack](#️-tech-stack)
* [🗄️ Database Design](#-database-design)
* [📊 Excel Workflow](#-excel-workflow)
* [🧮 Attendance Calculation Logic](#-attendance-calculation-logic)
* [🔐 Role-Based Access](#-role-based-access)
* [💌 Email Notification Logic](#-email-notification-logic)
* [🚀 Setup Instructions](#-setup-instructions)
* [🌐 API Endpoints Overview](#-api-endpoints-overview)
* [🧩 Application Flow](#-application-flow)
* [✅ Future Enhancements](#-future-enhancements)
* [👨‍💻 Author](#-author)

---

## 📖 Overview

The **Student Attendance Management System** provides an end-to-end solution for:

* Importing attendance data from Excel files.
* Calculating attendance percentages (excluding Sundays).
* Sending automated warning emails when attendance falls below 90%.
* Managing two roles:

  * **Admin:** Manage, upload, and monitor all students.
  * **Student:** View personal attendance details only.

---

## ✨ Features

### 👩‍💼 Admin Capabilities

* Secure JWT-based login.
* Upload Excel file for attendance records.
* Automatically calculate attendance percentage.
* View all students’ records in dashboard format.
* “Send Warning Email” enabled only when:

  * Attendance < 90%
  * **No medical reason provided**

### 🎓 Student Capabilities

* Secure login.
* View individual attendance percentage and details.
* See if a medical reason is marked for absence.

---

## 🏗️ Project Architecture

```bash
attendance_project/
├── app/
│   ├── __init__.py              # Initializes FastAPI package
│   ├── main.py                  # FastAPI app entry point
│   ├── config.py                # App configuration and environment
│   ├── core/
│   │   ├── database.py          # Async SQLAlchemy DB setup
│   │   ├── security.py          # JWT, password hashing
│   │   └── mailer.py            # SMTP email service
│   ├── models/
│   │   ├── __init__.py
│   │   ├── student.py           # Student ORM model
│   │   └── user.py              # User ORM model
│   ├── schemas/
│   │   ├── student_schema.py    # Pydantic schemas for student
│   │   └── user_schema.py       # Pydantic schemas for user/auth
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── landing.py           # Landing or root route
│   │   ├── admin_routes.py      # Admin endpoints
│   │   └── student_routes.py    # Student endpoints
│   ├── services/
│   │   ├── attendance_service.py # Excel handling + attendance logic
│   │   ├── email_service.py     # Handles email sending
│   │   └── auth_service.py      # Login & JWT issuance
│   ├── utils/
│   │   ├── date_utils.py        # Calculates working days, excludes Sundays
│   │   └── file_utils.py        # Handles Excel file validation
│   └── templates/               # Optional Jinja2 templates for UI
│
├── tests/                       # Unit and integration tests
│   ├── test_admin.py
│   ├── test_student.py
│   └── test_auth.py
│
├── .env                         # Environment variables
├── requirements.txt
├── run.py                       # Uvicorn launcher
└── README.md                    # Documentation
```

---

## ⚙️ Tech Stack

| Layer                   | Technology                       |
| :---------------------- | :------------------------------- |
| **Backend Framework**   | FastAPI                          |
| **Database**            | MySQL (Async SQLAlchemy)         |
| **ORM**                 | SQLAlchemy / Alembic             |
| **Authentication**      | JWT (PyJWT, Passlib)             |
| **Email Service**       | SMTP (FastAPI-Mail / aiosmtplib) |
| **Data Import**         | Pandas, openpyxl                 |
| **Frontend (optional)** | Jinja2 Templates, Bootstrap      |
| **Testing**             | Pytest                           |
| **Server**              | Uvicorn / Gunicorn               |

---

## 🗄️ Database Design

### **Table: students**

| Column            | Type             | Description                     |
| ----------------- | ---------------- | ------------------------------- |
| id                | INT PK AUTO      | Unique ID                       |
| student_name      | VARCHAR          | Full name                       |
| phone_no          | VARCHAR          | Contact number                  |
| email_id          | VARCHAR          | Email                           |
| course_start_date | DATE             | Course start date               |
| course_end_date   | DATE             | Course end date                 |
| classes_attended  | INT              | Number of classes attended      |
| medical_reason    | ENUM('Yes','No') | Indicates medical justification |

### **Table: users**

| Column     | Type                    | Description                             |
| ---------- | ----------------------- | --------------------------------------- |
| id         | INT PK AUTO             | Unique ID                               |
| username   | VARCHAR                 | Login username                          |
| password   | VARCHAR                 | Hashed password                         |
| role       | ENUM('admin','student') | Role type                               |
| student_id | INT FK                  | Linked to students (nullable for admin) |

---

## 📊 Excel Workflow

**Admin uploads an Excel sheet** (daily or weekly):

```csv
slno, student_name, phone_no, email_id, course_start_date, course_end_date, classes_attended, medical_reason
```

The backend will:

* Parse Excel with **pandas**.
* Update student records in DB.
* Recalculate attendance percentages automatically.

---

## 🧮 Attendance Calculation Logic

```python
def calculate_attendance_percentage(start_date, end_date, classes_attended):
    total_days = working_days_between(start_date, end_date)  # Excludes Sundays
    if total_days == 0:
        return 0
    return round((classes_attended / total_days) * 100, 2)
```

**Email Button Enable Logic:**

| Condition                                 | Send Mail Button |
| ----------------------------------------- | ---------------- |
| Attendance ≥ 90%                          | ❌ Disabled       |
| Attendance < 90% & medical_reason = 'Yes' | ❌ Disabled       |
| Attendance < 90% & medical_reason = 'No'  | ✅ Enabled        |

---

## 🔐 Role-Based Access

| Role                | Permissions                                  |
| ------------------- | -------------------------------------------- |
| **Admin**           | Upload Excel, view all students, send emails |
| **Student**         | View own attendance only                     |
| **Unauthenticated** | Redirect to login endpoint                   |

---

## 💌 Email Notification Logic

```python
if student.attendance_percentage < 90 and student.medical_reason == 'No':
    send_warning_email(student.email_id)
```

**Sample Email:**

> **Subject:** ⚠️ Low Attendance Alert
> Hello [Student Name],
>
> Your attendance stands at **[percentage]%**.
> Please ensure consistent attendance to meet course requirements.
>
> Regards,
> Training Admin Team

---

## 🚀 Setup Instructions

### 1️⃣ Clone Repository

```bash
git clone https://github.com/your-org/student-attendance-fastapi.git
cd student-attendance-fastapi
```

### 2️⃣ Create Virtual Environment

```bash
python -m venv venv
source venv/bin/activate   # Mac/Linux
venv\Scripts\activate      # Windows
```

### 3️⃣ Install Dependencies

```bash
pip install -r requirements.txt
```

### 4️⃣ Setup Environment Variables (`.env`)

```ini
DATABASE_URL=mysql+aiomysql://user:password@localhost/attendance_db
JWT_SECRET=your_secret_key
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your_email@gmail.com
EMAIL_PASS=your_app_password
```

### 5️⃣ Run Database Migrations

```bash
alembic upgrade head
```

### 6️⃣ Start the Server

```bash
uvicorn run:app --reload
```

Application runs at 👉 **[http://127.0.0.1:8000](http://127.0.0.1:8000)**

---

## 🌐 API Endpoints Overview

| Endpoint                           | Method | Role    | Description               |
| ---------------------------------- | ------ | ------- | ------------------------- |
| `/`                                | GET    | All     | Landing route             |
| `/admin/login`                     | POST   | Admin   | Login with credentials    |
| `/admin/upload`                    | POST   | Admin   | Upload Excel attendance   |
| `/admin/students`                  | GET    | Admin   | View all students         |
| `/admin/send-warning/{student_id}` | POST   | Admin   | Send low attendance email |
| `/student/login`                   | POST   | Student | Student login             |
| `/student/me`                      | GET    | Student | Fetch personal attendance |

---

## 🧩 Application Flow

```plaintext
Landing Page
 ├── Admin Login → Admin Dashboard
 │     ├── Upload Excel → DB Update
 │     ├── Calculate % (exclude Sundays)
 │     ├── Show All Students
 │     └── Send Email (if <90% & no medical reason)
 └── Student Login → Student Dashboard
       ├── View Own Attendance
       └── Medical Reason Display
```

---

## ✅ Future Enhancements

* 📈 Add dashboard analytics with Plotly/Chart.js.
* ☁️ Store uploaded Excel files in S3 / Azure Blob.
* 🔔 Integrate WhatsApp or SMS alerts.
* 🗓️ Calendar visualization for attendance.
* 📲 Add React or Next.js frontend.

---

