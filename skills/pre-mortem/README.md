# Pre-Mortem Devil's Advocate

Adversarial planning mode that finds the one assumption you're most wrong about.

---

## The Problem

Claude validates plans. You share an idea, Claude says "great concept, here are some
things to consider." The things to consider are never the thing that kills the plan.

Real plans fail on the one assumption everyone treated as obvious. This skill refuses
to validate and instead attacks the plan — building the strongest case against it.

---

## Workflow

```text
User shares a plan
    │
    ├─ Parse: goal, method, key claims, implicit claims, timeline, resources
    │
    ├─ Extract assumptions by category (market / execution / technical / etc.)
    │   Score each: fragility × load-bearing weight → surface top 3
    │
    ├─ Attack: adversarial analysis for this plan type
    │   (product / career / investment / architecture / hiring)
    │
    ├─ Rank failure modes by probability × damage
    │
    └─ The Verdict: one sentence on the biggest threat
```

---

## Slash Commands

| Command | Action |
| --- | --- |
| `/premortem <plan>` | Run the full adversarial analysis |
| `/assumptions` | Surface and score assumptions |
| `/steelman` | Build the strongest case against the plan |
| `/rank` | Rank failure modes by probability × damage |
| `/probability <failure>` | Estimate likelihood of a specific failure mode |
| `/blind-spots` | Surface what proximity to the plan makes hard to see |
| `/fix` | Identify the one change that most reduces risk |

---

## Key Behaviours

- Never ends on uplift ("but with these fixes, it could work!")
- Holds critique under pushback — if you defend the plan, the critique doesn't soften
- Specificity is the whole game — "your market assumption is wrong because X data shows Y"
  is useful. "Competition could be a risk" is not.

---

## Installation

Add to your `.claude/skills/` directory or reference in your `CLAUDE.md`:

```yaml
skills:
  - name: pre-mortem
    path: skills/pre-mortem/SKILL.md
    use-when: >
      User shares a plan and wants adversarial analysis or failure-mode thinking.
      Activate on: "what could go wrong", "pressure test this", "devil's advocate",
      "pre-mortem", "what am I missing", "poke holes in this".
```

---

## Example

**User:** I'm launching a SaaS for freelance contract management. I have 3 beta users.
I'm planning to charge $29/month. Launch in 6 weeks.

**Pre-Mortem:**

**Biggest threat:** The willingness-to-pay assumption.

You have 3 beta users who are not paying. "Beta users don't pay" is a common
rationalisation, but it means you have zero evidence that freelancers will pay $29/month
for contract management. The historical data on this category is rough — most freelancers
track contracts in Notion or Google Docs and consider that "good enough."

_Evidence you'd need to feel confident:_ At least one beta user who said "I'd pay for
this right now" without prompting, or a pre-order from someone you've never met.

_Probability of failure on this specific assumption:_ High. Most freelancer SaaS
tools fail here.

_What would change this:_ Find one stranger on the internet with the problem, offer
them the product for $29/month, and have them pay. One payment from a stranger
is worth more than 100 enthusiastic beta conversations.
