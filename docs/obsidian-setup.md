# Obsidian + GitHub setup

How to turn this repo into your living knowledge base — edit in Obsidian, sync to GitHub, publish via GitHub Pages.

## One-time setup (5 minutes)

### 1. Install Obsidian

You should already have the installer downloaded (`Obsidian-*.exe`). Run it, follow the default prompts.

### 2. Clone the repo locally

If you have Git installed:

```bash
cd C:\Users\user\Documents
git clone https://github.com/FirstStepToGithub/shelfvision-docs.git
```

If you don't have Git: download the repo ZIP from GitHub → extract to `C:\Users\user\Documents\shelfvision-docs`.

### 3. Open the `/vault` folder as an Obsidian vault

- Launch Obsidian.
- Click **"Open folder as vault"**.
- Select `C:\Users\user\Documents\shelfvision-docs\vault`.
- Obsidian will create a `.obsidian` folder (config) and open the vault.

### 4. Install the `obsidian-git` plugin

- Settings → Community plugins → **Turn on community plugins** (accept the dialog).
- Browse → search **"Obsidian Git"** by Vinzent03 → Install → Enable.
- Settings → Obsidian Git:
  - **Vault backup interval (minutes):** 10
  - **Auto pull interval (minutes):** 10
  - **Commit message:** `vault: {{date}}`
  - **Sync Method:** `Merge`

That's it — every 10 minutes, Obsidian auto-commits and pushes your notes.

### 5. Install recommended plugins

Optional but very useful:

- **Dataview** — query notes like a database (e.g. "all notes tagged #decision, sorted by date").
- **Templater** — dynamic note templates (see `/vault/_templates/`).
- **Excalidraw** — hand-drawn architecture diagrams inside Obsidian.
- **Tasks** — turn checkboxes into a queryable task list.

## Daily workflow

1. Open Obsidian → write / think / link notes.
2. Obsidian-git auto-commits every 10 min.
3. The `/docs/*.md` files you edit are the same files served by GitHub Pages — changes go live within ~1 min of push.

## Website publishing

The website is served from the `main` branch root (`/index.html`). To turn on GitHub Pages:

1. Go to https://github.com/FirstStepToGithub/shelfvision-docs/settings/pages
2. **Source:** Deploy from a branch → **Branch:** `main` / `(root)` → Save.
3. Wait ~1 minute. Your site is live at https://firststeptogithub.github.io/shelfvision-docs/

## Vault structure

```
vault/
├── Home.md                       # Start here, linked from all major notes
├── 01-Architecture/
│   └── System overview.md
├── 02-Roadmap/
│   └── Current sprint.md
├── 03-Decisions/
│   └── YYYY-MM-DD-<slug>.md
├── 04-Daily-Notes/
│   └── 2026-04-21.md
├── 05-Reference/
│   ├── Glossary.md
│   └── Links.md
├── _templates/
│   ├── Decision.md
│   ├── Daily.md
│   └── Meeting.md
└── .obsidian/                    # Obsidian config (safe to commit)
```

## Conflict handling

Obsidian-git uses merge by default. If you edit the same note from two devices and push both, you'll get a merge conflict inside the `.md` file (the standard `<<<<<<<` markers). Resolve manually, commit.

## Privacy

The repo is public, which is fine for ShelfVision docs. **Don't paste secrets into notes** — Roboflow API key, Google credentials, etc. If you need private notes, keep them in a separate, non-synced folder or a second private repo.

## Troubleshooting

| Issue | Fix |
|---|---|
| "Authentication failed" on push | Install GitHub CLI (`gh auth login`) or set up a Personal Access Token in Obsidian-git settings |
| Merge conflicts on every commit | Check you're not running two Obsidian instances; lower auto-interval |
| Obsidian-git can't find Git | Set path to `git.exe` explicitly in plugin settings (usually `C:\Program Files\Git\cmd\git.exe`) |
| GitHub Pages 404 | Wait 2 min after first push; verify the Pages settings |
