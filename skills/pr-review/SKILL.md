---
name: pr-review
description: >
  Automated PR review and code quality analysis skill. Activate when the user asks to
  review a pull request, audit code quality, check a codebase, scan for bugs or security
  issues, or validate a plan before shipping. Fans out parallel sub-agents — one per
  review lens (security, architecture, dependencies, logging, testing, documentation) —
  then synthesises a structured severity-graded report with actionable suggestions.
  Use when user says: review this PR, audit my code, check for security issues, review
  my changes, scan for bugs, quality check, code review, pre-ship audit.
  First response: "PR Review skill active. Paste the diff, file paths, or describe what
  to review. I'll run every lens in parallel and give you a structured report."
license: Apache 2.0
---

# Automated PR Review & Code Quality

A structured, multi-lens code review system. Each lens runs independently so
coverage is deep, consistent, and fast. Output is always a severity-graded report
with concrete, actionable fixes — never vague observations.

---

## SLASH COMMANDS

| Command | Action |
|---|---|
| `/review` | Full review across all six lenses — security, architecture, deps, logging, testing, docs |
| `/security` | Security lens only — OWASP Top 10, secrets, injection, auth |
| `/arch` | Architecture lens only — coupling, cohesion, SOLID, complexity |
| `/deps` | Dependency lens only — outdated packages, license risk, supply chain |
| `/logging` | Logging & observability lens only — missing logs, PII leakage, trace IDs |
| `/testing` | Testing lens only — coverage gaps, brittle tests, missing edge cases |
| `/docs` | Documentation lens only — missing docstrings, stale comments, unclear APIs |
| `/fix <issue-id>` | Generate a ready-to-apply patch for a specific issue from the report |
| `/issues` | Open GitHub issues for every P0 and P1 finding in the last report |
| `/summary` | One-paragraph executive summary of the last report |
| `/diff` | Show only the delta between this review and the previous review on the same code |

---

## HIGH-LEVEL WORKFLOW

```
Input (diff / file paths / plan / repo)
    │
    ├─ Phase 1: Reconnaissance  ──  read files, understand intent, build context map
    │
    ├─ Phase 2: Fan-out          ──  launch all six lenses in parallel (sub-agents)
    │     ├─ Lens A: Security
    │     ├─ Lens B: Architecture
    │     ├─ Lens C: Dependencies
    │     ├─ Lens D: Logging & Observability
    │     ├─ Lens E: Testing
    │     └─ Lens F: Documentation
    │
    ├─ Phase 3: Synthesis        ──  merge findings, deduplicate, assign severities
    │
    └─ Phase 4: Report           ──  structured output + optional GitHub issue creation
```

---

## PHASE 1 — RECONNAISSANCE

Before any lens fires, build a context map:

1. **Identify scope** — diff only, changed files, or entire repo?
2. **Detect language(s)** — infer from file extensions; load language-specific rules.
3. **Detect framework(s)** — React, Django, Spring, etc.; apply framework-specific checks.
4. **Read entry points** — `main`, `index`, `app`, route registrations.
5. **Understand intent** — read PR description, commit messages, linked issue titles.
6. **Flag obvious blockers** — syntax errors, merge conflicts, broken imports — report
   these immediately before continuing.

---

## PHASE 2 — REVIEW LENSES

### LENS A — SECURITY

> Goal: find every exploitable vulnerability before it ships.

**Checklist:**

- [ ] **Injection** — SQL, command, LDAP, XPath, template injection. Verify all user
  input is parameterised or sanitised before use.
- [ ] **Authentication & authorisation** — broken auth flows, missing `@login_required`
  / auth middleware, insecure direct object references (IDOR), JWT algorithm confusion.
- [ ] **Secrets & credentials** — hardcoded API keys, passwords, tokens in source or
  config files. Check `.env` handling; verify secrets are never logged.
- [ ] **Cryptography** — weak algorithms (MD5, SHA-1, DES, ECB mode), insufficient key
  lengths, predictable IVs, insecure random number generation.
- [ ] **Data exposure** — PII in logs, stack traces returned to clients, verbose error
  messages, over-permissive CORS, missing HTTPS enforcement.
- [ ] **Input validation** — missing length/type/range checks at system boundaries,
  unvalidated redirects, path traversal vulnerabilities.
