# Label Sentinel v2

**Read the label. Show the evidence. Keep humans accountable.**

Evidence-based, human-accountable label-compliance decision support for India. Scan packaged food, medicine, soap, detergent, appliances and more; Label Sentinel checks the labelling rules that apply (Legal Metrology, FSSAI, CDSCO, BIS) and shows the rule, the evidence, the confidence and what to fix. Uncertain findings become **NEEDS REVIEW** for an officer; nothing is guessed.

Team Kanda Poha · YCCE HackKumbh 2026 · Healthcare

## What is in v2

| Area | What it does |
|---|---|
| Auth and roles | Email/password sign-up (citizens), sign-in for Citizen / Officer / Admin, signed HttpOnly session cookie, route protection by role, rate limits, hash-chained audit log |
| Live scanner | Rear camera, continuous focus, torch/zoom, live quality coaching (blur, light, glare), auto-capture, live "which declarations are visible" checklist, barcode detection |
| Image pipeline | Quality analysis, auto-crop, upscaling, shadow flattening, contrast levels, glare tone-compression, sharpening, rotation retry, then on-device OCR (Tesseract.js, self-hosted) |
| AI vision | Gemini reads each panel to a strict JSON schema; cross-checked against OCR. Agreement raises confidence, disagreement flags the field for review. AI never decides a verdict |
| Rule engine | Versioned registry; applicability engine (RAE) picks modules: Legal Metrology (Rules 3, 6, 7, 8, 9, 26), FSSAI, CDSCO (drugs, cosmetics, devices), BIS. Four verdicts: PASS, FAIL, NEEDS_REVIEW, NOT_APPLICABLE |
| Human review | Officer confirms or overrides any finding with a mandatory reason; recomputes the verdict; written to the audit log |
| Online listing check | Paste a product link, listing text, or screenshots. SSRF-safe fetch; Rule 6(10) checks |
| Know More AI | Ingredients and short/long-term health notes (English, Hindi, Marathi). Informational only |
| Officer tools | Review queue, complaint inbox, inspections, cases, compliance reports (print to PDF) |
| Admin tools | Users and roles, rule registry (switch rules on/off, legal-review status), jurisdictions, audit logs |
| Storage | Neon PostgreSQL (JSONB tables), Cloudinary for evidence images. No keys needed for a demo: runs on an in-memory store |

## Quick start (local)

```bash
npm install
cp .env.example .env.local        # optional: add GEMINI_API_KEY, DATABASE_URL, Cloudinary keys
npm run dev                       # http://localhost:3000
```

With no `DATABASE_URL` the app runs in **demo mode** (in-memory data that resets on restart). Demo accounts (password `Sentinel@2026`, or `SEED_PASSWORD`):

- `citizen@labelsentinel.in`
- `officer@labelsentinel.in`
- `admin@labelsentinel.in`

Camera access needs HTTPS or `localhost`. Use the **Upload** button on devices without a camera.

## Tests

```bash
npm test                  # 15 engine tests + 10 server tests (no network needed)
npm run typecheck
node scripts/eval-dataset.cjs <dataset_dir> --source clean|ocr   # accuracy on a generated dataset
```

## Datasets and accuracy measurement

`training/generate_label_dataset.py` creates labelled synthetic label photos (with injected defects and realistic degradations); `training/ocr_dataset.py` runs Tesseract over them; `scripts/eval-dataset.cjs` scores the engine. See `DEPLOY.md` and `docs/DATASET.md`.

## Project map

```
src/app/                 pages (landing, login, signup, citizen/, officer/, admin/) and api/ route handlers
src/components/scanner/  LiveScanner, ScanWorkspace, ListingCheck
src/lib/engine/          extract (parser), merge (OCR+AI), rae, rules (registry + checks), evaluate
src/lib/vision/          enhance (quality + enhancement), ocr (Tesseract worker + retries)
src/server/              db (Neon / memory), auth, crypto, gemini, cloudinary, listing, audit, seed
db/schema.sql            reference schema (created automatically on first request)
public/tessdata, tesseract, tesseract-core   self-hosted OCR assets
scripts/                 tests and dataset evaluation
training/                Python dataset tools (Colab friendly)
```

## Honest limitations

- Rule clauses marked **pending legal review** in the registry come from the team's research report and must be verified against the Gazette text before any official use. Exemption is never treated as compliance.
- Rule 7 (character height) and Rule 8 (placement) need physical measurement; they are advisory officer-check items, not automatic verdicts.
- FSSAI / drug licence numbers are checked for format only; validity needs the regulator's registry.
- Large marketplaces often block server-side page reading; the listing check falls back to pasted text or screenshots.
- Accuracy numbers must come from a labelled test set (see the dataset tools). Nothing here claims a measured real-world accuracy yet.
- Decision support only. It does not replace statutory authority.
