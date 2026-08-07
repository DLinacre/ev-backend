# Deploying ev-bot.uk — Step by Step

Total cost: £0. Time: ~30 minutes.

---

## Part 1 — Push your code to GitHub

If you haven't already got a GitHub repo for this project:

1. Go to https://github.com/new → create a new repo called `ev-backend` (private is fine)
2. In your `D:\EV\ev-backend` folder, open a terminal and run:

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/DLinacre/ev-backend.git
git push -u origin main
```

---

## Part 2 — Deploy the backend on Render (free)

Render will run your Docker container for free.

1. Go to https://render.com and sign up (use your GitHub account)
2. Click **New → Web Service**
3. Connect your GitHub repo (`ev-backend`)
4. Settings:
   - **Name**: `ev-bot-backend`
   - **Region**: Frankfurt (closest to UK)
   - **Branch**: main
   - **Runtime**: Docker (Render detects your Dockerfile automatically)
   - **Instance type**: Free

5. Under **Environment Variables**, add:

| Key | Value |
|-----|-------|
| `ENVIRONMENT` | `production` |
| `AI_PROVIDER` | `openai` |
| `OPENAI_API_KEY` | your OpenAI key (get one at platform.openai.com) |
| `OPENAI_MODEL` | `gpt-4o-mini` |
| `MEMORY_BACKEND` | `sqlite` |
| `ADMIN_API_KEY` | pick a long random string |
| `CORS_ORIGINS` | `https://ev-bot.uk,https://www.ev-bot.uk` |

6. Click **Create Web Service**. Render will build and deploy (~5 minutes).

Your backend URL will be something like: `https://ev-bot-backend.onrender.com`

Test it: `https://ev-bot-backend.onrender.com/api/v1/health` should return `{"status": "ok"}`.

> **Free tier note**: Render's free tier spins down after 15 minutes of inactivity. First request after sleep takes ~30 seconds. This is fine to start — upgrade to the $7/month Starter tier once you're making money.

---

## Part 3 — Update the frontend with your backend URL

Open `D:\EV\ev-backend\ev-bot-frontend.html` and find this line near the top of the `<script>`:

```js
const API_BASE_URL = '';  // e.g. 'https://ev-bot-backend.onrender.com'
```

Change it to:

```js
const API_BASE_URL = 'https://ev-bot-backend.onrender.com';
```

Save the file.

---

## Part 4 — Deploy the frontend on Cloudflare Pages (free)

Cloudflare Pages serves static files for free, globally fast.

1. Go to https://pages.cloudflare.com and sign up
2. Click **Create a project → Connect to Git**
3. Connect your GitHub account and select your `ev-backend` repo
4. Settings:
   - **Project name**: `ev-bot`
   - **Build command**: *(leave empty)*
   - **Build output directory**: `/` *(root — the HTML file is right there)*
   - **Root directory**: `/` *(leave as default)*
5. Click **Save and Deploy**

Your frontend will be live at: `https://ev-bot.pages.dev`

---

## Part 5 — Point ev-bot.uk at Cloudflare Pages

1. In Cloudflare Pages, go to your project → **Custom domains**
2. Click **Set up a custom domain** → enter `ev-bot.uk`
3. Cloudflare will show you DNS records to add. Since you own the domain, log into your domain registrar and add them (usually two CNAME records).
4. Also add `www.ev-bot.uk` as a custom domain so both work.

DNS propagation takes 5–30 minutes. After that, https://ev-bot.uk is live.

---

## Part 6 — Get your OpenAI API key

If you don't have one yet:

1. Go to https://platform.openai.com/signup
2. Sign up and add a payment method (start with a $5 top-up)
3. Go to API Keys → Create new key
4. Copy it into Render's environment variables (Part 2, step 5)

**Cost reality check**: GPT-4o-mini costs ~$0.00015 per 1K tokens. A typical chat message exchange is ~500 tokens. That's $0.000075 per conversation — you'd need 270,000 conversations to spend $20. Early traffic will cost pennies.

---

## Part 7 — Add affiliate links (where the money comes in)

Open `ev-bot-frontend.html` and search for `data-affiliate`. You'll find links like:

```html
<a href="#" data-affiliate="ohme">Ohme</a>
```

Replace `#` with your actual affiliate URLs:

| Brand | Where to get affiliate link |
|-------|----------------------------|
| Ohme | https://ohme-ev.com/affiliates |
| Pod Point | https://pod-point.com/affiliates |
| Easee | Contact them directly at easee.com |
| Octopus Energy | https://octopus.energy/affiliates |

Even without affiliate links, you can add a simple "Get a quote" button pointing to comparison sites like https://www.go.compare.com/electric-car-insurance with your referral tag.

---

## What to do next (once live)

1. **Submit to Google Search Console** — tells Google your site exists
2. **Write 3-5 short pages** — "Best home EV chargers UK 2026", "EV grants guide", etc. These rank in Google and bring free traffic
3. **Share in EV Facebook groups / Reddit r/electricvehiclesUK** — post helpful answers and mention your tool
4. **Track signups/clicks** — add free Google Analytics or Plausible

---

## Quick reference — your new files

| File | What it does |
|------|-------------|
| `app/api/routes/chat.py` | New `/api/v1/chat` endpoint |
| `app/services/web_chat_service.py` | EV expert AI orchestration |
| `ev-bot-frontend.html` | The website — deploy this to Cloudflare |
