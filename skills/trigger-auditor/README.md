# Skill Trigger Auditor

**Stop guessing why your skill won't load. Diagnose it.**

The #1 fix for a skill that doesn't trigger is rewriting the `description` field — but
most users iterate blindly. The Trigger Auditor applies a structured six-test diagnosis
and produces a corrected description with a reproducible test suite.

---

## The problem

You built a skill. You enabled it. You sent the perfect query. The skill didn't load —
or the wrong skill loaded instead. Now what?

The `description` field is the only signal Claude uses to decide whether to activate a
skill. If the vocabulary doesn't match your query, the field is too generic, or two
skills overlap, the skill will fail or misfire — every time.

---

## Failure modes

| Code | Name | What you see |
| --- | --- | --- |
| `F1` | **Miss** | Skill never loads, Claude responds generically |
| `F2` | **False positive** | Skill loads on unrelated queries |
| `F3` | **Collision** | Two skills load simultaneously and contradict each other |
| `F4` | **Hijack** | Wrong skill loads instead of the intended one |
| `F5` | **Dropout** | Skill loads on turn 1, stops applying by turn 3 |
| `F6` | **Delayed load** | Skill only triggers after you rephrase the query |

---

## Six diagnostic tests

Every audit runs all six tests and reports `PASS`, `WARN`, or `FAIL`:

| Test | What it checks |
| --- | --- |
| **1. Keyword Coverage** | Does the description contain the words you actually used? |
| **2. Specificity Score** | Is the description specific enough to beat a generic Claude response? |
| **3. Negative Trigger Gaps** | Are there adjacent queries that would accidentally trigger this skill? |
| **4. First Response Completeness** | Does the description include a `First response:` confirmation line? |
| **5. Use-when Coverage** | Does the `Use when user says:` list match your query's intent? |
| **6. Description Length** | Is the description in the 100–400 word sweet spot? |

---

## Workflow

```text
Failing conversation + skill description(s)
    │
    ├─ Classify failure mode (F1–F6)
    ├─ Run 6 diagnostic tests
    ├─ Cross-skill conflict check (if ≥2 skills provided)
    ├─ Produce rewritten description with per-change rationale
    └─ Generate 10-query test suite (5 should-trigger, 5 should-not-trigger)
```

---

## Sample audit report

```text
╔══════════════════════════════════════════════════════════════
  TRIGGER AUDIT REPORT
  Skill       : pr-review
  Failure mode: F1 — Miss
  Tests run   : 6 / 6
  Result      : 2 PASS · 2 WARN · 2 FAIL
╠══════════════════════════════════════════════════════════════
  TEST 1  Keyword Coverage       FAIL   18% — "scan", "diff" absent from description
  TEST 2  Specificity Score      WARN   4/8 — borderline
  TEST 3  Negative Trigger Gaps  PASS
  TEST 4  First Response         FAIL   line absent
  TEST 5  Use-when Coverage      WARN   closest phrase: 0.51 similarity
  TEST 6  Description Length     WARN   89 words — too brief
╠══════════════════════════════════════════════════════════════
  ROOT CAUSE
  User typed "scan my diff for security holes". None of those words appear
  in the description. Claude could not match the query to this skill.
╠══════════════════════════════════════════════════════════════
  FIX
  Add to Use when user says: "scan, scan for security, security holes,
  check diff, look at my diff, diff analysis"
  Add First response: "PR Review skill active. Paste the diff or PR URL."
╚══════════════════════════════════════════════════════════════
```

---

## Slash commands

| Command | What it does |
| --- | --- |
| `/audit <skill>` | Full 6-test audit of a single skill |
| `/compare <skill-a> <skill-b>` | Overlap score and routing ambiguity report |
| `/rewrite <skill>` | Corrected description with per-change rationale |
| `/test <skill>` | 10-query test suite (5 should-trigger, 5 should-not) |
| `/coverage <skill>` | Keyword coverage map |
| `/negative <skill>` | Adjacent queries the skill should reject but currently might not |
| `/rank <query>` | Score all registered skills against a query |
| `/batch` | Audit all skills at once; surface top conflicts |

---

## Cross-skill conflict detection

When you provide two skill descriptions, the auditor computes an overlap score and
recommends the right resolution strategy:

| Overlap | Result | Action |
| --- | --- | --- |
| 0–20 | Clean separation | No action needed |
| 21–50 | Partial overlap | Add routing rules |
| 51–80 | Significant ambiguity | One skill will shadow the other unpredictably |
| 81–100 | Near-identical | Merge or differentiate urgently |

Resolution strategies: input-type routing, action-verb routing, scope routing,
explicit exclusion, or merge.

---

## Rewrite format

Every rewrite follows the canonical structure and explains every change:

```text
REWRITE: pr-review
  Original: 89 words  [WARN]  →  Rewritten: 187 words  [PASS]

  ADDED
    + "scan, diff analysis" → Test 1 keyword miss
    + "Do NOT activate for: creating PRs" → Test 3 gap
    + First response line → Test 4 FAIL
    + 6 new Use-when phrases → Test 5 FAIL

  REMOVED
    - "helps with code" → too generic (Test 2 score −2)
```

---

## Installation

1. Add `skills/trigger-auditor/SKILL.md` to your Claude project.
2. Enable it alongside the skills you want to audit.
3. Paste a failing conversation + the relevant `description` field(s) to start.
