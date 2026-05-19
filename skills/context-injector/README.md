# Context Injector

Ends the "let me explain again" tax.

---

## The Problem

Every Claude session starts from zero. The user spends 5–10 minutes re-explaining
their role, project, stack, audience, and constraints. Claude acknowledges the context
and then gives an answer that ignores half of it.

The root cause isn't memory — it's that the context never gets structured enough to be
reliably applied. This skill builds a structured context template, scores it for quality,
and injects the right subset of context per question type.

---

## Workflow

```text
User sets up context (once, not every session)
    │
    ├─ /build — 5-question guided conversation across 5 dimensions:
    │           role, project, stack, audience, constraints
    │
    ├─ /score — identify what's missing and how much it changes answers
    │
    ├─ Context is injected silently per question type
    │   (technical → stack + hard limits; strategic → stage + goal; writing → audience)
    │
    └─ /drift — audit recent responses for signs context was forgotten
```

---

## Slash Commands

| Command | Action |
| --- | --- |
| `/build` | Interactive context template builder |
| `/score` | Score the template — what's missing that would change answers most |
| `/inject <question>` | Show what context would be applied to a given question |
| `/drift` | Audit last N responses for context drift |
| `/reset` | Clear the stored context |
| `/review` | Display the current context template |
| `/update <field> <value>` | Update a specific field |
| `/assume` | Show what Claude is currently assuming without stored context |

---

## The 5 Context Dimensions

| Dimension | What it captures |
| --- | --- |
| **Role** | Title, org type, technical level, decision-making style |
| **Project** | Name, type, stage, near-term goal, biggest constraint |
| **Stack** | Languages, frameworks, hosting, hard limits, preferred tools |
| **Audience** | User type, sophistication, core pain, churn signal |
| **Constraints** | Hard nos, already tried, budget/time, quality bar |

---

## Installation

Add to your `.claude/skills/` directory or reference in your `CLAUDE.md`:

```yaml
skills:
  - name: context-injector
    path: skills/context-injector/SKILL.md
    use-when: >
      User wants persistent context across questions. Activate on: "let me explain
      my situation", "as I mentioned", "you keep forgetting", "your answer is too
      generic", "I have to re-explain", "save my context", "remember this",
      "give me a personalised answer", "set up context".
```

---

## Example: /build flow

```text
User: /build

Context Injector: 5 questions. Start simple.

  Q1: What's your role? Give me the LinkedIn headline version — but also
      tell me what that means day-to-day.

User: Solo founder. Building B2B SaaS. I write most of the code but
      I'm more product than technical.

Context Injector: Technical enough to build, product-first in thinking.

  Q2: What are you building and where is it right now?

User: Contract management for freelancers. Live, 3 beta users, no paying customers.

  [continues through stack, audience, constraints...]

Template built. Score: 9/10.
One gap: what have you already tried for getting paying customers?
Add that and I won't recommend approaches you've ruled out.
```

## Example: /drift audit

```text
User: /drift

DRIFT AUDIT (last 4 responses)
  Response 1 : ⚠ DRIFT — recommended building a referral system.
               Your stored constraint: "no new features until first paying customer."
  Response 2 : ✓ Context applied
  Response 3 : ✓ Context applied
  Response 4 : ⚠ DRIFT — wrote for a technical audience.
               Your stored audience: non-technical freelancers.

  Drift: 2/4

  Correcting Response 1: [advice without the drifted recommendation]
```
