# Deploying Label Sentinel (Vercel + Neon + Cloudinary + Gemini)

Everything below has a free tier. The app also runs with none of it (demo mode).

## 1. Neon PostgreSQL
1. Create a project at https://neon.tech and copy the **pooled connection string** (`postgresql://...neon.tech/neondb?sslmode=require`).
2. That is your `DATABASE_URL`. Tables are created automatically on the first request. (Optional: paste `db/schema.sql` into the Neon SQL editor.)

## 2. Cloudinary
Dashboard -> copy **Cloud name**, **API key**, **API secret**. Images are uploaded server-side with a signed request into the `label-sentinel` folder.

## 3. Gemini
Create a key at https://aistudio.google.com/apikey -> `GEMINI_API_KEY`.
Model names are retired quickly. The app tries `GEMINI_MODEL` first, then `gemini-3.8-flash`, `gemini-3.5-flash`, `gemini-3.5-flash-lite`. If AI suddenly stops working, check the current model list at https://ai.google.dev/gemini-api/docs/models and set `GEMINI_MODEL`.

## 4. Vercel
1. Push this folder to a GitHub repository.
2. Vercel -> New Project -> import the repo (framework: Next.js, defaults are fine).
3. Add environment variables (Production):

| Variable | Value |
|---|---|
| `DATABASE_URL` | Neon connection string |
| `AUTH_SECRET` | random 32+ chars (`openssl rand -base64 48`) |
| `SEED_PASSWORD` | strong password for the first admin / officer / citizen accounts |
| `SEED_ADMIN_EMAIL` | your admin email |
| `SEED_DEMO_DATA` | `true` for a demo with sample records, `false` for a clean production start |
| `GEMINI_API_KEY` | your key |
| `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET` | from Cloudinary |

4. Deploy, open the site, then visit `/api/health`. Expected: `{"ok":true,"db":"neon-postgres","ai":true,"cloudinary":true}`.
5. Sign in with the seeded admin, change the password policy as needed, create officer accounts in **Admin -> Users**.

## Security checklist
- In production the app **refuses to create default accounts** unless `SEED_PASSWORD` is set, and **refuses to sign sessions** without `AUTH_SECRET`.
- Keep `NEXT_PUBLIC_SHOW_DEMO_LOGINS=false` on real deployments.
- The Gemini and Cloudinary keys are server-side only (never sent to the browser).
- Free-tier rate limits are per server instance; for heavy traffic add a shared limiter (e.g. Upstash).
- Vercel serverless request bodies are limited to ~4.5 MB; the client compresses every image to under ~650 KB before upload.

## Troubleshooting
| Symptom | Fix |
|---|---|
| Camera does not open | Must be HTTPS (Vercel is) or localhost; allow camera permission; use Upload as fallback |
| AI says unavailable | Check `GEMINI_API_KEY`, quota, and `GEMINI_MODEL` |
| Login loop | `AUTH_SECRET` must be set and identical across deployments |
| Listing link "blocked" | The marketplace blocks bots. Paste the text or upload screenshots |
| Slow first scan | First OCR run loads ~5 MB of language data; later scans are faster |
