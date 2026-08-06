# Skills

**A curated collection of Claude skills.** Load once, ship better output, save tokens.

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Discord](https://img.shields.io/badge/Discord-Join%20Community-5865F2?style=flat&logo=discord&logoColor=white)](https://discord.gg/MmsTNm8WF6)

---

## What is this?

This repo is a library of `SKILL.md` files that upgrade how Claude (and any Claude-powered tool) behaves — from decisions and writing voice to code review and skill management. Each skill auto-activates on matching triggers and ships with a documented first-response contract.

---

## The Skills

### Design
| Skill | What it does |
|---|---|
| [**claude-design**](skills/claude-design/) | 7 design systems merged into one: animation physics, typography, OKLCH color, token architecture, Framer Motion, Bento 2.0, anti-slop rules + 20+ slash commands. |

### Decision Making & Thinking
| Skill | What it does |
|---|---|
| [**anti-hedge**](skills/anti-hedge/) | Forced single-answer decisions. Bans "it depends", commits to one recommendation, explains what was ruled out and what would flip it. |
| [**real-problem**](skills/real-problem/) | Root-cause question reframing. Catches XY problems and answers the question behind the question. |
| [**pre-mortem**](skills/pre-mortem/) | Adversarial failure hunting. Stress-tests plans by building the strongest case against them before you commit. |
| [**constraint-solver**](skills/constraint-solver/) | Treats constraints as design material. Finds elegant solutions when budget, time, tech, or regulation tightly restrict the space. |
| [**cross-domain-synthesis**](skills/cross-domain-synthesis/) | Structural borrowing between fields — finds the mechanism in domain A that directly solves the problem in domain B. |
| [**iterative-refinement**](skills/iterative-refinement/) | 4–6 structured improvement cycles on any output without drifting from intent or regressing earlier gains. |
| [**gap-closer**](skills/gap-closer/) | Finishes 80%-done projects. Audits what exists vs. intended and outputs exactly ONE next action to unblock shipping. |

### Writing & Voice
| Skill | What it does |
|---|---|
| [**voice-auth**](skills/voice-auth/) | Fingerprints your writing (sentence length, vocab, punctuation, filler) and rewrites output to sound like you — kills the AI smell. |
| [**tone-voice-mimicry**](skills/tone-voice-mimicry/) | Copies any external voice — a brand, publication, or author — from sample text, stylistically indistinguishable. |
| [**structured-output**](skills/structured-output/) | Enforces exact machine-readable output (JSON, YAML, CSV, markdown) with zero deviation, prose, or invented fields. |

### Code & Quality
| Skill | What it does |
|---|---|
| [**pr-review**](skills/pr-review/) | Fans out 8 parallel review lenses (security, architecture, perf, dependencies, logging, testing, docs, a11y) into a severity-graded report. |
| [**red-teaming**](skills/red-teaming/) | Plays the attacker / adversarial user / skeptical engineer on code, systems, and plans. Surfaces the most dangerous findings first. |
| [**silent-failure-detection**](skills/silent-failure-detection/) | Catches confident wrongness — surfaces hidden assumptions and probes calibration before hallucination causes damage. |
| [**constitutional-reasoning**](skills/constitutional-reasoning/) | Self-critique loop: evaluates its own output against principles, argues against itself, and revises until all principles hold. |

### Documents & Large Context
| Skill | What it does |
|---|---|
| [**large-doc-mastery**](skills/large-doc-mastery/) | Precision synthesis of 50k–200k+ token contexts — codebases, PDFs, legal docs, transcripts — with perfect recall and cross-referencing. |
| [**claude-md-generator**](skills/claude-md-generator/) | Self-healing CLAUDE.md generator. Scans the codebase, writes a ≤50-line root CLAUDE.md, and patches gaps after every mistake. |

### Agent, Skill & Tool Management
| Skill | What it does |
|---|---|
| [**context-router**](skills/context-router/) | Meta-skill that routes each query to only the relevant skills/MCP tools, then unloads heavy skills to keep context clean. |
| [**mcp-deduplicator**](skills/mcp-deduplicator/) | Detects overlapping MCP tools across servers, generates routing rules, and kills the "too many tools" tax. |
| [**trigger-auditor**](skills/trigger-auditor/) | Debugs skill triggers — fixes skills that won't fire, fire too eagerly, or conflict with each other. |
| [**context-injector**](skills/context-injector/) | Ends the "let me explain again" tax — builds a structured context template and re-injects it per session. |
| [**precision-prompt-chaining**](skills/precision-prompt-chaining/) | Chains dependent prompts (research → outline → draft → edit) with named variables, validation gates, and failure recovery. |

---

## Quick Start

### Option 1 – Claude.ai Projects (persistent)

1. Go to [Claude.ai](https://claude.ai) → **Projects** → **Create new project**
2. In the project, go to **Knowledge** → **Upload file**
3. Upload the `SKILL.md` for the skill you want (e.g. `skills/anti-hedge/SKILL.md`)
4. Start a chat — the skill auto-activates on its triggers

### Option 2 – One-time upload

Attach the `SKILL.md` to your first message in any Claude chat.

### Option 3 – Any Claude-powered tool (Cursor, Windsurf, VS Code + Continue, API)

The `SKILL.md` files are fully portable:

- **Cursor**: paste the skill into `.cursor/rules` or Settings → Rules
- **Continue.dev**: add under `customRules` in `~/.continue/config.json`
- **Claude API / LangChain / LlamaIndex**: include the full text in your system prompt or as a retrieved document, prefixed with:

```text
You are now using the <skill-name> skill. Follow every rule in the loaded skill file exactly.
```

### Option 4 – Skill loader (agentic setups)

```yaml
skills:
- name: anti-hedge
  path: skills/anti-hedge/SKILL.md
```

---

## Recommended Stack

Many of these skills compose well together. A common setup:

- **context-injector** → persistent project context
- **context-router** → routes to the right skill per query
- **claude-design** → ships interfaces without slop
- **pr-review** + **red-teaming** → ships code safely
- **voice-auth** → output that sounds like you

---

## License

[Apache 2.0](LICENSE)

---

## Community

Join the Discord to share skills, get help, and follow updates: **[discord.gg/MmsTNm8WF6](https://discord.gg/MmsTNm8WF6)**

Star this repo if it saves you thousands of tokens and hours of prompt engineering.
