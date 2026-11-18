# TobascoPID.com: website prototype

A dental office picks its x-ray cone, enters three measurements, uploads a photo of the
cone, and downloads a collar file (STEP + STL + print instructions) sized to that cone.
Free, email address only, no login.

## What is in here

| Path | What it is |
|---|---|
| `app/main.py` | Web server (FastAPI): pages, `/api/generate`, downloads, admin export |
| `app/generator.py` | Turns measurements into a collar: tapered bore, liner allowance, STEP/STL/README zip |
| `app/collar_base.py` | The collar geometry (Rev C snap-fit). **Replace with Rev D once validated.** |
| `app/cones.py` | Cone library shown in the drop-down. Add measured dimensions in `MEASURED`. |
| `app/templates/` | Home page, terms and disclaimer, privacy policy (legal pages are DRAFTS) |
| `app/static/` | Styles, page script, favicon |
| `Dockerfile`, `requirements.txt` | For deployment |

## Run it on a computer

```
pip install -r requirements.txt
uvicorn app.main:app --port 8000
```
Open http://localhost:8000. Requests, photos and generated files are stored in `data/`.

## Settings (environment variables)

| Variable | Needed? | What it does |
|---|---|---|
| `SITE_URL` | yes, in production | e.g. `https://tobascopid.com` (used in emails and links) |
| `TOBASCO_DATA` | yes, in production | Folder on a persistent disk, e.g. `/data` |
| `ADMIN_TOKEN` | recommended | A long random password. Then `/admin/requests.csv?token=...` lists every request with links to photos and files |
| `RESEND_API_KEY` | optional | Turns on emailing the file (Resend.com). Without it, the file is download-only |
| `MAIL_FROM` | with email | e.g. `TobascoPID <files@tobascopid.com>` (domain must be verified with the email service) |
| `RATE_PER_HOUR` | optional | Requests per network per hour (default 20) |
| `CAD_WORKERS` | optional | Parallel collar builds (default 2) |

## Deploy (recommended: Render, Docker web service)

1. Put this folder in a GitHub repository (private is fine).
2. On render.com: New → Web Service → connect the repository. Render detects the Dockerfile.
3. Add a persistent disk mounted at `/data` (requests and photos must survive restarts).
4. Set the environment variables above (`SITE_URL`, `TOBASCO_DATA=/data`, `ADMIN_TOKEN`).
5. Settings → Custom domains → add `tobascopid.com` and `www.tobascopid.com`, then add the DNS
   records Render shows at your domain registrar. HTTPS is issued automatically.
6. Optional email: create a Resend account, verify `tobascopid.com` (DNS records), set `RESEND_API_KEY`.

Any host that runs a Docker container with a persistent disk works the same way
(Fly.io, Railway, AWS, Google Cloud Run with a storage bucket).

Tested here: full browser run (choose cone, measure, photo, download), inch input,
tapered cones, input validation, admin export, clean install from `requirements.txt`.
Not tested here: the Docker image build (no Docker engine in this workspace) and email delivery
(no email account yet).

## Before public launch

- [ ] Replace `app/collar_base.py` with the validated Rev D geometry; set `DESIGN_REV` in `generator.py`.
- [ ] Fill `MEASURED` in `app/cones.py` from GNYDM measurements (also in the Cone Library sheet).
- [ ] Attorney review of `terms.html` and `privacy.html` (they are marked DRAFT on the page).
- [ ] Replace the measuring diagram with real photos.
- [ ] Decide how long to keep photos and files; set up disk backups.
- [ ] Turn on email (optional) and test it.
- [ ] Ask Xometry about a partner/referral link and replace the plain link on the home page.
