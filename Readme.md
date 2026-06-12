# Drive Sheets Dashboard

A static dashboard that lists all Google Sheets in a Drive folder (and subfolders),
grouped by Year → Month → Date, with single-file `.xlsx` downloads and
"download all" ZIP buttons per year/month/date. Auto-refreshes data every 2 hours.

## How it works

- **`AppsScript_DriveSheetsScanner.gs`** — runs in Google Apps Script under YOUR
  Google account (has Drive access). Every 2 hours it scans the target folder
  recursively, builds `data.json`, and commits it to your GitHub repo.
- **`index.html`** — static page for GitHub Pages. Reads `data.json` from the
  same repo and renders the dashboard. Downloads pull directly from Google's
  export endpoint (`.../export?format=xlsx`), so files must remain shared
  appropriately (see Sharing section).

## Setup steps

### 1. Create the GitHub repo
- Create a new repo, e.g. `sheet-index`.
- Upload `index.html` to the repo root.
- Enable **GitHub Pages**: Settings → Pages → Source: `main` branch, `/ (root)`.
- Your dashboard will be live at `https://<username>.github.io/<repo>/`.

### 2. Create a GitHub Personal Access Token
- GitHub → Settings → Developer settings → Fine-grained tokens → Generate new token.
- Repository access: only the `sheet-index` repo.
- Permissions: **Contents: Read and write**.
- Copy the token — you'll paste it into the Apps Script config.

### 3. Set up the Apps Script
- Go to [script.google.com](https://script.google.com) → New project.
- Paste in the contents of `AppsScript_DriveSheetsScanner.gs`.
- Fill in the `CONFIG` section at the top:
  - `ROOT_FOLDER_ID` — open your target Drive folder, copy the ID from the URL
    (`https://drive.google.com/drive/folders/THIS_PART`)
  - `GITHUB_OWNER`, `GITHUB_REPO`, `GITHUB_BRANCH`, `GITHUB_TOKEN`
- Run `setup` once — this prompts Google's OAuth consent screen (grant Drive access).
- Run `pushDriveIndex` once manually — check the repo for a new `data.json`.
- Run `createTrigger` once — sets up the every-2-hours automatic refresh.

### 4. Sharing / downloads
For the `.xlsx` download links and the "download all" ZIP buttons to work for
viewers of the dashboard, each Google Sheet needs to be shared at least as
**"Anyone with the link – Viewer"**. If your sheets are private to your org,
the dashboard itself can still be private (e.g. only you visit it) — the
export links will work for you while logged into the right Google account.

If you need this to work for arbitrary outside visitors without exposing
sheet contents broadly, you'd need a small backend proxy — let me know and
I can adjust the approach.

## Notes
- "Created date" comes from `DriveApp` file creation timestamp (the date the
  Google Sheet was created, not last modified).
- Folder path shown under each sheet name shows where in the folder tree it lives.
- The 2-hour refresh is handled entirely by Apps Script's time-based trigger —
  no server needed.
