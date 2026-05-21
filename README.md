# NICDC · Legacy Industrial Cluster Questionnaire

A Streamlit web app that captures the **NICDC Legacy Industrial Cluster Questionnaire** from industry-association respondents and gives an administrator a password-gated dashboard to **view, search, inspect, and export** the responses (CSV / Excel / JSON).

---

## Features

- Clean, NICDC-branded portal UI (navy + saffron palette, India Reimagined identity).
- Full multi-section questionnaire (13 sections, ~80 questions) covering:
  - General information, Industry Association, Cluster Information, Support Firms / BDS, Infrastructure, Industrial Parks, Compliance, Value-chain / Export, Market Promotion, Raw Material (repeat for up to 3), Machinery (repeat for up to 3), Finance, Energy & Water, HR / Capacity Building, Waste Management, Conclusion.
- Question types: text, textarea, number, single-select, multi-select, **ranked** multi-select, Yes/No, repeat blocks for raw material / machinery.
- Required-field validation with friendly error messages.
- Server-side persistence in **SQLite** (`data/responses.db`).
- Administrator login (password from `secrets.toml` or env var).
- KPI dashboard, search, section-wise inspection, single-response delete, bulk export.

---

## Quick start (local)

```bash
git clone <this-repo-url> nicdc_app
cd nicdc_app
python -m venv .venv
source .venv/bin/activate         # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# Optional: set a custom admin password for local testing
cp .streamlit/secrets.toml.example .streamlit/secrets.toml
# edit .streamlit/secrets.toml  →  admin_password = "your-strong-password"

streamlit run app.py
```

Open <http://localhost:8501> in your browser.
- **Fill Questionnaire** tab — anyone with the link can submit.
- **Administrator** tab — sign in with the admin password to view / export responses.

If no password is configured, the fallback is `nicdc-admin-2026`. **Change this before deploying.**

---

## Deploy on Streamlit Community Cloud (free)

1. Push this folder to a GitHub repository.
2. Go to <https://share.streamlit.io/> → **New app** → pick your repo, branch, and `app.py`.
3. In **Advanced settings → Secrets**, paste:
   ```toml
   admin_password = "your-strong-password"
   ```
4. Deploy. Your public URL will look like `https://<your-app>.streamlit.app/`.

> ⚠️ **Persistence note:** Streamlit Community Cloud uses an ephemeral filesystem — `data/responses.db` may be reset when the app sleeps or redeploys. For production, point the DB at persistent storage:
> - Set the env var `NICDC_DB_PATH=/mount/data/responses.db` and attach a persistent disk, **OR**
> - Swap the SQLite layer in `storage.py` for Postgres / Google Sheets / S3 (see "Production hardening" below).
>
> Either way, downloading the CSV / Excel from the admin page gives you an offline backup at any time.

---

## Deploy elsewhere

The app is a stock Streamlit application — it runs on any host that supports Python:

| Host                | Notes                                                                  |
|---------------------|------------------------------------------------------------------------|
| Streamlit Cloud     | Easiest. See above.                                                    |
| Render / Railway / Fly.io | Add a Dockerfile or use their Python buildpack; expose port 8501.|
| AWS EC2 / Azure VM  | `streamlit run app.py --server.port 80 --server.headless true`.        |
| Kubernetes          | Wrap in the standard Python+Streamlit image; mount a PVC at `data/`.   |

---

## Project layout

```
nicdc_app/
├── app.py                    # Streamlit entry point (form + admin pages)
├── questions.py              # Source-of-truth schema for every section + question
├── storage.py                # SQLite persistence + DataFrame exports
├── requirements.txt          # Python dependencies (Streamlit, pandas, openpyxl)
├── README.md                 # This file
├── .gitignore                # Excludes secrets + local DB from git
├── .streamlit/
│   ├── config.toml           # Streamlit theme override (NICDC palette)
│   └── secrets.toml.example  # Template for admin_password (copy to secrets.toml locally)
├── assets/
│   └── logo.svg              # NICDC-style emblem + wordmark (inline SVG, no external dep)
└── data/
    └── .gitkeep              # responses.db is created at runtime (gitignored)
```

---

## Production hardening (recommended)

| Concern        | Quick win                                                                          |
|----------------|------------------------------------------------------------------------------------|
| Persistence    | Replace SQLite with Postgres (e.g. Supabase, Neon, RDS); swap calls in `storage.py`.|
| Auth           | Move to OAuth / SSO via `streamlit-authenticator` or your IDP for admin users.     |
| PII handling   | Encrypt `respondent_contact` at rest; restrict admin export to authorised IDs.     |
| Audit log      | Add an `audit_log` table to record admin views / deletes / exports.                |
| Backups        | Cron a daily download or `pg_dump` to S3 / GCS.                                    |
| Branding       | Drop the official NICDC logo PNG into `assets/` and reference it in `app.py`.      |

---

## Customising

- **Add / edit questions** → modify `SECTIONS` in `questions.py`. Storage and UI pick up changes automatically; new keys are exported on next download.
- **Theme** → edit the CSS block at the top of `app.py` and the colour values in `.streamlit/config.toml`.
- **Logo** → replace `assets/logo.svg` with the official NICDC mark. Keep the file name to avoid touching `app.py`.

---

## Licence

Internal use within NICDC and partner organisations. Not for redistribution.
