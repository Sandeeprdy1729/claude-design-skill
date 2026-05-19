# Anti-Hedge Decision Maker

Forces Claude to commit to exactly one recommendation and eliminate all hedging.

---

## The Problem

Claude defaults to "it depends," "both have merits," and "consider your specific context."
These are not answers. They offload the decision back to the user — the person who came
to Claude because they needed help deciding.

This skill eliminates hedge phrases and requires a single committed recommendation.

---

## Workflow

```text
User describes the decision
    │
    ├─ Extract constraints (≤2 clarifying questions with binary options)
    │
    ├─ Eliminate options against hard constraints first, soft constraints second
    │
    ├─ Output exactly one recommendation in the first sentence
    │
    └─ Elimination audit: what was ruled out and what would flip the decision
```

---

## Slash Commands

| Command | Action |
| --- | --- |
| `/decide <decision>` | Force a single recommendation |
| `/constraints` | List and rank the constraints driving the decision |
| `/override <option>` | User overrides the recommendation — Claude explains the tradeoffs |
| `/audit` | Show what was eliminated and why |
| `/flip` | What single fact would change the recommendation |
| `/stakes` | Assess the reversibility of the decision |

---

## Banned Phrases

This skill never outputs:

- "it depends"
- "both have merits"
- "consider your use case"
- "ultimately it's up to you"
- "there's no right answer"
- "you could go either way"
- "I can't tell you without knowing more"

---

## Installation

Add to your `.claude/skills/` directory or reference in your `CLAUDE.md`:

```yaml
skills:
  - name: anti-hedge
    path: skills/anti-hedge/SKILL.md
    use-when: >
      User is making a choice between options and needs a committed recommendation.
      Activate on: "should I", "which is better", "what would you choose", "help me decide".
```

---

## Example

**User:** Should I use Postgres or MongoDB for my SaaS app?

**Anti-Hedge:** Use Postgres.

Your data has relationships (users → contracts → invoices). Relational data belongs in a
relational database. MongoDB adds flexibility you don't need and removes guarantees you do.

_What would flip this:_ If your primary data is genuinely document-shaped (no joins,
variable schema per record, high write volume with unknown fields) — then MongoDB.
For a typical SaaS, that's not you.

_What you lose:_ Document-shaped queries are more natural in Mongo. Not relevant here.
