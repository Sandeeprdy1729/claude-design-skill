# Self-Healing CLAUDE.md Generator

**Stop re-teaching Claude the same things every morning.**

Users describe the same pattern: every new session, they re-explain the build command,
the test flags, the "don't edit that file" rules. This skill captures those patterns
once, writes them to disk as `CLAUDE.md` files, and patches them automatically
whenever Claude makes a mistake that better context would have prevented.

---

## The problem

`CLAUDE.md` is Claude's persistent memory for a project. When it is missing or
incomplete, Claude guesses — and guesses wrong in predictable, repeatable ways.

The fix is not writing a massive documentation file. It is capturing the minimal,
highest-signal context that prevents the specific mistakes Claude actually makes in
your project. This skill makes that systematic rather than manual.

---

## How it works

```text
Session 1: Claude runs `npm test` instead of `make test` and it fails.
You: /patch "Claude ran npm test, should use make test — npm is a stub"
Skill: adds the entry to CLAUDE.md with provenance comment.

Session 2: Claude reads the patched CLAUDE.md.
Claude runs `make test`. Correctly, every time after.
```

Every patch is shown for review before anything is written. The skill never silently
modifies files.

---

## Workflow

```text
Project files (package.json, Makefile, .env.example, CI workflows, …)
    │
    ├─ Phase 1: Scan — extract build commands, test flags, conventions, gotchas
    ├─ Phase 2: Gap Detection — what's missing vs what's already documented
    ├─ Phase 3: Structure Planning — root (≤50 lines) + subfolder files
    ├─ Phase 4: Generate / Patch — write with provenance, preserve manual edits
    └─ Phase 5: Validate — line count, dead links, stale commands, no duplicates
```

---

## Progressive disclosure

Root `CLAUDE.md` is hard-capped at **50 lines**. It contains only:

- The primary build command
- The primary test command (with all required flags)
- 3–5 project-wide gotchas
- Links to subfolder `CLAUDE.md` files for deeper context

Everything else lives in the subfolder closest to where it applies:

| Subfolder | What goes there |
| --- | --- |
| `src/db/CLAUDE.md` | Migration commands, ORM patterns |
| `src/api/CLAUDE.md` | Endpoint conventions, auth, versioning |
| `src/components/CLAUDE.md` | Component structure, styling, a11y rules |
| `packages/CLAUDE.md` | Cross-package import rules, change policy |
| `infra/CLAUDE.md` | Deploy flow, environment map |
| `scripts/CLAUDE.md` | What each script does, when to run it |

---

## What the scan detects

| Category | Sources scanned |
| --- | --- |
| Build & run commands | `package.json`, `Makefile`, `justfile`, `Dockerfile`, CI workflows |
| Test commands + flags | `jest.config.*`, `pytest.ini`, `pyproject.toml`, CI test jobs |
| File conventions | Casing patterns, barrel files, co-located vs separate test dirs |
| Environment setup | `.env.example`, `.nvmrc`, `.python-version`, `docker-compose.yml` |
| Architecture + gotchas | Generated dirs, deprecated files, layer boundaries, monorepo rules |

---

## Mistake categories

When you `/patch` a mistake, the skill classifies it to pick the right entry format
and target file:

| Code | Category | Example |
| --- | --- | --- |
| `M1` | Wrong command | `npm test` instead of `make test` |
| `M2` | Missing flag | `pytest` without `--tb=short -x` |
| `M3` | Wrong file | Editing `prisma/migrations/` directly |
| `M4` | Wrong location | New component in `src/` instead of `src/features/` |
| `M5` | Missing setup | Forgot `docker compose up -d` before running tests |
| `M6` | Deprecated path | Used old `src/utils/api.ts` instead of `src/lib/client.ts` |
| `M7` | Convention violation | `snake_case` file in a `kebab-case` project |
| `M8` | Architecture violation | Repository called directly from a controller |
| `M9` | Env var assumption | Assumed `DATABASE_URL` without checking `.env.example` |
| `M10` | Scope creep | Edited `packages/shared` when task was in `packages/app` |

---

## Sample audit output

```text
CLAUDE.MD COVERAGE AUDIT
────────────────────────────────────────────────────────────
  Patterns detected     : 31
  Patterns documented   : 23  (74%)
  ─────────────────────────────────────────────────────────
  GAPS (8)
    ✗ Test flags — jest.config.js has testTimeout: 30000 (undocumented)
    ✗ Generated files — src/generated/ has no "do not edit" note
    ✗ Env var DATABASE_URL — in .env.example, not in CLAUDE.md
    ✗ Pre-commit hook — husky installed, no mention
    ✗ Barrel files — src/components/index.ts pattern undocumented
    ✗ Lint command — eslint script in package.json, no entry
    ✗ Feature flag location — src/flags/ detected, no convention note
    ✗ API versioning — v1 deprecated, v2 active — not documented

  ACTION: run /generate to fill all 8 gaps
────────────────────────────────────────────────────────────
```

---

## Slash commands

| Command | What it does |
| --- | --- |
| `/scan [path]` | Extract all detectable patterns from the project tree |
| `/generate` | Write a full CLAUDE.md file set from scan results |
| `/patch "<mistake>"` | Add the missing entry that would have prevented a mistake |
| `/audit` | Coverage report — gaps, stale entries, bloat |
| `/diff` | What a fresh scan would add, update, or remove |
| `/promote <subfolder>` | Move a high-frequency subfolder entry up to root |
| `/demote <entry>` | Move a root entry to its subfolder to free root space |
| `/trim` | Remove entries that no longer apply (with confirmation) |
| `/explain <entry>` | Show the originating mistake or scan source for an entry |
| `/reset` | Regenerate everything from scratch (preserves manual entries) |

---

## Provenance tracking

Every generated or patched entry carries a comment so it can be validated,
diffed, and pruned:

```markdown
<!-- patched: claude-md-generator · M2 missing-flag · 2026-05-19 -->
Always run `pytest -x --tb=short --timeout=30` — the bare `pytest` hangs on
integration tests due to a missing timeout in the default config.
```

Content without a provenance comment is treated as manually written and never
overwritten.

---

## Monorepo support

Detected from `pnpm-workspace.yaml`, `packages/`, `apps/`, or `libs/` layout.
Generates a workspace root `CLAUDE.md` (workspace-level only, ≤50 lines) plus
one file per package/app with package-specific context.

---

## Installation

1. Add `skills/claude-md-generator/SKILL.md` to your Claude project.
2. Say "generate CLAUDE.md for this project" or run `/scan` to start.
3. After any session where Claude makes a repeatable mistake, run `/patch` with
   a one-sentence description of what went wrong.
