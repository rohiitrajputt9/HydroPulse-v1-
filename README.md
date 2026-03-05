# HydroPulse v1 — Multi-User Role-Based Water Monitoring Platform

A production-grade SaaS-style water consumption monitoring platform with statistical anomaly detection, role-based authentication, and advanced analytics.

---

## System Architecture

```
Browser (HTML + CSS + Vanilla JS + Chart.js)
    |  HTTP + session cookie
    v
Express.js Server (Node.js)
    ├── /api/auth/*          Auth routes (login, register, logout, me, profile)
    ├── /api/water/*         Water routes (entry, dashboard, alerts, owner)
    ├── Middleware            requireAuth | requireOwner | requireResident | errorHandler
    └── Static files         /frontend/**
    |
    v
MySQL Database
    ├── users                id, name, email, password, room_no, role, is_active
    ├── daily_entries        per-day water log with anomaly detection results
    └── anomaly_logs         all ALERT events with full statistical context
```

---

## Role-Based Access Flow

```
Public
  /                        → Landing page
  /pages/login.html        → POST /api/auth/login
  /pages/register.html     → POST /api/auth/register (RESIDENT only)

RESIDENT panel
  dashboard.html           → GET /api/water/dashboard
  entry.html               → POST /api/water/entry
  analytics.html           → GET /api/water/dashboard?days=N
  alerts.html              → GET /api/water/alerts
  profile.html             → PUT /api/auth/profile
  settings.html            → localStorage (theme)

OWNER panel
  dashboard.html           → GET /api/water/owner/dashboard
  residents.html           → GET + PATCH /api/water/owner/resident/:id/toggle
  compare.html             → GET /api/water/owner/dashboard (chart derived)
  anomalies.html           → GET /api/water/owner/anomalies
  export.html              → GET /api/water/owner/export (CSV)
```

---

## Anomaly Detection — How It Works

### Primary: Statistical (3+ days history)

1. Fetch last 3 days actual_usage for user.
2. Mean = sum / 3
3. Std Dev = sqrt( sum((x - mean)^2) / n )
4. Threshold = mean + 2σ
5. actual > threshold → ALERT, else NORMAL
6. On ALERT: write to anomaly_logs with full context.

### Fallback: 20% Rule (fewer than 3 days)

threshold = expected × 1.2  →  actual > threshold = ALERT

---

## Dark Mode Implementation

Uses CSS custom properties on html[data-theme="dark"]. Toggled via JS, persisted to localStorage. Charts regenerate colors by reading CSS variables at render time.

---

## Setup

```bash
# 1. Install
cd HydroPulse && npm install

# 2. Configure
cp .env.example .env
# Set DB_HOST, DB_USER, DB_PASS, DB_NAME, SESSION_SECRET

# 3. Database
mysql -u root -p < database/schema.sql

# 4. Run
npm start        # or: npm run dev

# 5. Open
http://localhost:3000
# All seed accounts use password: Admin@1234
# Owner:    owner@aquaguard.com
# Resident: rahul@aquaguard.com
```

---

## Future Scalability

- IoT: Replace manual tank readings with ESP32 flow sensors over MQTT
- Cloud: Deploy to AWS EC2 + RDS, serve frontend via CloudFront CDN
- Scaling: Replace express-session with JWT + Redis for stateless horizontal scaling
- Alerts: SMS/email via Twilio/SendGrid when anomaly is detected
- Forecasting: Linear regression on rolling history for predictive alerts
