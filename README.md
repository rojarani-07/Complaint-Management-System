# EduVoice — Digital Complaint Management System

AI/ML-powered complaint management for educational institutions.
Built with **Python (Flask)** backend, **MySQL** database, **HTML/CSS/JS** frontend,
and a **Random Forest** ML model for intelligent complaint routing.

---

## Features

- Student complaint submission with live AI prediction preview
- Random Forest classifier: auto-categorizes, predicts priority, sentiment, department
- Admin dashboard with analytics charts (Chart.js)
- Full complaint lifecycle: submit → review → in_progress → resolved
- Timeline tracking and comment threads per complaint
- Notification system for status updates
- ML model retraining endpoint
- Anonymous complaint submission option
- 12 educational complaint categories

---

## Project Structure

```
complaint_system/
├── backend/
│   ├── __init__.py
│   ├── app.py              ← Flask REST API (all routes)
│   └── database.py         ← MySQL connection helper
│
├── database/
│   └── schema.sql          ← All tables + seed data
│
├── frontend/
│   ├── templates/
│   │   ├── index.html      ← Login / Register page
│   │   ├── dashboard.html  ← Student dashboard
│   │   └── admin.html      ← Admin panel
│   └── static/
│       ├── css/
│       │   ├── main.css
│       │   ├── dashboard.css
│       │   └── admin.css
│       └── js/
│           ├── main.js     ← Shared utilities
│           ├── dashboard.js
│           └── admin.js
│
├── ml_model/
│   ├── __init__.py
│   ├── complaint_classifier.py   ← Random Forest pipeline
│   └── saved_models/             ← Auto-created on first run
│       ├── category_model.pkl
│       ├── priority_model.pkl
│       ├── sentiment_model.pkl
│       ├── routing_model.pkl
│       └── *_encoder.pkl
│
├── uploads/                      ← File attachments (auto-created)
├── run.py                        ← App entry point
├── init_db.py                    ← DB setup helper
├── requirements.txt
└── .env.example
```

---

## Setup Instructions

### 1. Prerequisites

- Python 3.10+
- MySQL 8.0+
- pip

### 2. Install Dependencies

```bash
cd complaint_system
pip install -r requirements.txt
```

### 3. Configure Database

Edit `init_db.py` and set your MySQL credentials:
```python
os.environ.setdefault('DB_PASSWORD', 'your_mysql_password')
```

Or copy `.env.example` to `.env` and fill in values.

### 4. Initialize Database

```bash
python init_db.py
```

This creates all tables and seeds:
- 3 sample institutions
- 12 complaint categories

### 5. Create Admin User

```sql
-- Run in MySQL:
USE complaint_management;
INSERT INTO users (institution_id, full_name, email, password_hash, role, department)
VALUES (1, 'Admin User', 'admin@nit.edu',
  '<hash_from_werkzeug>',  -- see note below
  'admin', 'Administration');
```

Or use Python to generate the hash:
```python
from werkzeug.security import generate_password_hash
print(generate_password_hash('admin123'))
```

### 6. Run the Application

```bash
python run.py
```

ML models train automatically on first launch.
Open **http://localhost:5000** in your browser.

---

## ML Model Details

| Model | Algorithm | Input | Output |
|-------|-----------|-------|--------|
| Category Classifier | Random Forest (200 trees) | TF-IDF bigrams | 12 categories |
| Priority Predictor | Random Forest (200 trees) | TF-IDF bigrams | low/medium/high/critical |
| Sentiment Analyzer | Random Forest (150 trees) | TF-IDF bigrams | positive/negative/very_negative |
| Department Router | Random Forest (200 trees) | TF-IDF bigrams | Department name |

### How it works

1. Complaint title + description → lowercase + clean
2. TF-IDF vectorization (unigrams + bigrams, max 5000 features)
3. Four separate Random Forest classifiers predict in parallel
4. If sentiment=`very_negative` AND confidence > 70% → auto-escalate to `critical`
5. Confidence score displayed to admins for transparency

### Retraining

- **Via API**: `POST /api/ml/retrain` (admin only)
- **Via UI**: Admin Panel → ML Insights → Retrain Model button
- **Via CLI**: `python ml_model/complaint_classifier.py`

---

## API Reference

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/auth/register` | None | Register new user |
| POST | `/api/auth/login` | None | Login |
| POST | `/api/auth/logout` | User | Logout |
| GET | `/api/auth/me` | User | Current user info |
| GET | `/api/institutions` | None | List institutions |
| GET | `/api/categories` | None | Complaint categories |
| POST | `/api/complaints` | User | Submit complaint |
| GET | `/api/complaints` | User | List complaints |
| GET | `/api/complaints/:id` | User | Complaint detail |
| PUT | `/api/complaints/:id/status` | Admin | Update status |
| POST | `/api/complaints/:id/comment` | User | Add comment |
| POST | `/api/ml/predict` | None | Live ML prediction |
| POST | `/api/ml/retrain` | Admin | Retrain models |
| GET | `/api/analytics/stats` | User | Dashboard stats |
| GET | `/api/notifications` | User | Notifications |

---

## User Roles

| Role | Permissions |
|------|-------------|
| `student` | Submit complaints, view own complaints, comments |
| `faculty` | Same as student |
| `admin` | All complaints at institution, update status, retrain ML |
| `super_admin` | All institutions |

---

## Complaint Categories

Academic · Infrastructure · Faculty · Administration · Hostel · Library ·
Canteen · Transportation · IT Services · Financial · Ragging/Harassment · Examination
"# Complaint-Management-System" 
