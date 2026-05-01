# Contributing to the Wiki

This wiki is maintained collaboratively. Anyone in the lab can contribute — no web development experience needed.

---

## Option A — Quick edits on GitHub (simplest)

For small fixes (typos, outdated info, adding a link):

1. Find the page on the wiki site
2. Click the **edit** icon (pencil) in the top right of any page
3. Edit the markdown directly in GitHub's editor
4. Click **Commit changes** — the site rebuilds automatically in ~1 minute

---

## Option B — Larger edits locally

For new pages or significant rewrites:

### Setup (one-time)

```bash
# Clone the repo
git clone https://github.com/MonashRobotics/wiki
cd wiki

# Install MkDocs Material
pip install mkdocs-material

# Preview locally (live reload on save)
mkdocs serve
```

Open http://127.0.0.1:8000 to preview. The site updates as you save files.

### Workflow

```bash
# Edit files in docs/
# Then push when ready
git add -A
git commit -m "Brief description of what changed"
git push
```

The site rebuilds automatically on every push to `main`.

---

## Option C — AI-assisted (recommended for new guides)

Use an AI assistant (Claude, ChatGPT, etc.) to help draft or improve content:

1. **Give the AI context:** paste in the relevant raw notes, terminal output, or documentation you want turned into a wiki page
2. **Ask it to write the page** in the style of the existing wiki (clear headings, code blocks for commands, admonition boxes for warnings)
3. **Review and edit** the output — AI drafts are a starting point, not final copy
4. **Paste into the right file** under `docs/` and push

### Tips for prompting

- *"Write a wiki page for students on how to use X. Audience: final-year undergrad students with basic Linux knowledge. Format: MkDocs markdown with admonition boxes for warnings."*
- *"Improve this draft — make it clearer and more concise, keep all the commands"*
- *"This guide has private admin details mixed in — extract only the student-facing content"*

### Using Claude Code specifically

If you have [Claude Code](https://claude.ai/code) set up locally:

```
# In the wiki repo directory, ask Claude to:
# - Draft a new page from your notes
# - Update an existing page with new information
# - Check consistency across pages
```

Claude Code can read the existing wiki files for context, draft new content, and commit directly — ask your lab admin to show you the workflow.

---

## Structure

```
docs/
  index.md          ← home page
  contributing.md   ← this page
  onboarding/       ← getting started guides
  hpc/              ← HPC system guides
```

Add new pages in the right folder, then register them in `mkdocs.yml` under `nav:`.

---

## Style guide

- Write for a **final-year undergraduate** — assume Linux basics, no HPC experience
- Use **code blocks** for every command (never inline)
- Use **admonition boxes** for warnings and tips:

```markdown
!!! warning
    Scratch storage is purged after 21 days — save important data elsewhere.

!!! tip
    Run `mkdocs serve` to preview changes before pushing.
```

- Keep pages **focused** — one system or topic per page
- Link between pages rather than duplicating content
