# DailyMax v2 — Evidence-Based Powerlifting Autoregulation

A mobile-first PWA for daily readiness estimation and working weight calculation, designed for RPE-prescribed powerlifting programs.

## Algorithm

Every formula is sourced from peer-reviewed research. See `algorithm-spec.md` for the full specification with citations.

**Core pipeline:**
1. Blended Epley-Brzycki e1RM with RPE adjustment (DiStasio 2014, Zourdos et al. 2016)
2. 1-point velocity calibration at 70-75% e1RM (Gemini deep research report)
3. Z-score readiness against 30-day velocity rolling average
4. Traffic light decision matrix (GREEN/AMBER/RED/CRITICAL)
5. Continuous readiness modifier with 0.5× dampening (true 1RM varies only 1-2%)
6. Tuchscherer RPE table for weight prescription
7. Deadlift sensitivity ×1.25 (no SSC, highest CNS demand)
8. Subjective 3-item Hooper Index → volume recommendation (Saw et al. 2016)

## Deploy to GitHub Pages (5 minutes)

1. Create a new repo on GitHub
2. Copy everything inside the `public/` folder to the repo root:
   ```
   index.html
   manifest.json
   sw.js
   icon-192.png
   icon-512.png
   ```
3. Go to **Settings → Pages → Source** → select `main` branch, root folder
4. Save. Your app will be live at `https://yourusername.github.io/repo-name`
5. On your Pixel, open the URL in Chrome → three-dot menu → **Install app**

## Alternative: GitLab Pages

1. Push the `public/` folder contents to a GitLab repo
2. Add this `.gitlab-ci.yml` to the repo root:
   ```yaml
   pages:
     stage: deploy
     script:
       - echo "Deploying"
     artifacts:
       paths:
         - public
     only:
       - main
   ```
3. The app will be at `https://yourusername.gitlab.io/repo-name`

## Features

- **Velocity-based readiness**: Compare today's warm-up velocity to 30-day rolling average
- **Traffic light system**: GREEN/AMBER/RED/CRITICAL with evidence-based thresholds
- **Working weight calculator**: Translates coach's RPE prescription into exact kg
- **Dual-signal architecture**: Velocity governs load, subjective governs volume
- **Post-set feedback loop**: Logged RPE feeds back into e1RM tracking
- **Historical data import**: Bootstrap from existing RPE training logs
- **Coach export**: Copy session summary to clipboard
- **Full backup/restore**: JSON export/import for data safety
- **PWA**: Installable, works offline, data persists in localStorage

## Data

All data is stored in your browser's localStorage. Nothing leaves your device.
Use the backup feature regularly — clearing browser data will erase everything.