- [ ] **Dependency vulnerabilities** — known CVEs in direct and transitive dependencies
  (cross-reference with dependency lens).
- [ ] **Supply chain** — typosquatted packages, unpinned versions, absent lockfile
  integrity checks.
- [ ] **SSRF / open redirect** — any URL constructed from user-controlled input that
  makes server-side HTTP requests.
- [ ] **Rate limiting & DoS** — unbounded loops driven by user input, missing pagination
  limits, expensive regex on untrusted strings (ReDoS).

**Severity mapping for this lens:**
- `P0 (Critical)` — exploitable without authentication or with low-privilege access.
- `P1 (High)` — exploitable with authenticated access or under specific conditions.
- `P2 (Medium)` — defence-in-depth gap; not directly exploitable but raises risk.
- `P3 (Low)` — best-practice deviation with minimal direct risk.

---

### LENS B — ARCHITECTURE

> Goal: identify structural problems that compound over time.

**Checklist:**

- [ ] **Single Responsibility** — functions/classes doing more than one job. Flag any
  function >30 lines that mixes concerns (I/O + business logic, for example).
- [ ] **Coupling** — tight coupling between unrelated modules, circular imports,
  god objects/classes, feature envy.
- [ ] **Cohesion** — modules with unrelated responsibilities, poor layer separation
  (e.g., database queries inside UI components).
- [ ] **Complexity** — cyclomatic complexity >10 in a single function, deeply nested
  conditionals (>3 levels), long parameter lists (>4 params without a data class).
- [ ] **Naming** — misleading names, abbreviations, Hungarian notation, generic names
  (`data`, `info`, `temp`, `manager`, `utils` with >15 methods).
- [ ] **Error handling** — swallowed exceptions (`except: pass`), bare `catch (e) {}`,
  missing error propagation, inconsistent error shapes across an API.
- [ ] **Performance anti-patterns** — N+1 queries, synchronous I/O in hot paths,
  unbounded in-memory collections, missing indexes on queried fields.
- [ ] **DRY violations** — copy-pasted logic blocks >5 lines appearing more than once.
- [ ] **Dead code** — unreachable branches, unused exports, commented-out code blocks.
- [ ] **Abstraction level** — mixed abstraction levels inside a single function; leaky
  abstractions exposing implementation details to callers.

---

### LENS C — DEPENDENCIES

> Goal: keep the dependency tree lean, secure, and legally safe.

**Checklist:**

- [ ] **Outdated packages** — dependencies >1 major version behind their latest stable
  release; note breaking-change risk.
- [ ] **Unused dependencies** — packages imported in `package.json` / `requirements.txt`
  / `go.mod` but never `import`-ed in code.
- [ ] **Duplicate functionality** — two packages doing the same job (e.g., `axios` and
  `node-fetch` in the same project).
- [ ] **Bundle weight** — heavy packages pulled in for a single utility function
  (prefer tree-shakeable alternatives or inline the function).
- [ ] **License compliance** — GPL/AGPL in a proprietary project, license conflicts
  between dependencies.
- [ ] **Lockfile hygiene** — missing `package-lock.json` / `poetry.lock`, lockfile not
  committed, lockfile out of sync with manifest.
- [ ] **Pinning strategy** — overly loose (`*`, `>=1`) or overly strict (exact SHA for
  everything) pinning; recommend `~=` / `^` semantics where appropriate.
- [ ] **Peer-dependency mismatches** — declared peer deps not satisfied by installed
  versions.

---

### LENS D — LOGGING & OBSERVABILITY

> Goal: ensure the system is debuggable in production without leaking sensitive data.

**Checklist:**

- [ ] **Coverage** — every external call (HTTP, DB, queue, cache) has at least one log
  entry at the start and on error/success.
- [ ] **Log levels** — correct use of `DEBUG`, `INFO`, `WARN`, `ERROR`, `FATAL`.
  Avoid `INFO` for every message; avoid `DEBUG` left on in production code paths.
- [ ] **Structured logging** — logs are machine-parseable (JSON, key=value). No raw
  `print()` / `console.log()` in production paths.
- [ ] **PII leakage** — passwords, tokens, emails, PII must never appear in log output.
  Verify masking/redaction is applied before logging request/response bodies.
