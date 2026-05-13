# Content Admin Guide

A small custom editor lives at **`/admin/`** on the live site. It's a
password-gated UI that edits the YAML files under `/content/` and
commits changes back to this GitHub repo. The deploy workflow then
rebuilds the live site within a minute.

## Quick map

```
NationalTalents/
├── admin/
│   └── index.html        ← the password-gated editor (self-contained)
├── content/
│   ├── programs.yml      ← every Tālia program (add/edit/remove)
│   ├── jobs.yml          ← every open role on the Careers page
│   └── settings.yml      ← site-wide editable text (hero, contacts, …)
└── index.html            ← the site; reads content/*.yml at runtime
```

## How it works

1. Admin opens `https://<your-site>/admin/`.
2. Types the **shared password** → editor unlocks.
3. On first use only, pastes a **GitHub Personal Access Token** → it's
   stored in their browser (localStorage) and never sent anywhere except
   GitHub's API.
4. Edits programs, roles, or site text in form fields.
5. Clicks **Save changes** → editor commits the updated YAML to GitHub
   via the API → GitHub Actions rebuilds the site → ~1 minute later the
   change is live.

## Security note

The password is a **soft gate** — it's stored in the editor's source
code, so anyone who knows where to look can find it. The **real**
security boundary is the GitHub Personal Access Token: only people you
explicitly issue a token to can actually publish changes.

Treat the password as a "no random visitors can see the admin UI"
control, not a serious lock.

## Changing the password

1. Edit `admin/index.html`.
2. Find the line near the top of the `<script>` block:
   ```js
   const ADMIN_PASSWORD = 'talents2026';
   ```
3. Change the string, commit, push. The next admin visit will require
   the new password.

## Creating a GitHub Personal Access Token

1. Go to <https://github.com/settings/personal-access-tokens/new>
   (fine-grained, recommended).
2. **Token name**: e.g. "National Talents CMS · `<admin-name>`".
3. **Expiration**: pick what you want (max 1 year for fine-grained).
4. **Repository access** → **Only select repositories** → pick
   `michelhabchyy/NationalTalents`.
5. **Permissions → Repository permissions**:
   - **Contents**: Read and write
   - (leave everything else as "No access")
6. Click **Generate token** and copy it immediately (it won't show
   again).
7. Paste it into the editor when prompted.

The token expires after the date you set. The editor will prompt for a
new one when GitHub rejects the old one.

## What you can edit from `/admin/`

| Tab                | Fields |
|--------------------|--------|
| **Tālia Programs** | slug, name, short description, country, duration, price, featured flag, display order, detail-page link |
| **Open Roles**     | title, department, location, type, experience, display order |
| **Site Text**      | homepage hero (tag, two heading lines, subtitle), Tālia tagline + apply label, four contact emails |

For each list (programs, roles) you can add new entries, remove
entries, and reorder with the ↑ / ↓ buttons. Each tab has its own Save
button — saves are per-file.

Anything not in this table still lives directly in `index.html`. If
you want more fields editable, ask and I can extend the schema +
the runtime loader.

## Troubleshooting

- **"Wrong password"** — case-sensitive, no leading/trailing space.
- **"Token rejected by GitHub"** — the PAT doesn't have
  `Contents: write` on the repo, or it's expired. Create a new one.
- **"Conflict: the file changed since you loaded it"** — someone else
  (or you in another tab) edited the same file. Reload the admin page
  and re-make the change.
- **Saved but the site didn't update** — check the deploy workflow at
  <https://github.com/michelhabchyy/NationalTalents/actions>. Each save
  triggers it.
