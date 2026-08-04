# CLAUDE.md: Make Claude Code Truly Understand Your Project

> 📅 2026-07-27 · Getting Started
> A deep dive into how CLAUDE.md lets Claude Code understand your project's conventions and preferences.

---

## CLAUDE.md: Make Claude Code Truly Understand Your Project

### What is CLAUDE.md

CLAUDE.md is Claude Code's project memory file. When you place a `CLAUDE.md` in your project root, Claude Code reads it automatically on every startup, understanding this project's conventions, tech stack, common commands, and boundaries.

Many times, the problem of AI "not listening" isn't that the model isn't smart enough — it's that the system never received clear project constraints.

***

### A template you can use right away

```markdown
# Project Description

- This is a Next.js + TypeScript project
- Styling uses Tailwind CSS
- Pages live in the app/ directory
- Shared components live in the components/ directory

# Development Conventions

- Reuse existing components first
- Don't add dependencies casually
- Use camelCase for variable naming
- Run a build check after making changes

# Common Commands

- Install dependencies: npm install
- Local development: npm run dev
- Production build: npm run build

# Notes

- Don't modify the legacy/ directory
- For anything involving payment logic, present a plan first — don't change it directly
```

***

### One principle for writing CLAUDE.md

**Don't write filler. Write content that actually influences behavior.**

| Worth writing | Not worth writing |
|--------|----------|
| Must run `npm run build` after changes | We value code quality |
| No new dependencies unless you explain why first | Keep the code clean |
| Form components always reuse `components/forms` | This is a Next.js project |

CLAUDE.md isn't an introduction blurb; it's the long-term working manual for Claude Code in this project. The more specific it is, and the closer it matches real constraints, the more consistent Claude Code's behavior will be.

***

### Advanced tips

**Layered structure:**

* Global CLAUDE.md (`~/.claude/CLAUDE.md`) — your personal general conventions, applies to all projects
* Project CLAUDE.md (`<project root>/CLAUDE.md`) — applies only to the current project

**Common uses:**

* Define the tech stack and framework versions
* Record special dependencies and environment variables
* Specify code style preferences
* Mark directories or files that shouldn't be modified
* Write down solutions to common problems
