# Project Gap Closer

Turns half-built things into done things.

---

## The Problem

The world is full of 80%-done projects. The user knows what it should be, can't
clearly see the gap, and is paralysed. Claude given the project either describes
what already exists or suggests improvements — neither is useful.

What's needed: find the specific gap that is blocking shipping, and output exactly
one next action.

---

## Workflow

```text
User describes their half-built project
    │
    ├─ Existence audit: map what's actually there vs what was intended
    │
    ├─ Gap classification:
    │   BROKEN (exists, doesn't work, blocks shipping)
    │   MISSING (intended, doesn't exist, blocks shipping)
    │   OPTIONAL (nice-to-have, defer to v1.1)
    │   WRONG-DIRECTION (exists, leading away from the goal — remove)
    │
    ├─ Identify the blocker: the one gap that blocks everything else
    │
    ├─ Single next action: completable in one session, clear done signal
    │
    └─ Done criteria: what 'done enough to ship' looks like
```

---

## Slash Commands

| Command | Action |
| --- | --- |
| `/audit <project>` | Map what exists vs what was intended |
| `/gaps` | Classify all identified gaps |
| `/next` | Output the single next action |
| `/done` | Define done-enough-to-ship criteria |
| `/scope` | Separate in-scope from deferrable |
| `/blocker` | Identify the one gap blocking all other progress |
| `/sequence` | Order gaps by dependency if multiple are required |

---

## Gap Classification

| Class | Meaning | Action |
| --- | --- | --- |
| **BROKEN** | Exists, doesn't work | Fix before shipping |
| **MISSING** | Was intended, doesn't exist | Build before shipping |
| **OPTIONAL** | Would be nice, not blocking | Defer to v1.1 |
| **WRONG-DIRECTION** | Exists, leading away from goal | Remove or descope |

The most valuable classification is WRONG-DIRECTION — things that look like progress
but are actually debt. Complex admin panels before any users. Features for imaginary
power users. Abstractions before the concrete use case is stable.

---

## Next Action Rules

The single next action must be:

- Completable in one work session (≤4 hours)
- Specific: not "fix auth" but "make the login endpoint return a session token and redirect to /dashboard"
- The one action that unblocks the most other gaps

It is never:

- A roadmap
- A list of options
- An improvement suggestion
- A nice-to-have masquerading as blocking

---

## Installation

Add to your `.claude/skills/` directory or reference in your `CLAUDE.md`:

```yaml
skills:
  - name: gap-closer
    path: skills/gap-closer/SKILL.md
    use-when: >
      User has a partially-built project and needs to identify what's blocking
      shipping. Activate on: "help me finish", "what's left", "I can't ship",
      "I'm stuck", "almost done but", "half-built", "80% done", "never shipped",
      "what's blocking me", "what do I do next".
```

---

## Example: half-built SaaS

**User:** "I have a contract management SaaS — backend done, frontend has the list
and create form. Never shipped it."

```text
EXISTENCE AUDIT
  Exists : Auth, CRUD endpoints, PDF generation, contract list, create form
  Missing: Landing page (not confirmed), edit/delete, payments, error states

GAP CLASSIFICATION
  BROKEN    : (none identified)
  MISSING   : ✗ Landing page — no entry point for new users
  OPTIONAL  : ○ Edit contract  ○ Delete contract  ○ Billing  ○ Error states
  WRONG-DIR : (none identified)

NEXT ACTION
  Write a single-page landing page with: (1) a headline explaining who
  this is for and what it does, (2) a signup link to your existing auth,
  (3) your email for questions. Nothing else.

  Done when : A stranger reads it and knows what to do next.
  Unblocks  : Sending the link to your first potential user.
  Time      : 2–3 hours.

DONE ENOUGH TO SHIP
  □ Landing page is live and signup works
  □ A new user can create a contract and download the PDF
  □ You can be notified when someone signs up
```
