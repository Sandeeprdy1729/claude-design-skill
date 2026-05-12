# pr-review

> Automated code review through six parallel lenses. Every finding is severity-graded,
> file-precise, and comes with a ready-to-apply fix.

---

## What it does

Manual code review is slow, inconsistent, and misses things. `pr-review` replaces the
back-and-forth by fanning out six specialised sub-agents simultaneously — each focused
on a single quality dimension — then synthesising a single structured report with
actionable suggestions and severity levels.

Paste a diff, a list of file paths, or a description of what changed. Get back a report
in seconds that a senior engineer would be proud to sign off on.

---

## The six lenses

Each lens runs in parallel. Coverage is deep because every agent has a single job.

| Lens | What it catches |
| --- | --- |
| **Security** | OWASP Top 10, injection, hardcoded secrets, weak crypto, IDOR, SSRF, supply chain |
| **Architecture** | Coupling, complexity, N+1 queries, dead code, error handling, DRY violations |
| **Dependencies** | Outdated packages, CVEs, license conflicts, lockfile hygiene, bundle weight |
| **Logging** | Missing logs, PII leakage, wrong log levels, missing trace IDs, silent failures |
| **Testing** | Coverage gaps, happy-path-only tests, flaky tests, brittle selectors, weak assertions |
| **Documentation** | Missing docstrings, stale comments, README gaps, type annotations, TODO hygiene |

---

## Output format

Every finding is structured, precise, and actionable — never a vague observation.

```text
═══════════════════════════════════════════════════════════
  PR REVIEW REPORT
  Scope   : 4 files changed, +312 −47 lines
  Lenses  : Security · Architecture · Dependencies · Logging · Testing · Docs
  Summary : 1 critical  3 high  5 medium  8 low
═══════════════════════════════════════════════════════════

┌─ [SEC-001] 🔴 P0 Critical · Security
│  File    : src/api/users.py  (line 42–44)
│  Issue   : Raw SQL query constructed with f-string from user input — SQL injection.
│  Impact  : Attacker can dump or delete the entire users table.
│  Fix     :
│            # Before
│            db.execute(f"SELECT * FROM users WHERE id = {user_id}")
│            # After
│            db.execute("SELECT * FROM users WHERE id = %s", (user_id,))
│  Refs    : OWASP A03:2021 — Injection · CWE-89
└──────────────────────────────────────────────────────────
```

### Severity levels

| Level | Meaning | Expected action |
| --- | --- | --- |
| 🔴 **P0 Critical** | Exploitable, data-loss, or outage risk | Block merge immediately |
| 🟠 **P1 High** | Significant bug or security gap | Fix before merge |
| 🟡 **P2 Medium** | Quality or maintainability debt | Fix in follow-up PR |
| 🔵 **P3 Low** | Style, minor inconsistency, nice-to-have | Backlog or skip |

Findings tagged by two or more lenses are automatically promoted one severity level.

---

## Slash commands

```text
/review              Full review across all six lenses
/security            Security lens only
/arch                Architecture lens only
/deps                Dependency lens only
/logging             Logging & observability lens only
/testing             Testing lens only
/docs                Documentation lens only
/fix <issue-id>      Generate a ready-to-apply patch for a specific finding
/issues              Open GitHub issues for every P0 and P1 finding
/summary             One-paragraph executive summary of the last report
/diff                New/resolved findings vs the previous review on the same files
```

---

## GitHub integration

Run `/issues` after a report to automatically open a labelled GitHub issue for every
P0 and P1 finding. Each issue includes the problem description, impact, and a
before/after code snippet. Issues are labelled `bug` (P0), `security` (security
findings), or `tech-debt` (P2/P3).

---

## How the workflow runs

```text
Input (diff / file paths / plan / repo)
    │
    ├─ Phase 1: Reconnaissance
    │     Read files, detect language + framework, map entry points,
    │     read PR description and commit messages.
    │
    ├─ Phase 2: Fan-out  (parallel)
    │     ├─ Lens A: Security
    │     ├─ Lens B: Architecture
    │     ├─ Lens C: Dependencies
    │     ├─ Lens D: Logging & Observability
    │     ├─ Lens E: Testing
    │     └─ Lens F: Documentation
    │
    ├─ Phase 3: Synthesis
    │     Merge findings, deduplicate cross-lens findings,
    │     promote severity when multiple lenses agree.
    │
    └─ Phase 4: Report
          Structured output + optional GitHub issue creation.
```

---

## Language-specific rules

The skill extends its checklist automatically based on the detected language.

| Language | Extra checks |
| --- | --- |
| Python | Type annotations, mutable default args, `except Exception` breadth |
| TypeScript / JS | `any` overuse, missing `null` checks, `async` without `await` |
| Java / Kotlin | Swallowed checked exceptions, thread-safety, `equals`/`hashCode` contract |
| Go | Ignored error returns, goroutine leaks, unbuffered channels in hot paths |
| SQL | Non-SARGable predicates, implicit type coercions, missing `WHERE` on mutations |
| Terraform / IaC | Public S3 buckets, wildcard IAM, unencrypted storage, missing state locking |

---

## Usage

**Full review from a diff:**

> Paste your diff or file paths and say "review this PR."
> Claude will confirm: *"PR Review skill active. Running all six lenses…"*

**Targeted scan:**

> `/security src/payments/` — runs only the Security lens on the payments directory.

**Apply a fix:**

> `/fix SEC-001` — Claude outputs a unified diff patch ready to apply.

**Open GitHub issues:**

> `/issues` — opens one labelled issue per P0/P1 finding with full context.

---

## Guarantees

- Every finding names the exact file and line range — no vague feedback.
- Every finding includes a before/after code snippet when a fix is suggested.
- Findings are never duplicated across lenses — cross-lens matches are merged.
- The report is silent on things that look correct — silence means approval.
- P3 findings are suppressed from detail view when total findings exceed 30,
  keeping the report focused on what matters.

---

## Installation

Drop the `pr-review/` folder into your Claude skills directory or register it via the
Claude Code plugin system:

```bash
# Claude Code
/plugin install ./skills/pr-review
```

Or upload `SKILL.md` directly via the Claude.ai skills interface.

---

## License

Apache 2.0 — see [LICENSE](../../LICENSE).
