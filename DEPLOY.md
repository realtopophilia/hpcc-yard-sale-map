# Deploying the Yard Sale Map with Claude Code

## One-time setup (you)

- Install Claude Code if you haven't: `npm install -g @anthropic-ai/claude-code`
- Install the GitHub CLI (`winget install GitHub.cli` on Windows) and log in: `gh auth login`
- Open a terminal in this folder (`yardsale-site/`) and run `claude`

## The prompt — paste this into Claude Code

> Read CLAUDE.md first. This folder contains a finished single-file static site (index.html) for the Highland Park Yard Sale. Deploy it to GitHub Pages:
>
> 1. git init, commit all files with message "Initial release: HPCC yard sale map"
> 2. Create a public GitHub repo named "hpcc-yard-sale-map" under my account using `gh repo create`, and push main.
> 3. Enable GitHub Pages serving from the main branch root using `gh api repos/{owner}/hpcc-yard-sale-map/pages -X POST -f "source[branch]=main" -f "source[path]=/"`.
> 4. Wait for the Pages build, then verify: curl the published URL and confirm it returns HTTP 200 and the response body contains "Highland Park Neighborhood Yard Sale 2026".
> 5. Give me the final public URL and a QR code I can put on flyers (generate qr.png pointing at the URL using the `qrencode` package or a Python library).
>
> Do not modify index.html unless something blocks deployment, and tell me if you change anything.

## After it's live

The URL will be `https://<your-github-username>.github.io/hpcc-yard-sale-map/`

Updates are just: edit index.html → commit → push. Pages redeploys automatically in ~1 minute. Or tell Claude Code what to change and let it push — CLAUDE.md gives it everything it needs to edit safely.

## Content updates to ask Claude Code for later

- "Set the printable map URL to <link>" (replaces `#PRINTABLE_MAP_URL`)
- "Add the music stage schedule: 12pm Band A, 1:30pm Band B…" (fills `FESTIVAL_SCHEDULE`)
- "Add these flea market vendors to the festival stop: …"
- "Re-show the welcome modal to everyone" (bumps the `hpccWelcomeSeen_v2` key)

## Day-of note

GitHub Pages is static hosting with a global CDN — it will comfortably absorb the whole neighborhood hitting it at once. No server, nothing to babysit.
