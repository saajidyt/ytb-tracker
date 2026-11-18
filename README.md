# YTB Edits Tracker

A small web app for the YT for Brands team to see **who is working on which video, at what stage, and what's late**.
Editors, content managers and admins sign in on the website; everything they do is saved in a Google Sheet.

- **Live tracker:** https://tracker.ytforbrands.com
- **Pipeline prototype:** https://tracker.ytforbrands.com/prototype.html (sample data only, not connected to the sheet)

---

## How it works

```
Browser (index.html on GitHub Pages)
        │  HTTPS requests (JSON)
        ▼
Google Apps Script web app  ──►  Google Sheet "YTB Edits Tracker"
(sign-in, permissions, logging)    one tab per editor + Overview, Lists, Activity
```

| Part | Where it lives | What it does |
|---|---|---|
| `index.html` | This repo, served by GitHub Pages | The whole website: sign-in, Board, Table, Admin dashboard, Activity, Access, Settings. One file, no build step. |
| Apps Script backend | Extensions → Apps Script in the Google Sheet (not in this repo) | Checks sign-ins, reads and writes rows, keeps the Activity log, colours the sheet. |
| Google Sheet | Google Drive (owner: Saajid) | The data. One tab per editor, plus **Overview**, **Lists** (dropdown options) and **Activity** (change log). |
| `prototype.html` | This repo | Standalone preview of the next version (pre/post-production boards, people tags, History reports). Uses sample data kept in the browser. |

The website only knows the Apps Script URL (`CONFIG.API_URL` near the top of `index.html`). No passwords, keys or data are stored in this repo.

---

## Features

- **Board** (drag cards between stages) and **Table** views, filters by editor, channel, format and overdue
- **Stages:** Not started → Scripting → Editing → Internal review → Revisions → Client review → Approved → Delivered → Uploaded, plus On hold
- Several people on one video ("Also working on it"), including pod leads
- **Day slot** (Full day / 1st half / 2nd half) and a "Deadline missed by N days" line on late videos
- **Required fields** checked before saving; missing ones also turn red in the sheet
- **Admin dashboard** with workload per editor and a chart of who is at which stage
- **Activity log**: every change, who made it, where (website, sheet or QC tool)
- **QC tool** can push QC results into the right row with a QC key
- Light and dark mode (Graphite)

---

## Sign-in and roles

Everyone signs in with **their email + the password for their role**. Passwords are set by the main owner on the **Settings** page and stored only as salted SHA-256 hashes in the script's properties.

| Tab | Who | Can open |
|---|---|---|
| Team | Editors, content managers | Board, Table |
| Admin | Admins | + Admin dashboard, Activity |
| Admin | Owners (own email + own password) | + Access |
| Admin | Main owner | + Settings |

People stay signed in until they press **Sign out**. Sessions also end when that role's password is changed or when the owner uses **Sign everyone out**. Sign-in tokens are signed (HMAC) by the backend, and wrong passwords are rate-limited (8 tries, then 15 minutes).

---

## Updating the website

1. Edit `index.html` (or `prototype.html`).
2. Commit to `main`. GitHub Pages republishes in about a minute.
3. Open the site with `?v=<anything>` added to the URL to skip the browser cache and check the change.

To try the website without touching real data, set `API_URL: ""` in the `CONFIG` block and open the file locally: it runs in **demo mode** with sample data.

## Updating the backend (Apps Script)

1. Open the Google Sheet → **Extensions → Apps Script**.
2. Replace the code in `Code.gs` and save.
3. Run `selfTest` from the function menu; it adds, edits and deletes a test row and checks sign-in tokens. It must log `SELF-TEST PASSED`.
4. **Deploy → Manage deployments → Edit (pencil) → Version: New version → Deploy.** The web app URL stays the same, so the website needs no change.

Do **not** run `setupTracker` on the live sheet: it rebuilds the tabs from scratch.

---

## Custom domain (tracker.ytforbrands.com)

Already set up. For reference, or to set it up again:

1. **Squarespace** (ytforbrands.com → DNS → DNS Settings → Custom records): add `CNAME`, name `tracker`, data `saajidyt.github.io`.
2. **GitHub** (repo → Settings → Pages → Custom domain): enter `tracker.ytforbrands.com`, save, then tick **Enforce HTTPS** once the check passes.
3. Keep the `CNAME` file that GitHub adds to this repo.

Everyone signs in once again after the address changes, because browsers keep sign-ins per address.

---

## Security notes

- The repo holds only the website code. Data and passwords live in Google and never in this repo.
- Never commit the Apps Script code with passwords in it, or paste passwords into issues or commits.
- If a role password leaks, change it on the Settings page: everyone using it is signed out.
- Anyone with edit access to the Google Sheet can see every tab, including Activity. Keep sheet access limited to owners and admins.

## Cost

GitHub Pages and Google Apps Script are free at this size. Keeping this repo **private** with Pages needs a GitHub Pro plan.

---

Maintained by Saajid (YT for Brands).
