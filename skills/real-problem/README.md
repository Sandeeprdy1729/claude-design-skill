# Real Problem Detector

Answers the question behind the question.

---

## The Problem

Users ask questions based on their current mental model. Mental models are often wrong
— which means the question is often wrong. Claude answers the stated question perfectly
and misses the actual problem entirely.

"How do I get more followers?" might be the wrong question if the goal is customers.
"How do I stop procrastinating?" might be the wrong frame if the work is genuinely wrong
for the person. "How do I optimise my queries?" is wrong if there are no slow queries yet.

This skill runs a root-cause check before answering anything.

---

## Workflow

```text
User asks a question
    │
    ├─ Parse: stated question, embedded method assumption, underlying goal
    │
    ├─ Root cause check (3 tests):
    │   1. Goal alignment — does answering this achieve the underlying goal?
    │   2. Question validity — XY problem? Premature? False dilemma? Wrong layer?
    │   3. Faster path — is there a simpler path to the goal?
    │
    ├─ If a reframe is found: surface it explicitly
    │
    ├─ Consent gate: answer this question, or the real one?
    │
    └─ Answer the chosen question fully
```

---

## Slash Commands

| Command | Action |
| --- | --- |
| `/detect <question>` | Run the root-cause check |
| `/reframe` | Surface the reframed version of the last question |
| `/answer-this` | Answer the stated question (skip the reframe) |
| `/answer-real` | Answer the reframed question |
| `/xy` | Diagnose if this is an XY problem |
| `/goal` | Ask: what are you ultimately trying to achieve? |
| `/why` | Run a 5-why root cause chain |

---

## Reframe Taxonomy

| Type | Example |
| --- | --- |
| Wrong metric | "More followers" when the goal is customers |
| Wrong layer | Fixing a CSS bug when the layout model is wrong |
| Wrong problem | "I need to be more productive" when the work is the wrong work |
| Unnecessary problem | Building auth when an auth service exists |
| Premature problem | Optimising performance before having users |
| XY problem | Asking about the attempted solution instead of the goal |
| False dilemma | "React or Vue" when a CMS would work |

---

## Installation

Add to your `.claude/skills/` directory or reference in your `CLAUDE.md`:

```yaml
skills:
  - name: real-problem
    path: skills/real-problem/SKILL.md
    use-when: >
      User may be asking the wrong question. Activate on: "how do I",
      "why won't this work", "what's the best way to", "I'm trying to",
      "I keep failing at", "what am I doing wrong", "help me figure out".
      Skip for: exact factual lookups, specific debugging, explicit "just answer this".
```

---

## Example: XY Problem

**User:** "How do I remove the .html extension from URLs in my Node.js app?"

```text
XY PROBLEM DETECTED
  You're asking about  : URL rewriting to hide .html extensions
  I think the goal is  : Clean, user-friendly URLs
  Why X may be wrong   : Rewriting is a workaround. The root cause is
                         that the files have extensions at all.
  Faster path          : If using Vercel/Netlify, enable "clean URLs"
                         in the config — no code needed.
────────────────────────────────────────────────────────────────
If you specifically need URL rewriting (migrating an existing site),
I'll answer that directly — just confirm.
```

## Example: Premature Problem

**User:** "How do I optimise the database queries for my SaaS app?"

```text
REAL PROBLEM DETECTED
  You asked     : Database query optimisation
  I think you want : Fast application performance
  The gap       : Optimisation is premature before you have slow queries
                  to optimise. Adding complexity now costs time without
                  measurable benefit.
  Better question: "How do I instrument my queries so I know when
                    optimisation becomes necessary?"
────────────────────────────────────────────────────────────────
[A] How to optimise queries in general
[B] How to instrument and identify slow queries first
[C] Both
```
