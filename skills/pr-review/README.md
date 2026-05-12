# pr-review

> Automated code review through eight parallel lenses. Every finding is severity-graded,
> file-precise, confidence-scored, and comes with a ready-to-apply fix.

---

## What it does

Manual code review is slow, inconsistent, and misses things. `pr-review` replaces the
back-and-forth by fanning out eight specialised sub-agents simultaneously — each focused
on a single quality dimension — then synthesising a single structured report with
confidence scores, effort estimates, and actionable suggestions.

Paste a diff, a list of file paths, or a description of what changed. Get back a report
in seconds that a senior engineer would be proud to sign off on.

---

## The eight lenses

Each lens runs in parallel. Coverage is deep because every agent has a single job.

| Lens | What it catches |
| --- | --- |
| **Security** | OWASP Top 10, injection, hardcoded secrets, weak crypto, CORS, XXE, deserialization |
| **Architecture** | Coupling, complexity, transaction boundaries, circuit breakers, idempotency, DRY |
| **Performance** | N+1 queries, memory leaks, unbounded growth, missing pagination, async blocking |
| **Dependencies** | Outdated packages, CVEs, license conflicts, abandoned packages, native binding risk |
| **Logging** | Missing logs, PII leakage, wrong log levels, missing trace IDs, silent failures |
| **Testing** | Coverage gaps, flaky tests, brittle selectors, property-based gaps, contract tests |
| **Documentation** | Missing docstrings, stale comments, README gaps, type annotations, TODO hygiene |
| **Accessibility** | WCAG 2.1 AA violations, ARIA, keyboard traps, focus management, color contrast |

---

## Output format

Every finding is structured, precise, and actionable — never a vague observation.

```text
═══════════════════════════════════════════════════════════
  PR REVIEW REPORT
  Scope   : 4 files changed, +312 −47 lines
  Mode    : full
  Lenses  : Security · Architecture · Performance · Deps · Logging · Testing · Docs · a11y
  Summary : 1 critical  3 high  5 medium  8 low
═══════════════════════════════════════════════════════════

┌─ [SEC-001] 🔴 P0 Critical · Security
│  File       : src/api/users.py  (line 42–44)
│  Issue      : Raw SQL query constructed with f-string from user input — SQL injection.
│  Impact     : Attacker can dump or delete the entire users table.
│  Confidence : high
│  Effort     : trivial
│  Fix        :
│               # Before
│               db.execute(f"SELECT * FROM users WHERE id = {user_id}")
│               # After
│               db.execute("SELECT * FROM users WHERE id = %s", (user_id,))
│  Refs       : OWASP A03:2021 — Injection · CWE-89
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
/review              Full review across all eight lenses
/security            Security lens only
/arch                Architecture lens only
/perf                Performance lens only
/a11y                Accessibility lens only
/deps                Dependency lens only
/logging             Logging & observability lens only
/testing             Testing lens only
/docs                Documentation lens only
/fix <issue-id>      Generate a ready-to-apply patch for a specific finding
/explain <issue-id>  Educational deep-dive: why this is dangerous, not just how to fix
/issues              Open GitHub issues for every P0 and P1 finding
/pr-comment          Post findings as inline PR review comments on specific lines
/summary             One-paragraph executive summary of the last report
/tldr                One-sentence summary for Slack/Discord notifications
/diff                New/resolved findings vs the previous review + main branch baseline
```

---

## GitHub integration

Run `/issues` after a report to automatically open a labelled GitHub issue for every
P0 and P1 finding. Each issue includes the problem description, impact, and a
before/after code snippet. Issues are labelled `bug` (P0), `security` (P1 Security),
`enhancement` (P1 other), or `tech-debt` (P2/P3).

Run `/pr-comment` to post findings as inline review comments on specific diff lines
rather than a monolithic report — much more actionable for the PR author.

---

## How the workflow runs

```text
Input (diff / file paths / plan / repo)
    │
    ├─ Phase 0: Guardrails
    │     Size check (>100 files → refuse; >20 files → high-signal mode).
    │     Privacy warning. Pre-flight secret redaction.
    │     Load .pr-review.yml config.
    │
    ├─ Phase 1: Reconnaissance
    │     Read files, detect language + framework, map entry points,
    │     read PR description, commit messages, CODEOWNERS.
    │
    ├─ Phase 2: Fan-out  (parallel)
    │     ├─ Lens A: Security
    │     ├─ Lens B: Architecture
    │     ├─ Lens C: Performance & Scalability
    │     ├─ Lens D: Dependencies
    │     ├─ Lens E: Logging & Observability
    │     ├─ Lens F: Testing
    │     ├─ Lens G: Documentation
    │     └─ Lens H: Accessibility (a11y)
    │
    ├─ Phase 3: Synthesis
    │     Merge, deduplicate, confidence score, apply config overrides.
    │
    ├─ Phase 4: Prioritization
    │     Effort estimates, CODEOWNERS attribution, team baseline trend.
    │
    └─ Phase 5: Output
          Structured report + optional CI exit codes, SARIF, or GitHub actions.
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
> Claude will confirm: *"PR Review skill active. Running all eight lenses…"*

**Targeted scan:**

> `/security src/payments/` — runs only the Security lens on the payments directory.

**Performance scan:**

> `/perf src/api/` — runs only the Performance lens; useful after adding new endpoints.

**Accessibility scan:**

> `/a11y src/components/` — checks all UI components for WCAG 2.1 AA compliance.

**Apply a fix:**

> `/fix SEC-001` — Claude outputs a unified diff patch ready to apply.

**Understand a finding:**

> `/explain SEC-001` — Claude explains the issue from first principles with real-world examples.

**Open GitHub issues:**

> `/issues` — opens one labelled issue per P0/P1 finding with full context.

**Post inline review comments:**

> `/pr-comment` — posts findings as inline comments on the PR diff.

---

## Guarantees

- Every finding names the exact file and line range — no vague feedback.
- Every finding includes a confidence score and effort estimate.
- Every finding includes a before/after code snippet when a fix is suggested.
- Findings are never duplicated across lenses — cross-lens matches are merged and promoted.
- The report is silent on things that look correct — silence means approval.
- P3 findings are suppressed from detail view when total findings exceed 30.
- Low-confidence findings are grouped separately — never mixed with high-confidence results.
- Large PRs (>100 files) are refused with a helpful scoping prompt rather than silently failing.
- Suppress any finding with `# pr-review-ignore: REASON` — suppressions are logged in the footer.

---

## CI/CD integration

Exit codes: `P0/P1 findings → exit 1` (blocks CI), `P2/P3 only → exit 0`. Configurable
via `fail_on: p0_only` in `.pr-review.yml`.

SARIF output for GitHub Advanced Security:

```yaml
# .github/workflows/pr-review.yml
- run: claude --skill pr-review --review --sarif > results.sarif
- uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: results.sarif
```

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