- [ ] **Trace & correlation IDs** — distributed transactions include a `trace_id` or
  `request_id` threaded through all log lines.
- [ ] **Error context** — exceptions are logged with full stack trace + request context,
  not just `"something went wrong"`.
- [ ] **Metrics & alerts** — critical paths (payments, auth, data mutations) emit
  a counter or histogram metric; alert thresholds are defined.
- [ ] **Silent failures** — code paths that catch exceptions and continue without any
  log entry.

---

### LENS E — TESTING

> Goal: find coverage gaps and brittle tests before they become production incidents.

**Checklist:**

- [ ] **Coverage gaps** — new code paths with no corresponding test; branches inside
  conditionals that are never exercised.
- [ ] **Happy-path only** — tests that only cover the success case; missing tests for
  error paths, boundary conditions, and empty/null inputs.
- [ ] **Edge cases** — off-by-one errors, empty collections, `null`/`undefined`,
  maximum-length inputs, concurrent access, timezone edge cases.
- [ ] **Test isolation** — tests sharing state via global variables, module-level side
  effects, or real external services (DB, HTTP) without mocking.
- [ ] **Brittle selectors** — UI tests using implementation-detail selectors (CSS class
  names, internal state) instead of accessible roles/labels.
- [ ] **Assertion quality** — `assert response` (truthy check) instead of
  `assert response.status_code == 200`; missing error message in assertion.
- [ ] **Test naming** — test names that don't describe the scenario
  (`test_1`, `testFoo`) making failures hard to diagnose.
- [ ] **Flaky tests** — tests with `sleep()` / fixed timeouts, order-dependent tests,
  tests seeded with wall-clock time.
- [ ] **Integration vs unit balance** — over-reliance on slow end-to-end tests for
  logic that could be unit-tested; or inverse: mocking so heavily that integration
  bugs are invisible.

---

### LENS F — DOCUMENTATION

> Goal: ensure the code is understandable by a new engineer without a walkthrough.

**Checklist:**

- [ ] **Public API docstrings** — every exported function, class, and method has a
  docstring explaining purpose, params, return value, and exceptions raised.
- [ ] **Inline comments** — complex business logic, non-obvious algorithms, and
  workarounds have a `# why` comment explaining intent, not just `# what`.
- [ ] **Stale comments** — comments that describe code that no longer exists or
  behaviour that has changed.
- [ ] **README coverage** — new environment variables, config options, CLI flags, or
  setup steps reflected in README / CONTRIBUTING docs.
- [ ] **Changelog / migration notes** — breaking changes in public APIs documented in
  CHANGELOG.md or a migration guide.
- [ ] **Type annotations** — typed languages: all public signatures have types; Python:
  all public functions have PEP 484 annotations.
- [ ] **Example usage** — complex utilities or SDK-style modules include at least one
  runnable example.
- [ ] **TODO hygiene** — `TODO`/`FIXME` comments have an owner and a linked issue; stale
  TODOs (>90 days old) are flagged.

---

## PHASE 3 — SYNTHESIS RULES

When merging findings from all lenses:

1. **Deduplicate** — if Security and Architecture both flag the same function, emit one
   finding with both lens tags.
2. **Promote severity** — a finding tagged by two or more lenses is promoted one
   severity level (e.g., P2 → P1).
3. **Group by file** — sort findings by file path, then by line number.
4. **Count totals** — report `P0 / P1 / P2 / P3` counts in the header.
5. **Trim noise** — suppress P3 findings if total finding count >30 (surface summary
   only); focus attention on actionable items.

---

## PHASE 4 — OUTPUT FORMAT

### Report Header

```
═══════════════════════════════════════════════════════════
  PR REVIEW REPORT
  Scope   : <files reviewed / diff size>
  Lenses  : Security · Architecture · Dependencies · Logging · Testing · Docs
  Summary : <P0_count> critical  <P1_count> high  <P2_count> medium  <P3_count> low
═══════════════════════════════════════════════════════════
```

### Finding Block (one per issue)

```
┌─ [<ID>] <SEVERITY> · <LENS>
│  File    : path/to/file.ext  (line <N>–<M>)
│  Issue   : <one-sentence description of the problem>
│  Impact  : <what goes wrong if this is not fixed>
│  Fix     : <concrete, actionable suggestion — include code snippet when helpful>
│  Refs    : <OWASP link / CWE / RFC / relevant doc — only when directly applicable>
└──────────────────────────────────────────────────────────
```

