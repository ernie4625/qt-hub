[README.md](https://github.com/user-attachments/files/29611344/README.md)
# qt-hub# QT HUB

Field, office & money workspace for Quality Temp HVAC. Single-page app; data
syncs to the client's own Supabase project.

## Hosting (GitHub Pages)
1. This repo must contain **index.html** (the app).
2. Settings → Pages → Source: Deploy from a branch → Branch: **main** / **/(root)** → Save.
3. Live at: https://<username>.github.io/<repo>/
4. To update: upload a new index.html over the old one (same link).

The app has no secrets in it — the Supabase URL + publishable key are entered at
runtime in Settings → Cloud sync (the publishable key is client-safe by design).
