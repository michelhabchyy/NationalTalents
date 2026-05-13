# National Talents Website

A static, single-file marketing site for National Talents (and the Tālia sub-brand). Everything lives in `index.html` — HTML, CSS, and JavaScript are inlined, so there's no build step.

## Run locally

Open `index.html` directly in a browser, or serve the folder with any static server:

```powershell
# Python 3
python -m http.server 8080

# Node (if you have npx)
npx serve .
```

Then visit http://localhost:8080.

## Deploy to GitHub Pages

1. Create a new repository on GitHub (e.g. `national-talents-website`).
2. From this folder:

   ```powershell
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<your-repo>.git
   git push -u origin main
   ```

3. On GitHub, go to **Settings → Pages**.
4. Under **Build and deployment**, set:
   - **Source:** Deploy from a branch
   - **Branch:** `main` / `(root)`
5. Save. Your site will be live at `https://<your-username>.github.io/<your-repo>/` within a minute or two.

### Custom domain (optional)

In **Settings → Pages → Custom domain**, enter your domain (e.g. `nationaltalents.com`) and add a `CNAME` record at your DNS provider pointing to `<your-username>.github.io`. GitHub will create a `CNAME` file in the repo automatically.

## Files

- `index.html` — the entire site (all pages, styles, scripts)
- `404.html` — fallback that redirects to the homepage so deep links keep working on GitHub Pages
- `.gitignore` — keeps OS/editor cruft out of the repo

## Editing

Open `index.html` in any editor. The file is organized as:

- `<style>` — design tokens at the top (`:root` variables), then component styles
- `<body>` — each "page" is a `<section class="page">` toggled by JS
- `<script>` — navigation, page switching, scroll reveals, mobile menu
