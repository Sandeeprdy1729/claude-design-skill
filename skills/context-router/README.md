# context-router

> One skill to rule them all. Reads your query, loads only what's needed,
> keeps everything else out of the way.

---

## The problem it solves

Claude's reasoning quality degrades when too many skills and MCP tools are loaded
at once. The official documentation warns about this explicitly: give the model only
what it needs for the task at hand.

In practice, most setups do the opposite — 10–20 skills enabled globally, 40+ MCP
tool definitions always in context. The result is slower responses, wrong tool
selection, and instructions that contradict each other.

`context-router` fixes this by acting as a gating layer. You enable **one skill**.
It reads every query, loads the minimal set of skills and tools for that task,
delegates to them, then unloads them when done.

---

## How it works

```text
Incoming query
    │
    ├─ Phase 1: Intent Classification
    │     Extract domain signals, action verbs, technology mentions
    │
    ├─ Phase 2: Skill Selection
    │     Match signals against registry · resolve conflicts · check budget
    │
    ├─ Phase 3: Load
    │     Inject selected SKILL.md files · expose only required MCP tools
    │
    ├─ Phase 4: Delegate
    │     Transparent pass-through to the loaded skill
    │
    └─ Phase 5: Cleanup
          Unload single-use skills · log freed tokens
```

Every routing decision is logged in a compact footer block so you always know what
was loaded, what was skipped, and why.

---

## Routing log

Every response includes a router log:

```text
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  CONTEXT ROUTER LOG
  Query intent  : code-quality  (confidence: 94%)
  ─────────────────────────────────────────────────────
  Loaded        : pr-review          weight: heavy  (+8 200 tokens)
  MCP tools     : github-pr-read  (1 / 47 total tools exposed)
  Skipped       : design-system      reason: no UI signals
  ─────────────────────────────────────────────────────
  Budget used   : 29 400 / 169 000 tokens  (17%)
  Unloaded      : pr-review          freed: ~8 200 tokens
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Slash commands

```text
/route <query>        Analyse query, load optimal skills + tools, execute
/skills               List all registered skills and their current status
/tools                List all MCP tools and which skill they're assigned to
/load <skill>         Force-load a specific skill
/unload <skill>       Unload a skill and free its context budget
/conflicts            Show the conflict map
/registry             Print the full skill registry
/budget               Show current context token usage by skill
/reset                Unload all skills, return to router-only state
/audit                Explain why each loaded skill was selected
```

---

## MCP tool gating

This is the most impactful feature. Without the router, every MCP tool definition
lives in context on every turn — the model sees 47 tools and frequently picks the
wrong one.

With the router, a skill declares which tools it needs:

```yaml
mcp_tools:
  pr-review:
    - mcp_io_github_git_pull_request_read
    - mcp_io_github_git_create_pull_request_review

  design-system:
    - open_browser_page
    - screenshot_page
```

When `pr-review` is loaded, only its 2 tools are surfaced. The other 45 are invisible.
The model picks correctly every time.

---

## Skill registry

The registry maps trigger keywords and intent categories to skills. It lives in
`.context-router.yml` in your project root.

```yaml
skills:
  - name: pr-review
    triggers: [review, PR, pull request, diff, audit, code quality]
    intent_categories: [code-quality, security]
    context_weight: heavy
    unload_after: single-use
    conflicts: []
    skill_file: ./skills/pr-review/SKILL.md

  - name: design-system
    triggers: [design, UI, component, animation, CSS, typography, layout]
    intent_categories: [ui-design]
    context_weight: heavy
    unload_after: session
    conflicts: []
    skill_file: ./skills/claude-design/skill.md
```

---

## Confidence thresholds

| Confidence | Behaviour |
| --- | --- |
| ≥80% | Silent auto-load and execute |
| 50–79% | Load with a one-line note explaining the routing decision |
| <50% | Ask one clarifying question before loading anything |

---

## Conflict resolution

Skills that would contradict each other can't be co-loaded. The router picks the
higher-scoring (more specific) skill and logs the decision:

```text
[ROUTER] Conflict: pr-review vs code-review — loaded pr-review (score: 92 vs 61)
```

Built-in conflict rules:

- Two competing database skills (contradictory query syntax)
- Two competing style systems (contradictory class naming)
- A general skill and its specialised variant (specialised wins)

---

## Context budget

```text
Total context           : ~200 000 tokens
Reserved for output     :   10 000
Reserved for router     :    1 000
Reserved for conversation:  20 000
Available for skills    : ~169 000
```

If loading the selected set would exceed the budget, the router summarises the
lowest-weight skill and logs the compression. If still over budget, it unloads
single-use skills from prior turns.

---

## Installation

### 1. Drop the skill into your skills directory

```bash
/plugin install ./skills/context-router
```

### 2. Create `.context-router.yml` in your project root

```yaml
skills:
  - name: pr-review
    triggers: [review, PR, diff, audit]
    intent_categories: [code-quality, security]
    context_weight: heavy
    unload_after: single-use
    skill_file: ./skills/pr-review/SKILL.md
    mcp_tools:
      - mcp_io_github_git_pull_request_read

  - name: design-system
    triggers: [design, UI, component, animation]
    intent_categories: [ui-design]
    context_weight: heavy
    unload_after: session
    skill_file: ./skills/claude-design/skill.md
    mcp_tools:
      - open_browser_page
      - screenshot_page
```

### 3. Disable all other skills

In your Claude project settings, disable every skill except `context-router`.
The router will load the others on demand.

---

## Adding a new skill to the router

1. Create `./skills/<name>/SKILL.md`.
2. Add an entry to `.context-router.yml`.
3. Run `/registry` to confirm it registered.
4. Run `/route <test query>` to verify it triggers correctly.

**Standard intent categories:**

| Category | Covers |
| --- | --- |
| `code-quality` | Review, audit, refactor, testing |
| `ui-design` | Frontend, components, animation, styling |
| `data` | Database queries, schema, migrations |
| `infrastructure` | CI/CD, deployment, IaC, Docker |
| `documentation` | Writing, README, changelogs |
| `security` | Vulnerability scanning, threat modelling |
| `explanation` | Teaching, debugging, concept walkthroughs |
| `planning` | Architecture decisions, task breakdown |

---

## Why this beats manually managing skills

| Manual approach | With context-router |
| --- | --- |
| Enable 20 skills globally | Enable 1 skill |
| 47 MCP tools always in context | 1–3 tools per turn |
| Skills contradict each other | Conflict map enforces single authority |
| Stale skill context accumulates | Single-use skills unload automatically |
| No visibility into what's loaded | Router log on every response |
| Heavy skills waste tokens on simple queries | Loaded only when triggered |

---

## License

Apache 2.0 — see [LICENSE](../../LICENSE).