### Severity Legend

| Severity | Emoji | Meaning | Expected action |
|---|---|---|---|
| `P0 Critical` | 🔴 | Exploitable, data-loss, or outage risk | Block merge immediately |
| `P1 High` | 🟠 | Significant bug or security gap | Fix before merge |
| `P2 Medium` | 🟡 | Quality or maintainability debt | Fix in follow-up PR |
| `P3 Low` | 🔵 | Style, minor inconsistency, nice-to-have | Backlog or skip |

### Report Footer

```
───────────────────────────────────────────────────────────
  RECOMMENDED ACTIONS
  1. Address all P0s before merge.
  2. <specific top recommendation based on findings>
  3. <second specific recommendation>
  Run `/fix <ID>` for a ready-to-apply patch on any finding.
  Run `/issues` to open GitHub issues for P0 + P1 findings.
───────────────────────────────────────────────────────────
```

---

## GITHUB INTEGRATION

When the user runs `/issues` after a report:

1. **Authenticate** — use the available GitHub tool or confirm the user's `gh` CLI is
   authenticated (`gh auth status`).
2. **One issue per finding** (P0 and P1 only by default; ask before opening P2).
3. **Issue title format**: `[pr-review][<SEVERITY>] <one-sentence description>`
4. **Issue body template**:

```markdown
## Finding

**ID**: <report-ID>
**Severity**: <P0/P1>
**Lens**: <lens name>
**File**: `path/to/file.ext` (line N–M)

## Problem

<Issue description from report>

## Impact

<Impact description from report>

## Suggested Fix

<Fix description from report — include code snippet>

## References

<Refs from report>

---
*Opened automatically by the `pr-review` skill.*
```

5. **Label issues** with `bug` (P0), `security` (Security lens findings), or
   `tech-debt` (P2/P3).
6. **Report back** with a list of created issue URLs.

---

## LANGUAGE-SPECIFIC RULE EXTENSIONS

Load additional rule sets based on detected language:

| Language | Extra checks to apply |
|---|---|
| **Python** | Type annotations (PEP 484), `__all__` exports, mutable default args, `except Exception` breadth |
| **TypeScript / JS** | `any` overuse, missing `null` checks, `async` without `await`, prototype pollution |
| **Java / Kotlin** | Checked exceptions swallowed, thread-safety of shared state, `equals`/`hashCode` contract |
| **Go** | Error return ignored (`_`), goroutine leak, unbuffered channel in hot path |
| **SQL** | Implicit type coercions, missing `WHERE` on `UPDATE`/`DELETE`, non-SARGable predicates |
| **Terraform / IaC** | Public S3 buckets, wildcard IAM policies, unencrypted storage, missing state locking |

---

## BEHAVIOUR RULES

- **Never be vague.** Every finding must name the exact file, line range, and fix.
- **Show code.** When suggesting a fix, always include a before/after snippet.
- **No false praise.** Do not comment on things that are fine; silence means approval.
- **Stay in scope.** Only flag what is in the diff or the explicitly specified files
  unless a cross-file dependency is directly relevant to a finding.
- **Respect intent.** Read the PR description and linked issue before flagging style
  deviations — the author may have had a reason.
- **Be fast.** Use parallel sub-agents for the six lenses. Do not serialise lens
  execution unless there is a dependency.
- **One finding, one fix.** Do not bundle multiple problems into a single finding block.

---

## EXAMPLES

### Example: invoking a full review

> User: "Review this PR — here's the diff: [paste diff]"

Claude fans out six parallel lens sub-agents, then synthesises:

```
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
│  Fix     : Replace with parameterised query:
│            # Before
│            db.execute(f"SELECT * FROM users WHERE id = {user_id}")
│            # After
│            db.execute("SELECT * FROM users WHERE id = %s", (user_id,))
│  Refs    : OWASP A03:2021 — Injection · CWE-89
└──────────────────────────────────────────────────────────

...
```

### Example: targeted security scan only

> User: "/security src/payments/"

Claude runs only the Security lens against all files under `src/payments/`.

### Example: generating a fix

> User: "/fix SEC-001"

Claude produces a complete, ready-to-apply patch in unified diff format.
