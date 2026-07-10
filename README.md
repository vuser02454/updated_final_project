# Crowd Heatmap & Business Intelligence Platform

[![Python](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)
[![Django](https://img.shields.io/badge/django-5.2+-green.svg)](https://www.djangoproject.com/)
[![AI](https://img.shields.io/badge/AI-Google%20Gemini-orange.svg)](https://ai.google.dev/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

An end-to-end **Business Intelligence (BI)** web platform that helps entrepreneurs and business owners evaluate where to open a business in India. The system combines **real-time crowd density mapping**, **machine-learning business recommendations**, a **multi-factor revenue forecasting engine**, and an **AI assistant** powered by Google Gemini.

---

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Architecture](#architecture)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Running the Application](#running-the-application)
- [API Endpoints](#api-endpoints)
- [Revenue & Feasibility Engines](#revenue--feasibility-engines)
- [Machine Learning Model](#machine-learning-model)
- [User Guide](#user-guide)
- [Deployment](#deployment)
- [Documentation](#documentation)
- [License](#license)

---

## Overview

The platform answers three core questions for any location in India:

1. **How crowded is this area?** — Sector-based heatmap using OpenStreetMap POI data.
2. **What business should I open here?** — Scikit-Learn decision tree + rule-based feasibility checks.
3. **How much revenue can I expect?** — Multi-factor simulation engine with time-of-day, customer quality, and competition modifiers.

Data is sourced live from **OpenStreetMap** (Nominatim geocoding + Overpass API) within a **2 km analysis radius** (5 km for location matching).

---

## Key Features

### Real-Time Crowd Heatmap
- Search any location in India or use browser geolocation.
- Divides the area into a **3×3 sector grid** and classifies each sector as **High**, **Medium**, or **Low** intensity based on POI density.
- Thresholds: High ≥ 15 POIs/sector, Medium 5–14, Low &lt; 5.

### AI-Powered Business Recommendations
- **Scikit-Learn DecisionTreeClassifier** trained on `business_dataset.csv` predicts the best business type from crowd intensity, shop density, and area type.
- **Google Gemini** chatbot ("Antigravity") provides conversational guidance and can trigger map commands.
- **Smart Relocation (AI Zones)**: Scans ~1.7 km offsets to rank alternative high-potential locations.

### Revenue & Feasibility Engine
- **Smart Revenue Forecast** (`calculate_smart_revenue`): Footfall × conversion × dynamic average spend, adjusted for competition, overload, and customer quality.
- **Per-POI Revenue Enrichment** (`enrich_places_with_revenue`): Estimates revenue for each existing business in the area.
- **Feasibility Checker**: Go/No-Go decision using CSV intensity rules, ML prediction, and live POI evidence.
- **Legacy Revenue Score** (`predict_revenue`): Quick estimate from crowd score (₹1,20,000 + score × ₹7,200/month).

### User Authentication
- Custom email-based user model (`users.CustomUser`) with roles: **Businessman** and **Customer**.
- Registration, login, logout, and password reset flows.

### Interactive Dashboard
- Futuristic analytics UI with live revenue animation, business intelligence panel, and feasibility banners.
- WebSocket-powered chatbot for real-time AI assistance.

---

## Architecture

```text
┌─────────────────────────────────────────────────────────────────┐
│                        Browser (Frontend)                       │
│  Leaflet Map │ Dashboard UI │ Chatbot │ Business Form           │
└──────────────┬──────────────────────────────┬───────────────────┘
               │ HTTP/REST                    │ WebSocket
               ▼                              ▼
┌──────────────────────────┐    ┌──────────────────────────────┐
│   Django Views (REST)    │    │  Django Channels (Daphne)    │
│  heatmap_app/views.py    │    │  heatmap_app/consumers.py    │
└──────────────┬───────────┘    └──────────────┬───────────────┘
               │                               │
               ▼                               ▼
┌──────────────────────────┐    ┌──────────────────────────────┐
│   Revenue & ML Utils     │    │   Google Gemini API          │
│   heatmap_app/utils.py   │    │   (Generative AI Chatbot)    │
└──────────────┬───────────┘    └──────────────────────────────┘
               │
               ▼
┌──────────────────────────┐    ┌──────────────────────────────┐
│  business_model.pkl      │    │  OpenStreetMap APIs          │
│  (Scikit-Learn)          │    │  Nominatim + Overpass        │
└──────────────────────────┘    └──────────────────────────────┘
               │
               ▼
┌──────────────────────────┐
│  SQLite / PostgreSQL     │
│  (User & form data)      │
└──────────────────────────┘
```

---

## Technology Stack

| Layer | Technology |
|-------|------------|
| Backend | Django 5.2+, Django Channels 4.1 |
| ASGI Server | Daphne 4.1 |
| Frontend | HTML5, CSS3, Vanilla JavaScript (ES6+) |
| Machine Learning | Scikit-Learn, Pandas, NumPy |
| Generative AI | Google Generative AI (Gemini 2.0 Flash) |
| Geospatial Data | OpenStreetMap (Nominatim & Overpass API) |
| Database | SQLite (dev) / PostgreSQL via `dj-database-url` (prod) |
| Static Files | WhiteNoise 6.7 |
| Deployment | Render / Railway (Procfile included) |

---

## Project Structure

```text
Business_Intelligence-main/
├── crowd_heatmap_project/       # Django project configuration
│   ├── settings.py              # App settings, DB, Channels, auth
│   ├── urls.py                  # Root URL routing
│   ├── asgi.py                  # ASGI + WebSocket routing
│   ├── wsgi.py
│   ├── train_model.py           # ML model training script
│   ├── business_dataset.csv     # Training data (crowd, shops, area → business)
│   └── business_model.pkl       # Trained DecisionTreeClassifier
│
├── heatmap_app/                 # Core application
│   ├── views.py                 # REST endpoints, feasibility, ML inference
│   ├── utils.py                 # Revenue engine, AI zone generation
│   ├── consumers.py             # WebSocket chatbot (Gemini)
│   ├── models.py                # BusinessUser, ContactMessage
│   ├── forms.py                 # Business registration & contact forms
│   ├── urls.py                  # App URL patterns
│   └── migrations/
│
├── users/                       # Authentication app
│   ├── models.py                # CustomUser (email-based)
│   ├── views.py                 # Login, register, password reset
│   ├── forms.py
│   └── urls.py
│
├── templates/                   # HTML templates
│   ├── heatmap_app/             # Home, dashboard, contact
│   ├── users/                   # Auth pages
│   └── base.html
│
├── static/                      # CSS, JavaScript assets
│   ├── css/style.css
│   └── js/main.js, analytics-dashboard.js, dashboard-form.js
│
├── docs/                        # Extended documentation
│   ├── RUN_INSTRUCTIONS.md
│   ├── GEOLOCATION_GUIDE.md
│   ├── AUTH_STRUCTURE_SUMMARY.md
│   └── REVENUE_STRUCTURE_REPORT.md
│
├── requirements.txt
├── manage.py
├── Procfile                     # Production: daphne ASGI server
└── .env                         # Environment variables (create locally)
```

---

## Getting Started

### Prerequisites

- **Python 3.10+**
- **pip** (Python package manager)
- **Google AI Studio API Key** — [Get one here](https://aistudio.google.com/apikey)

### 1. Clone the Repository

```bash
git clone https://github.com/Rajan-4900/demo_business_intel.git
cd demo_business_intel
```

### 2. Create a Virtual Environment

**macOS / Linux:**
```bash
python3 -m venv venv
source venv/bin/activate
```

**Windows (PowerShell):**
```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure Environment

Create a `.env` file in the project root (see [Environment Variables](#environment-variables)).

### 5. Run Database Migrations

```bash
python manage.py migrate
```

### 6. (Optional) Create Admin User

```bash
python manage.py createsuperuser
```

### 7. (Optional) Retrain ML Model

If `business_model.pkl` is missing or you update the dataset:

```bash
cd crowd_heatmap_project
python train_model.py
cd ..
```

---

## Environment Variables

Create a `.env` file in the project root:

```env
# Required for AI chatbot
GEMINI_API_KEY=your_google_api_key_here

# Optional: override default Gemini model
GEMINI_MODEL_NAME=models/gemini-2.0-flash

# Django settings
SECRET_KEY=your_django_secret_key_here
DEBUG=True

# Production database (optional — defaults to SQLite)
# DATABASE_URL=postgres://user:pass@host:5432/dbname
```

| Variable | Required | Description |
|----------|----------|-------------|
| `GEMINI_API_KEY` | Yes (for chatbot) | Google AI Studio API key |
| `GEMINI_MODEL_NAME` | No | Gemini model identifier (default: `models/gemini-2.0-flash`) |
| `SECRET_KEY` | Yes (prod) | Django secret key |
| `DEBUG` | No | Set to `True` for development |
| `DATABASE_URL` | No | PostgreSQL connection string for production |

---

## Running the Application

### Development (HTTP + WebSockets)

Daphne is listed first in `INSTALLED_APPS`, so `runserver` supports WebSockets:

```bash
python manage.py runserver
```

Open **http://127.0.0.1:8000/**

### Production (ASGI)

```bash
daphne -b 0.0.0.0 -p 8000 crowd_heatmap_project.asgi:application
```

Or use the included `Procfile` on Render/Railway.

---

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | Home page (map) |
| GET | `/dashboard/` | Analytics dashboard |
| POST | `/search-location/` | Geocode a place name |
| GET | `/autocomplete-location/` | Location autocomplete (India) |
| POST | `/find-popular-places/` | POIs within 2 km + per-place revenue |
| POST | `/analyze-crowd-intensity/` | Sector heatmap + ML business prediction |
| POST | `/check-feasibility/` | Go/No-Go feasibility for a business type |
| POST | `/api/analyze-location/` | Full smart revenue forecast |
| POST | `/api/generate-best-locations/` | AI Zone candidates (top 3) |
| GET | `/api/business-types/` | Dynamic business type dropdown |
| POST | `/api/find-matching-locations/` | Match locations by intensity + business |
| POST | `/submit-form/` | Submit business registration form |
| POST | `/chat/` | HTTP chatbot fallback |
| WS | `/ws/chat/` | WebSocket chatbot (Gemini) |

### Example: Analyze Location Revenue

```bash
curl -X POST http://127.0.0.1:8000/api/analyze-location/ \
  -H "Content-Type: application/json" \
  -d '{"latitude": 12.9716, "longitude": 77.5946, "business_type": "cafe"}'
```

Response includes `revenue_data`, `crowd_score`, `feasibility_score`, and `recommendations`.

---

## Revenue & Feasibility Engines

The platform uses **three revenue calculation paths**:

| Engine | Function | Use Case |
|--------|----------|----------|
| Smart Revenue | `calculate_smart_revenue()` | Dashboard, AI zones, location analysis |
| POI Enrichment | `enrich_places_with_revenue()` | Popular places list, area totals |
| Legacy Score | `predict_revenue(crowd_score)` | Quick estimate from crowd intensity |

**Smart Revenue Formula:**

```text
footfall = (22 + POI_count × 2.8) × popularity × time_multiplier
effective_customers = footfall × smart_conversion × customer_quality
daily_revenue = effective_customers × dynamic_avg_spend
monthly_revenue = daily_revenue × 30
```

Modifiers include **Customer Quality Index (CQI)**, **competition density**, **overload penalty**, and **daypart multipliers**.

For a full breakdown, see **[Revenue Structure Report](docs/REVENUE_STRUCTURE_REPORT.md)**.

---

## Machine Learning Model

| Property | Value |
|----------|-------|
| Algorithm | DecisionTreeClassifier (Scikit-Learn) |
| Features | `crowd`, `shops`, `area` (one-hot encoded) |
| Target | `business` (recommended business type) |
| Training script | `crowd_heatmap_project/train_model.py` |
| Model file | `crowd_heatmap_project/business_model.pkl` |

The model is loaded lazily at runtime and used by `predict_business()` in `heatmap_app/views.py`.

---

## User Guide

1. **Register / Login** — Create an account at `/accounts/register/` (email-based).
2. **Search a Location** — Enter a city, neighbourhood, or landmark in India.
3. **Find My Location** — Use browser geolocation to center the map.
4. **Analyze Crowd** — Click "Find Popular Places" or run crowd intensity analysis.
5. **View Heatmap** — High (red), Medium (yellow), Low (green) sectors appear on the map.
6. **Check Feasibility** — Submit the business form with your intended business type.
7. **Revenue Forecast** — Open the Dashboard (`/dashboard/?ai=true`) for detailed revenue metrics.
8. **AI Chatbot** — Ask Antigravity for help, e.g. `"open cafe in Koramangala"`.
9. **AI Zones** — Generate ranked alternative locations within ~1.7 km.

### Chatbot Command Examples

| Command | Action |
|---------|--------|
| `"find my location"` | Center map on user's GPS position |
| `"search for Indiranagar"` | Geocode and navigate to location |
| `"open cafe in Koramangala"` | Feasibility check + map marker |
| `"show popular places"` | Fetch POIs within 2 km |

---

## Deployment

The project is configured for **Render** and **Railway**:

- `Procfile`: `web: daphne -b 0.0.0.0 -p $PORT crowd_heatmap_project.asgi:application`
- `whitenoise` serves static files in production.
- Set `DATABASE_URL` for PostgreSQL.
- Set `DEBUG=False` and a strong `SECRET_KEY` in production.
- Add your domain to `CSRF_TRUSTED_ORIGINS` in `settings.py`.

---

## Documentation

| Document | Description |
|----------|-------------|
| [Run Instructions](docs/RUN_INSTRUCTIONS.md) | Step-by-step setup guide |
| [Geolocation Guide](docs/GEOLOCATION_GUIDE.md) | Map and GPS integration |
| [Auth Structure Summary](docs/AUTH_STRUCTURE_SUMMARY.md) | Authentication system details |
| [Revenue Structure Report](docs/REVENUE_STRUCTURE_REPORT.md) | Full revenue engine analysis |
| [Map Debug Checklist](docs/MAP_DEBUG_CHECKLIST.md) | Troubleshooting map issues |
| [Static Testing](docs/STATIC_TESTING.md) | Static file verification |

---

## License

Distributed under the **MIT License**. See `LICENSE` for more information.

---

## Contributing

1. Fork the repository.
2. Create a feature branch.
3. Make your changes and test locally.
4. Submit a pull request with a clear description.

---

## Acknowledgements

- [OpenStreetMap](https://www.openstreetmap.org/) contributors for geospatial data
- [Google Gemini](https://ai.google.dev/) for generative AI capabilities
- [Django](https://www.djangoproject.com/) and [Channels](https://channels.readthedocs.io/) communities
