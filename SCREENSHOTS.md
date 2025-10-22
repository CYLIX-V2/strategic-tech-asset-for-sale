# CYLIX V2 – Screenshot Guide

## 🎯 Purpose
Provide an accurate, honest visual preview without video. This guide lists the recommended screenshots, capture steps, and redaction practices.

---

## 📸 Shot List (10–12 images)

- 01_dashboard_overview.png
  - Main dashboard with metrics cards and charts (1920x1080)
- 02_login_page.png
  - Login screen before credentials
- 03_login_success.png
  - Post-login state (mask usernames/emails if needed)
- 04_events_list.png
  - Events/threats table view (show filters)
- 05_event_details.png
  - Single event detail modal/drawer if present
- 06_ai_insights.png
  - AI insights panel (simulated insights)
- 07_settings_theme.png
  - Settings page with dark/light toggle
- 08_api_docs.png
  - FastAPI Swagger UI at /docs (mock backend)
- 09_health_endpoint.png
  - /health response in browser or curl output
- 10_grafana_login.png
  - Grafana login page (before SSO unification)
- 11_grafana_dashboard.png
  - Grafana dashboard (sanitize any real host/IP)
- 12_sso_unification_diagram.png
  - From UNIFIED_SSO_SETUP.md (optional diagram)

Store all images in `screenshots/` and reference them from `README.md` if desired.

---

## 🧭 Where to Capture From

- UI (frontend): `http://localhost:3001`
- Mock API Swagger: `http://localhost:8001/docs`
- Health endpoint: `http://localhost:8001/health`
- Grafana (if running): `http://localhost:3000` or your configured host

Frontend start (example):
```bash
# from cylix_v2_build/frontend/
VITE_API_URL=http://localhost:8001 npm run dev -- --port 3001
```

Mock auth backend:
```bash
# from cylix_v2_build/
python3 mock_backend_auth.py
```

---

## 🛠️ Capture Commands (Linux)

- GNOME Screenshot (window):
```bash
gnome-screenshot -w -B -f screenshots/01_dashboard_overview.png
```

- Flameshot (interactive):
```bash
flameshot gui -p screenshots/
```

- ImageMagick (crop/resize):
```bash
mogrify -resize 1600x900 -quality 92 screenshots/*.png
```

---

## 🔒 Redaction & Hygiene

- Remove or blur any secrets, tokens, emails, or internal IPs
- Prefer masked user data (e.g., admin@masked.local)
- Use ImageMagick to blur areas:
```bash
convert screenshots/02_login_success.png -region 500x60+700+220 -blur 0x12 screenshots/02_login_success.png
```
- Ensure no system tray/personal apps are visible

---

## 🧾 File Naming & Alt Text

- Use the `NN_title.png` scheme (ordered flow)
- Provide short alt text in markdown embeds, e.g.:
```markdown
![Dashboard overview](screenshots/01_dashboard_overview.png)
```

---

## ✅ Quality Tips

- Resolution: 1920x1080 preferred
- Theme: Dark mode for SOC-optimized visuals
- Cursor: Keep steady, avoid covering key UI
- Consistency: Same window size and zoom for all

---

## 📦 Delivery

- Include `screenshots/` folder in the repo (sanitized)
- Alternatively, provide a ZIP in the release assets (optional)

---

*As-is representation aligned with CURRENT_STATE_ASSESSMENT.md.*