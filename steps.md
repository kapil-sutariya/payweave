# Publishing PayWeave Skill to skills.sh

## Step 1 — GitHub repository

The skill lives at: **https://github.com/kapil-sutariya/payweave**

The repo root must contain `SKILL.md` directly (not nested in a subfolder).

```bash
# Push latest changes
git add .
git commit -m "feat: add PayWeave agent skill"
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
# Install from local path (all agents)
npx skills add .

# Install targeting Claude Code specifically
npx skills add . -a claude-code

# Or install from GitHub after pushing
npx skills add kapil-sutariya/payweave -a claude-code
```

Confirm the skill appears in your agent:
- **Claude Code** — type `/payweave` in the prompt bar; it should appear in the autocomplete menu
- **Cursor** — check Settings → Skills
- **Cline** — check Skills panel

---

## Step 3b — How consumers install the skill (Claude Code)

Once published to GitHub, consumers install the skill with a single command:

```bash
# Project-scoped (installs to .claude/skills/ in current directory)
npx skills add kapil-sutariya/payweave -a claude-code

# Global (available in all projects)
npx skills add kapil-sutariya/payweave -a claude-code -g
```

After installation, restart Claude Code and type `/payweave` in the prompt bar.

---

## Step 4 — Publish to skills.sh

1. Go to **https://skills.sh**
2. Click **Submit a skill** (or **Add repository**)
3. Enter your GitHub repository URL: `https://github.com/kapil-sutariya/payweave`
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

Update to:
```ts
const skillInstall = `npx skills add kapil-sutariya/payweave -a claude-code`;
```

---

## Ongoing maintenance

| Action | What to do |
|--------|-----------|
| Update tool docs | Edit `SKILL.md` body and push to GitHub — skills.sh re-crawls periodically |
| Update API reference | Edit the Discovery API section in `SKILL.md` and push |
| Change MCP URL | Update `SKILL.md` and re-submit if needed |
| Add new tools | Add a new section to `SKILL.md` under `## MCP Tools` |
