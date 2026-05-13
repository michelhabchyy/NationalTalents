# Content Admin Guide

The site uses **Decap CMS** (formerly Netlify CMS) — a browser-based
editor that lives at **`/admin/`** and commits changes directly to this
repo. Once a change is committed, GitHub Actions rebuilds the live site
within a minute.

## Quick map

```
NationalTalents/
├── admin/
│   ├── index.html        ← the CMS UI (loads Decap from CDN)
│   └── config.yml        ← what collections / fields the CMS exposes
├── content/
│   ├── programs.yml      ← every Tālia program (add/edit/remove)
│   ├── jobs.yml          ← every open role on the Careers page
│   └── settings.yml      ← site-wide editable text (hero, contacts, ...)
└── index.html            ← the site; reads content/*.yml at runtime
```

## What you can edit from `/admin/`

| Collection         | Fields |
|--------------------|--------|
| **Tālia Programs** | name, short description, country, duration, price, featured flag, display order, detail-page link |
| **Open Roles**     | title, department, location, type, experience, display order |
| **Site Text**      | homepage hero (tag, heading, subtitle), Tālia tagline, contact emails |

Anything not in this list still lives directly in `index.html`. If you
want more fields editable, add them to `admin/config.yml` and add a
matching `data-cms="..."` attribute in `index.html`.

---

## One-time setup: enabling login

Decap needs to authenticate the admin with GitHub before it can commit.
On a plain GitHub Pages site this needs a tiny OAuth proxy. **Pick the
path that fits you best.**

### Option A — move the site to Netlify (easiest)

Netlify hosts static sites the same way GitHub Pages does (free), and
provides built-in auth — no proxy to set up.

1. Sign up at https://netlify.com with the same GitHub account.
2. **Add new site → Import an existing project → GitHub** → pick
   `michelhabchyy/NationalTalents`. Leave the build settings empty
   (no build command, publish directory = root). Click Deploy.
3. The site is now live at `<random>.netlify.app`. You can leave GitHub
   Pages running in parallel — they don't conflict.
4. In Netlify: **Site settings → Identity → Enable Identity**.
5. **Identity → Registration → Invite only** (so randos can't sign up).
6. **Identity → Services → Enable Git Gateway** (this is what lets
   Decap commit on your behalf).
7. **Identity → Invite user** → enter the admin's email.
8. In `admin/config.yml`, change the backend section to:
   ```yaml
   backend:
     name: git-gateway
     branch: main
   ```
   (replace the existing `name: github` block). Commit & push.
9. The admin opens `https://<your-site>.netlify.app/admin/`, accepts
   the invite email, sets a password, and is in.

### Option B — stay on GitHub Pages + Cloudflare Worker OAuth proxy

Keeps the current `michelhabchyy.github.io/NationalTalents/` URL.
Requires deploying a small worker. Most users find Option A easier.

1. **Create a GitHub OAuth App**:
   - https://github.com/settings/developers → **New OAuth App**
   - Application name: `National Talents CMS`
   - Homepage URL: `https://michelhabchyy.github.io/NationalTalents/`
   - Authorization callback URL: `https://<your-worker>.workers.dev/callback`
     (you'll fill in `<your-worker>` after step 3)
   - Save the **Client ID** and generate a **Client Secret**.

2. **Sign up at Cloudflare** (free): https://dash.cloudflare.com/sign-up

3. **Workers & Pages → Create application → Create Worker**, give it
   a name like `nt-cms-auth`, click Deploy.

4. After deploy, click **Quick Edit** and paste the worker code from
   https://github.com/sterlingwes/decap-proxy/blob/main/index.ts
   (or any maintained Decap OAuth proxy). Save.

5. **Settings → Variables → Add Environment Variable**:
   - `OAUTH_CLIENT_ID` = the Client ID from step 1
   - `OAUTH_CLIENT_SECRET` = the Client Secret from step 1 (mark as
     encrypted)

6. Go back to your GitHub OAuth App and paste the worker URL
   (`https://nt-cms-auth.<account>.workers.dev/callback`) into the
   Authorization callback URL.

7. In `admin/config.yml`, set:
   ```yaml
   backend:
     name: github
     repo: michelhabchyy/NationalTalents
     branch: main
     base_url: https://nt-cms-auth.<account>.workers.dev
   ```
   Commit & push.

8. The admin opens `/admin/`, clicks "Login with GitHub", authorizes
   the OAuth app once, and is in.

---

## Day-to-day editing

1. Go to **`/admin/`** on the live site.
2. Log in.
3. Pick a collection on the left (Tālia Programs, Open Roles, Site
   Text).
4. Edit fields. Decap shows a live preview where applicable.
5. Click **Publish → Publish now**.
6. Behind the scenes Decap writes the file to GitHub, the deploy
   workflow runs, and the site updates in ~1 minute.

### Adding a new Tālia program
- **Tālia Programs → Programs** collection → **Add Program**.
- Fill in the fields. `id` is the slug (lowercase, no spaces — used
  in the URL and as the CSS class for the card's background image).
- `featured: true` puts the "FEATURED" badge on the card.
- `detail_page` should be the page ID (e.g. `talia-rome`) of an
  existing detail page, or left empty if you haven't built one yet.
- Save → Publish.

### Adding a new job
- **Open Roles → Jobs** → **Add Role**. Fill in the fields. Save → Publish.

### Editorial workflow (drafts)
`admin/config.yml` has `publish_mode: editorial_workflow` on, which
means every change goes through a draft pull request before going
live. If you'd rather commit directly to main, delete that line.

---

## Image uploads (next phase)

The current programs use CSS gradient backgrounds rather than real
photos. Once you want to upload actual photography:

1. The CMS already has a media folder configured at `/images/`.
2. Add an `image` field to the program card render so uploaded files
   replace the gradient background.

I can wire that up when you're ready.

---

## Troubleshooting

- **`/admin/` is blank** — open the browser console. Common causes:
  the `backend.repo` value in `config.yml` is wrong, or the OAuth
  proxy URL hasn't been set yet.
- **"Failed to load entries"** — the YAML files in `/content/` are
  missing or have a syntax error. Open them on GitHub and check.
- **Saved a change but the live site didn't update** — check the
  Actions tab: https://github.com/michelhabchyy/NationalTalents/actions
  The deploy workflow runs after every commit.
