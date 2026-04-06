# Publishing PayWeave Skill to skills.sh

## Step 1 — Move to a separate GitHub repository

Create a new public repository (e.g. `github.com/payweave/payweave-skill`) and copy the contents of this folder into it:

```
payweave-skill/
├── SKILL.md
└── steps.md
```

The repo root should contain `SKILL.md` directly (not nested in a subfolder).

```bash
# Example: initialise and push
git init
git add .
git commit -m "feat: add PayWeave agent skill"
git remote add origin git@github.com:payweave/payweave-skill.git
git push -u origin main
```

---

## Step 2 — Validate the skill locally

```bash
npx skills-ref validate .
```

Expected output: `✓ payweave — valid skill`.

---

## Step 3 — Test installation locally

```bash
# Install from local path
npx skills add .

# Or install from GitHub after pushing
npx skills add payweave/payweave-skill
```

Confirm the skill appears in your agent:
- **Claude Code** — type `/payweave` in the prompt bar; it should appear in the autocomplete menu
- **Cursor** — check Settings → Skills
- **Cline** — check Skills panel

---

## Step 4 — Publish to skills.sh

1. Go to **https://skills.sh**
2. Click **Submit a skill** (or **Add repository**)
3. Enter your GitHub repository URL: `https://github.com/payweave/payweave-skill`
4. skills.sh will auto-detect `SKILL.md` and register the skill

> skills.sh tracks install counts automatically — no additional publish command is needed. The skill appears in the marketplace once the repo is registered and at least one installation is recorded.

---

## Step 5 — Update the MCP URL (before publishing)

Replace the placeholder URL in `SKILL.md` with the production MCP server URL. Find and update this line:

```
# Current (placeholder)
claude mcp add payweave --transport http https://api.payweave.app/mcp

# Update to actual production URL if different
claude mcp add payweave --transport http https://<your-actual-mcp-domain>/mcp
```

---

## Step 6 — (Optional) Add to the Connect Agent modal

Once the skill repo is public, update the `npx skills add` command in the discovery page modal:

**File:** `apps/platform/web/src/routes/portal/discovery.tsx`

Find the line:
```ts
const skillInstall = `npx skills add payweave/payweave`;
```

Update to match your actual org/repo:
```ts
const skillInstall = `npx skills add payweave/payweave-skill`;
```

---

## Ongoing maintenance

| Action | What to do |
|--------|-----------|
| Update tool docs | Edit `SKILL.md` body and push to GitHub — skills.sh re-crawls periodically |
| Update API reference | Edit the Discovery API section in `SKILL.md` and push |
| Change MCP URL | Update `SKILL.md` and re-submit if needed |
| Add new tools | Add a new section to `SKILL.md` under `## MCP Tools` |
