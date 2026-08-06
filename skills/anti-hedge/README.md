# Anti-Hedge Decision Maker

Anti-Hedge is a decision-forcing skill that commits to one recommendation when a user asks for a choice.

It is designed for prompts like: should I use X or Y, which is better, pick one, just tell me, what would you do.

## What It Does

- Bans hedging language and non-answers.
- Asks at most 2 clarifying questions.
- Commits to exactly one recommendation.
- Explains what was ruled out and why.
- States the single condition that would flip the decision.

## When To Activate

Activate when the user is asking for a comparative judgment and wants a direct answer.

Typical triggers:

- which should I use
- should I pick X or Y
- what is better
- recommend one
- make a decision
- help me decide

Do not activate for:

- open-ended creative work
- requests that explicitly ask for multiple options
- research summaries where multiple answers are equally valid

## First Response Contract

On activation, the assistant should declare:

Anti-Hedge active. I will ask at most 2 questions, then commit to one answer. No it depends.

## Workflow

1. Constraint extraction
2. Option elimination
3. Single decision in sentence one
4. Elimination audit
5. Override path
6. Decision chain (next 2 downstream decisions)

## Output Format

1. Recommendation first sentence:
Use OPTION.

2. Brief rationale:
One primary reason plus practical implementation context.

3. Ruling-out section:
RULED OUT with one killer constraint per rejected option.

4. Assumptions section:
State assumptions and the user signals that caused them.

5. Flip condition:
WHAT WOULD FLIP THIS with exactly one new constraint that changes the result.

## Clarifying Question Rules

- Ask no more than 2 questions.
- Each question should be binary or no more than 3 options.
- Ask only if the answer can change the recommendation.
- If user says just decide, skip questions and proceed with explicit assumptions.

## Banned Phrases

The response must not include:

- it depends
- both have merits
- it really comes down to
- consider your specific use case
- ultimately it is up to you
- there is no right answer
- you could go either way
- I cannot tell you without knowing more

## Confidence Calibration

- Reversible choices: Use X.
- Medium-cost reversals: Use X, and here is what to watch for.
- Hard reversals: Use X, with one explicit caveat.
- Near-irreversible: X, and here is the single thing that would make me wrong.

## Slash Commands

| Command | Action |
| --- | --- |
| /decide [question] | Force an immediate decision with current context |
| /constraints | List constraints driving the decision |
| /override [new constraint] | Re-run recommendation with an added top-priority constraint |
| /audit | Show full elimination reasoning for the last decision |
| /flip | Show the strongest argument for the ruled-out option |
| /stakes | Assess reversibility and adjust confidence language |
| /chain | Surface the next 2 decisions this choice forces |
| /deeper | Stress-test the most fragile assumption |
| /iterate [new info] | Re-run and explain what changed |
| /session | Show decisions made in this session and dependencies |

## Installation

Place this folder under your skills directory and reference SKILL.md in your skill loader.

Example reference:

```yaml
skills:
- name: anti-hedge
  path: skills/anti-hedge/SKILL.md
```

## Quick Example

User:
Should I use Next.js or Remix for my new SaaS?

Anti-Hedge:
Use Next.js.

Reason:
You need fastest path-to-launch with broad ecosystem support and hiring leverage.

RULED OUT:
Remix is ruled out by operational complexity relative to your launch-stage constraints.

WHAT WOULD FLIP THIS:
If your product depends on streaming-heavy real-time UX from day one, choose Remix instead.
