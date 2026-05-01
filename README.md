Office Farm – Android App (TWA)

A **Trusted Web Activity (TWA)** Android app that wraps your GitHub Pages site
for Google Play publishing.  No complex native code — your HTML/CSS/JS IS the app.

---

## Project Structure

```
office-farm-app/
│
├── index.html              ← App main screen (2 buttons)
├── manifest.json           ← Web App Manifest (PWA)
├── assetlinks.json         ← Digital Asset Links (upload to GitHub Pages)
│
├── app/
│   ├── build.gradle
│   └── src/main/
│       ├── AndroidManifest.xml
│       └── res/
│           ├── values/colors.xml
│           ├── values/strings.xml
│           ├── values/styles.xml
│           └── drawable/splash.xml
│
├── build.gradle
├── settings.gradle
└── .github/workflows/build.yml   ← Auto-build on GitHub
```

---

## STEP-BY-STEP PUBLISHING GUIDE

### STEP 1 — Host the web files on GitHub Pages

1. Go to your repo: `https://github.com/kampung-kampus/office-farm`
2. Upload `index.html` and `manifest.json` to the repo root (or `/docs` folder).
3. Go to **Settings → Pages** → Source: `main` branch → `/root` → Save.
4. Your site will be live at:
   `https://kampung-kampus.github.io/office-farm/`

---

### STEP 2 — Create a keystore (signing certificate)

Run this **once** on your computer (requires Java installed):

```bash
keytool -genkey -v \
  -keystore office-farm-release.jks \
  -alias officefarm \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000
```

**⚠️ Keep office-farm-release.jks safe — you need it for every update.**

---

### STEP 3 — Get your SHA-256 fingerprint

```bash
keytool -list -v \
  -keystore office-farm-release.jks \
  -alias officefarm
```

Copy the `SHA256:` value that looks like:
`AB:CD:12:34:...` (32 pairs of hex separated by colons)

---

### STEP 4 — Update assetlinks.json

Open `assetlinks.json` and replace the placeholder:
```
"REPLACE_WITH_YOUR_SHA256_SIGNING_CERTIFICATE_FINGERPRINT"
```
with your actual SHA-256 fingerprint (keep the colons, no quotes around each pair).

Then upload `assetlinks.json` to your GitHub Pages repo at this exact path:
```
https://kampung-kampus.github.io/.well-known/assetlinks.json
```

To do this in GitHub, create a folder `.well-known` in your repo and put
`assetlinks.json` inside it.

---

### STEP 5 — Add GitHub Secrets for CI signing

In your GitHub repo → **Settings → Secrets and variables → Actions**, add:

| Secret name        | Value                                      |
|--------------------|--------------------------------------------|
| `SIGNING_KEY`      | Base64-encoded `.jks` file (see below)     |
| `KEY_ALIAS`        | `officefarm`                               |
| `KEY_STORE_PASSWORD` | Password you used in Step 2             |
| `KEY_PASSWORD`     | Key password you used in Step 2            |

**Convert your keystore to Base64:**
```bash
base64 -w 0 office-farm-release.jks
```
Copy the output and paste it as the `SIGNING_KEY` secret.

---

### STEP 6 — Push the Android project to GitHub

Create a new GitHub repo (e.g. `office-farm-android`) and push:

```bash
git init
git add .
git commit -m "Initial TWA app"
git remote add origin https://github.com/kampung-kampus/office-farm-android.git
git push -u origin main
```

---

### STEP 7 — Build the APK via GitHub Actions

Tag a release to trigger the build:
```bash
git tag v1.0.0
git push origin v1.0.0
```

GitHub Actions will:
1. Build the APK
2. Sign it with your keystore
3. Attach it to a GitHub Release

Download the signed APK from the **Releases** page of your repo.

---

### STEP 8 — Publish to Google Play Console

1. Go to https://play.google.com/console
2. **Create app** → Fill in name, language, app type (App), free/paid
3. Complete the **Store listing**:
   - Short description (80 chars)
   - Full description (4000 chars)
   - Screenshots (min 2, phone)
   - Feature graphic (1024×500 px)
   - App icon (512×512 px)
4. Go to **Production → Create new release**
5. Upload your signed `.apk` file
6. Fill in release notes → **Save → Review → Roll out**

Google will review your app (usually 1–3 days for first submission).

---

## Verifying TWA Works

After publishing, visit:
```
https://developers.google.com/digital-asset-links/tools/generator
```
Enter your domain and package name to verify the asset link is correct.

---

## FAQ

**Q: Will the app show the browser address bar?**
A: No — TWA hides it completely once Digital Asset Links are verified.

**Q: Can I update the app without re-submitting to Play Store?**
A: Yes! Just update your GitHub Pages HTML/CSS/JS. The app auto-serves new content.

**Q: The Quit button doesn't fully close on some devices. Why?**
A: Android restricts apps from closing themselves. The button calls `window.close()`
   which works in TWA/standalone mode. On some devices it minimises instead — this
   is normal Android behaviour.
