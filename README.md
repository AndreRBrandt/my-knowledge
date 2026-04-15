# my-knowledge

Personal knowledge vault, structured for **agentic context engineering**.

## Quick start

### First time setup
1. Clone: `git clone git@github.com:AndreRBrandt/my-knowledge.git`
2. Open folder in Obsidian: "Open folder as vault"
3. Install community plugin "Obsidian Git"
4. Configure auto-commit + auto-push (e.g., every 10 minutes)

### Entry point
Always start from [[00-Index/HOME]].

### Rules
See [[00-Index/Operating Principles]] — canonical ruleset for using this vault.

## Structure

```
my-knowledge/
├── 00-Index/          → MOCs, HOME, rules — navigation entry
├── 10-Domain/         → Business knowledge (cross-project)
├── 20-Architecture/   → Technical knowledge (cross-project)
├── 30-Decisions/      → ADRs
├── 40-Projects/       → Thin project hubs (curated links only)
├── 50-Sessions/       → Work session logs
├── 60-References/     → External learning (books, articles)
├── 70-Templates/      → Note templates by type
└── _attachments/      → Images, diagrams, PDFs
```

## Sync strategy

Git-based sync via this GitHub repo. No Obsidian Sync subscription needed.

- Plugin: [Obsidian Git](https://github.com/denolehov/obsidian-git)
- Auto-pull on startup, auto-commit every 10min, auto-push on commit
- Conflicts: resolved via Git merge (rare with single user)

## License / Privacy

Private repository. All content is personal.
