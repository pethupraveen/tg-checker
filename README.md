# FitCheck — Tempered Glass Checker Engine

A free, single-file web tool that matches tempered glass / screen guard stock against phone display measurements. Runs entirely in the browser — no server, no signup, no data leaves the device.

## Features

- **Data 1 — Phone database**: brand, model, display type (Flat / 3D Curved / 2D Curved), camera cutout (Punch Hole Center/Left/Right, Dynamic Island, U Cut, V Cut), corner radius, overall phone H×W, display area H×W
- **Data 2 — Glass database**: brand, model, glass type (Clear/Full Cover Flat, Hot Bending 3D, Hot Bending 2D), camera cutout, corner radius, glass H×W
- **Checker engine** with 5 criteria: glass type ↔ display type mapping, camera cutout match, corner radius tolerance, height tolerance, width tolerance
- Two modes: **phone → find glasses** and **glass → find phones**
- Adjustable size and radius tolerances, compare against display area or overall body
- **To-scale visual fit overlay** (SVG technical drawing) for every match
- Edit / delete entries, data persists in the browser (localStorage)
- **CSV import/export** for both databases — bulk-load your inventory from Excel (save as CSV)
- **Search phone data online** — searches GSMArena, pick a result and its dimensions plus computed display H×W (from screen size + resolution) load straight into the phone database. Duplicates are detected and skipped with a notification — this applies to online loads, manual adds, and CSV imports alike (existing entries are never overwritten; only new phones are added).

### Notes on online search

GSMArena has no official API and blocks direct browser requests, so the tool tries up to 5 free proxy routes in order (allorigins, Jina Reader, codetabs, corsproxy) until one works, with block-page detection and both HTML and plain-text parsing. GSMArena does not publish corner radius or cutout position, so loaded phones use a 3.0 mm default radius and a best-guess cutout (Dynamic Island for recent iPhones, Punch Hole Center otherwise, curved-display detection from the spec text) — hit **Edit** on the loaded row to fine-tune.

### Online search not working? The permanent fix (5 minutes, free)

Free public proxies share their IPs with thousands of users, so GSMArena rate-limits them ("blocked by site" / "HTTP 429"). Deploy your own tiny proxy on Cloudflare Workers — only you use it, so it never gets blocked:

1. Sign up free at dash.cloudflare.com.
2. Go to **Workers & Pages → Create → Create Worker → Deploy** (accept the default hello-world).
3. Click **Edit code**, delete everything, paste the contents of the included `worker.js`, click **Deploy**.
4. Copy the worker URL shown (looks like `https://tg-proxy.your-name.workers.dev`).
5. In FitCheck, open **Phone data → Advanced** under the search box, paste the URL, click **Save**.

Your proxy is now tried first on every search. Cloudflare's free tier allows 100,000 requests per day — far more than you'll ever need, and responses are cached for an hour so repeat searches are instant.

Other quick checks if search fails:
- Wait 1–2 minutes and retry — free proxies recover quickly.
- Try a shorter query ("S24 Ultra" instead of "Samsung Galaxy S24 Ultra 5G 256GB").
- Open the browser console (F12) — if you see CORS or 429 errors, it's the proxies, not your data; the custom worker above fixes it.
- Ad blockers / privacy extensions sometimes block proxy domains — try once with them off.

## Host it free with GitHub Pages

1. Create a GitHub account at github.com if you don't have one.
2. Click **New repository**, name it e.g. `tg-checker`, keep it **Public**, click **Create repository**.
3. Click **uploading an existing file**, drag `index.html` (and this `README.md`) in, click **Commit changes**.
4. Go to **Settings → Pages**. Under *Build and deployment*, set Source to **Deploy from a branch**, Branch to **main** / **(root)**, click **Save**.
5. Wait about a minute. Your tool is live at:
   `https://<your-username>.github.io/tg-checker/`

Any time you edit `index.html` and commit, the site updates automatically.

## Run locally

Just double-click `index.html` — it works offline after the first load (fonts are the only external resource).

## CSV formats

**phones.csv**
```
brand,model,display_type,camera_cutout,corner_radius,overall_h,overall_w,display_h,display_w
Samsung,Galaxy S24 Ultra,Flat,Punch Hole - Center,2.5,162.3,79.0,157.8,75.6
```

**glasses.csv**
```
brand,model,glass_type,camera_cutout,corner_radius,glass_h,glass_w
XShield,TG-S24U Pro,Clear / Full Cover (Flat),Punch Hole - Center,2.5,157.5,75.2
```

Accepted values — display_type: `Flat`, `3D Curved`, `2D Curved` · glass_type: `Clear / Full Cover (Flat)`, `Hot Bending 3D`, `Hot Bending 2D` · camera_cutout: `Punch Hole - Center`, `Punch Hole - Left`, `Punch Hole - Right`, `Dynamic Island`, `U Cut`, `V Cut`. All measurements in millimetres.

## Matching rules

| Display type | Required glass type |
|---|---|
| Flat | Clear / Full Cover (Flat) |
| 3D Curved | Hot Bending 3D |
| 2D Curved | Hot Bending 2D |

A glass is **Compatible** when all five checks pass; 60%+ shows as **Partial** with the failing criteria listed so you can see exactly why (e.g. width off by +2.3 mm).
