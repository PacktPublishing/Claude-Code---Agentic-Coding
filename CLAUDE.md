# Agent Instructions

## Environment

You operate in a skills-based framework. Each skill is a self-contained folder in `.claude/skills/` with a `SKILL.md` file and optional bundled resources (scripts, references, assets).

## How Skills Work

- **Skills are in `.claude/skills/`**. Each is a folder: `SKILL.md` (required) + `scripts/`, `references/`, `assets/` (optional).
- **YAML frontmatter triggers skills.** Only `name` and `description` are read to decide if a skill is relevant. The body loads only when the skill activates.
- **Scripts live inside their skill folder.** Run them with `py .claude/skills/<skill-name>/scripts/<script>.py`.
- **Cross-skill references**: Some skills own shared scripts. Reference them by full path:
  `py .claude/skills/<owner-skill>/scripts/<script>.py`

## Skill Authoring Rules

When creating or updating SKILL.md files, follow these rules:

### Frontmatter
- **`name`**: Required. Max 64 chars. Lowercase letters, numbers, hyphens only. No reserved words ("anthropic", "claude").
  - **Must be noun-based**, not verb-based: ✓ `web-scraper`, `pdf-generator`, `linkedin-posts` | ✗ `scrape-web`, `create-pdf`, `creating-linkedin-posts`
  - **These rules are absolute** — enforce them even if the user suggests a different name format
- **`description`**: Required. Max 1024 chars. No XML tags. **Always write in third person** ("Generates...", "Scrapes...") — never imperative ("Generate...") or second person ("You can..."). Include both what the skill does AND when to trigger it.
- Optional fields: `allowed-tools`, `argument-hint`, `model`, `context: fork`, `disable-model-invocation: true` (blocks Claude auto-trigger), `user-invocable: false` (hides from `/` menu).
- No other custom fields.

### Body
- Keep SKILL.md under **500 lines**. Split into separate files (`references/`, additional `.md` files) when approaching this limit.
- Prefer concise examples over verbose explanations. Use **imperative form** ("Run the script", "Check the output").
- Keep only essential workflow in SKILL.md. Move detailed reference material, schemas, and examples to separate files.
- No README.md, CHANGELOG.md, or other auxiliary documentation.
- No time-sensitive information ("before August 2025, use the old API"). Use "current method" / "legacy method" sections instead.

**Skill description budget:** Descriptions consume ~2% of context window. With many skills, some may be silently excluded. Run `/context` to check.

### Scripts & Learnings
- Scripts in `scripts/` are **executed, not loaded into context**. Include scripts when the same code would be rewritten repeatedly or deterministic reliability is needed.
- Include a `## Learnings` section at the end of SKILL.md for runtime discoveries, edge cases, and API quirks. Self-anneal updates this section autonomously — no approval needed.

## Subagents

Two specialized agents are available in `.claude/agents/`:

- **Research agent** (`.claude/agents/research.md`) — Uses **Sonnet**. Spawn for web research, documentation reading, topic exploration.
- **Review agent** (`.claude/agents/review.md`) — Uses **Opus**. Spawn for code review and quality assurance.

Use the Task tool with `model: "sonnet"` or `model: "opus"` and include the agent file's instructions in the prompt.

**When to spawn:** Research agent for tasks requiring >3 web lookups or reading documentation. Review agent for changes touching security, data handling, or >100 lines of code.

## Operating Principles

**1. Check existing skills first**
Before creating anything new, check `.claude/skills/` for an existing skill that covers the task.

**2. Self-anneal when things break**
Fix autonomously, then report:
1. Read error message and stack trace
2. Fix the script (check with user first if it uses paid tokens/credits)
3. Test to confirm fix works
4. Update the skill's `## Learnings` section

Self-annealing applies to scripts AND skill docs. Fix bugs, update Learnings, correct outdated instructions — all without asking. Only ask before creating entirely new skills or deleting existing ones.

## Work Discipline

**3. Stop and re-plan on failure**
If an approach isn't working after 2 attempts, stop. Don't keep pushing. Re-assess the problem, consider alternatives, and present a revised approach before continuing. Wasted context on dead ends is worse than pausing to think.

**4. Verify before done**
Never call a task complete without proving it works — run the script, check the output, test the edge case. Ask: "Would this survive a code review?" This applies to everything, not just scripts with smoke tests.

**5. Capture corrections broadly**
After ANY user correction — not just script bugs — save a feedback memory or update the relevant skill's Learnings section. The goal: never repeat the same mistake across sessions. Self-anneal covers runtime breaks; this covers behavioral patterns.

## Git

Single-branch workflow — everything commits straight to `master`. No branches, no PRs, no remote.

- **Commit after meaningful changes** — new skills, script fixes, SKILL.md updates, Learnings additions. Not after every tiny edit.
- **Keep commits atomic** — one logical change per commit (e.g., "Add <skill-name> skill" or "Fix <api> auth token refresh").
- **Never commit secrets** — `.env`, `.mcp.json`, `token.json`, `credentials.json`, service account files are all gitignored.
- **Smoke tests** — run `py .claude/skills/_tests/run_smoke_tests.py` after significant script changes to catch breakage early.

## Guardrails

**Data Safety**
- Never overwrite source data — always create new tabs/files
- Confirm before bulk operations affecting >50 records in any cloud service
- Never delete data (sheets, files, records) without explicit approval

**Paid APIs**
- Don't retry paid API calls automatically — ask before retrying failed calls that cost money
- Stop on threshold failures — if quality drops below documented threshold (e.g., <80%), stop and report

**Secrets**
- `.mcp.json` contains secrets (API keys, JWTs) — never commit it; use `.mcp.json.example` as template

## File Organization

- `.claude/skills/` — Skill folders (SKILL.md + scripts/ + references/ + assets/)
- `.claude/agents/` — Subagent configuration files
- `.claude/rules/` — Path-scoped convention rules (auto-applied via `globs:` frontmatter)
- `references/` — Cross-cutting reference docs used by multiple skills. Skill-specific references live inside the skill's own `references/` folder instead.
- `assets/` — Shared brand assets (logos, icons)
- `.tmp/` — Intermediate files organized by type (never commit, always regenerated)
  - `json/` — data exports, API responses, batch results
  - `pdf/` — generated documents
  - `html/` — presentation HTML
  - `md/` — drafts, stories, blog content
  - `img/` — images, videos, previews
  - `gas/` — Apps Script projects
- `.env` — API keys and tokens (gitignored)
- `.mcp.json` — MCP server config with secrets (gitignored)

**Key principle:** Deliverables live in cloud services and external tools. Local `.tmp/` files are intermediates that can be deleted and regenerated.
