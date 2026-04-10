# DA Intake Form - Frontend (GitHub Pages)

Static HTML form hosted on GitHub Pages. Posts submissions to a Cloudflare Worker
which emails the results (with Excel summary + attachments) to the DA team.

## Files

- `index.html` — The intake form
- `success.html` — Thank-you page shown after successful submission

## Setup Steps

### 1. Create a GitHub repository

```bash
# On your laptop or VM
git init da-intake-form
cd da-intake-form
# Copy index.html and success.html into this folder
git add index.html success.html README.md
git commit -m "Initial intake form"
git remote add origin https://github.com/YOUR_USERNAME/da-intake-form.git
git push -u origin main
```

### 2. Enable GitHub Pages

1. Go to your repo on github.com
2. **Settings** → **Pages** (left sidebar)
3. Under **Source**, select:
   - Branch: `main`
   - Folder: `/ (root)`
4. Click **Save**
5. After ~1 minute, your form will be live at:
   `https://YOUR_USERNAME.github.io/da-intake-form/`

### 3. Edit `index.html` to plug in your Worker URL and Turnstile key

You need to update **two placeholders** in `index.html` after you deploy
the Cloudflare Worker and create a Turnstile widget:

**a) Worker URL** — near the bottom of `index.html`, find:
```js
var WORKER_URL = "https://YOUR-WORKER-NAME.YOUR-SUBDOMAIN.workers.dev/submit";
```
Replace with your actual Worker URL from the Cloudflare dashboard
(e.g. `https://da-intake-form.yourname.workers.dev/submit`).

**b) Turnstile site key** — in the HTML body, find:
```html
<div class="cf-turnstile" data-sitekey="YOUR_TURNSTILE_SITE_KEY"></div>
```
Replace `YOUR_TURNSTILE_SITE_KEY` with your Turnstile **site key** (public,
starts with `0x4AAAAAAA...`).

How to get a Turnstile site key:
1. Log in to Cloudflare dashboard → **Turnstile** (left sidebar)
2. **Add site** → name it "DA Intake Form"
3. Hostnames: add `YOUR_USERNAME.github.io` (and `localhost` for testing if needed)
4. Widget mode: **Managed**
5. Click **Create**
6. Copy the **Site Key** (goes in HTML) and **Secret Key** (goes in Worker as `TURNSTILE_SECRET`)

### 4. Commit and push the updated HTML

```bash
git add index.html
git commit -m "Plug in Worker URL and Turnstile site key"
git push
```

GitHub Pages will auto-publish within ~1 minute.

## Testing

1. Visit `https://YOUR_USERNAME.github.io/da-intake-form/`
2. Fill out the form
3. Complete the Turnstile check
4. Click **Submit Request**
5. You should see the spinner, then be redirected to `success.html`
6. Check `jack.juang@sciencelogic.com` and `charlene.huang@sciencelogic.com` for the email

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| "Network error" on submit | Worker URL wrong or Worker not deployed | Verify `WORKER_URL` matches Cloudflare dashboard |
| "CORS error" in browser console | Worker's `ALLOWED_ORIGIN` doesn't match GitHub Pages domain | Update `ALLOWED_ORIGIN` in `wrangler.toml` and redeploy |
| "Verification check failed" | Turnstile site key wrong, or hostname not whitelisted | Verify site key; add GitHub Pages hostname in Turnstile settings |
| Form submits but no email arrives | Resend API key wrong, or email landed in spam | Check Worker logs (`wrangler tail`); check spam folder |
| File upload rejected | File too big (>10 MB) or too many files (>5) | Shrink files or split into multiple submissions |

## Security Notes

- **Honeypot** — a hidden `website` field catches dumb bots
- **Turnstile** — Cloudflare's CAPTCHA alternative, required
- **CORS** — Worker only accepts requests from your configured origin
- **File size limits** — enforced both client-side and server-side
- **Reply-To** — email sent to the team has Reply-To set to the submitter, so you can reply directly

## Customization

- **Change recipients** — edit `RECIPIENTS` in `cloudflare/wrangler.toml` and redeploy Worker
- **Change branding** — edit CSS `:root` variables in `index.html`
- **Change file limits** — edit `MAX_FILE_SIZE`, `MAX_FILES`, `MAX_TOTAL_SIZE` in both `index.html` AND `cloudflare/src/index.js` (keep them in sync)

