# MCP Tool Deduplicator

**47 tools. 3 servers. One task. Which tool does Claude pick?**

The MCP ecosystem has 2000+ servers and users routinely connect 3–6 per project.
GitHub, GitLab, Jira, Linear, and Notion all expose tools for creating issues, posting
comments, and searching — but under different names and schemas. Without deduplication,
Claude burns tokens at every turn evaluating near-identical tools and sometimes picks
the wrong one.

This skill maps all tools across every connected server, detects semantic duplicates,
selects a canonical tool per operation, and generates routing rules that surface a
clean deduplicated toolset.

---

## The problem

With 4 servers connected (GitHub + GitLab + Jira + Linear) you can easily have:

- `create_issue`, `create_ticket`, `createIssue` — all meaning the same thing
- `list_pull_requests`, `list_merge_requests` — identical except for naming convention
- `search_code` appearing on both GitHub and GitLab with identical schemas

Claude sees all of them. Every turn, it has to evaluate whether to call `create_issue`
or `create_ticket` — wasting tokens and sometimes choosing inconsistently.

After deduplication, Claude sees one canonical tool per operation with routing rules
that silently switch to the right server based on context signals.

---

## Workflow

```text
Tool manifests from ≥2 MCP servers
    │
    ├─ Phase 1: Inventory — collect all tools, names, schemas
    ├─ Phase 2: Semantic Grouping — assign canonical operation categories
    ├─ Phase 3: Duplicate Detection — score similarity within each group
    ├─ Phase 4: Canonical Selection — pick the best tool per group
    ├─ Phase 5: Routing Layer — generate .mcp-dedup.yml config
    └─ Phase 6: Validation — verify coverage, no orphans, no circular routes
```

---

## Duplicate classification

| Score | Class | Action |
| --- | --- | --- |
| ≥90 | Exact duplicate | Keep one; suppress the other entirely |
| 70–89 | Functional equivalent | Keep one as canonical; other available via override |
| 50–69 | Near-duplicate | Keep both; add routing rules to disambiguate |
| <50 | Distinct | Keep both; no deduplication |

---

## Sample stats output

```text
MCP DEDUPLICATION SUMMARY
────────────────────────────────────────────────────────────
  Servers scanned        : 4  (github, gitlab, jira, linear)
  Total tools found      : 87
  Exact duplicates       : 14  (suppressed)
  Functional equivalents : 9   (demoted to overrides)
  Near-duplicates        : 6   (routing rules added)
  Unique tools           : 58  (always available)
  ────────────────────────────────────────────────────────
  Tool reduction         : 26%  (87 → 64 canonical + unique)
  Token savings estimate : ~340 tokens/turn
────────────────────────────────────────────────────────────
```

---

## Generated config format

```yaml
routing:
  issue.create:
    default: github/create_issue
    overrides:
      - condition: "user mentions 'Jira' or issue ID matches /[A-Z]+-\\d+/"
        use: jira/create_ticket
      - condition: "user mentions 'Linear'"
        use: linear/createIssue
  pr.create:
    default: github/create_pull_request
    overrides:
      - condition: "user mentions 'GitLab' or 'merge request'"
        use: gitlab/create_merge_request

suppressed:
  - linear/createIssue     # exact duplicate of github/create_issue
  - gitlab/list_issues     # exact duplicate of github/list_issues

unique:
  - jira/get_sprint_board  # no equivalent in other servers
  - notion/query_database
```

---

## Deduplication matrix example

```text
                     github   gitlab   jira   linear
─────────────────────────────────────────────────────
issue.create           ●        ○        ↑       ✕
issue.comment          ●        ✕        ✕       –
pr.create              ●        ↑        –       –
pr.merge               ●        ✕        –       –
search.code            ●        ✕        –       –
sprint.board           –        –        ●       –

●  canonical  ○  demoted  ↑  override  ✕  suppressed  –  not available
```

---

## Slash commands

| Command | What it does |
| --- | --- |
| `/scan` | Full scan of all connected servers; output duplicate map |
| `/diff <server-a> <server-b>` | Compare two specific servers tool-by-tool |
| `/abstract <operation>` | Show canonical abstraction for a named operation |
| `/route <query>` | Recommend the single best tool for a natural language task |
| `/matrix` | Full deduplication matrix (all servers × all canonical operations) |
| `/config` | Generate `.mcp-dedup.yml` ready to paste |
| `/audit <server>` | List all tools from one server with duplicate annotations |
| `/stats` | Summary: total tools, duplicates, reduction %, token savings |

---

## Common duplicate patterns

### GitHub + GitLab

| GitHub | GitLab | Class |
| --- | --- | --- |
| `create_pull_request` | `create_merge_request` | Functional equivalent |
| `merge_pull_request` | `merge_merge_request` | Exact duplicate |
| `get_file_contents` | `get_file` | Functional equivalent |
| `search_code` | `search_code` | Near-identical |

### GitHub + Jira

| GitHub | Jira | Class |
| --- | --- | --- |
| `create_issue` | `create_ticket` | Functional equivalent |
| `add_issue_comment` | `add_comment` | Functional equivalent |

### GitHub + Linear

| GitHub | Linear | Class |
| --- | --- | --- |
| `create_issue` | `createIssue` | Exact duplicate |
| `list_issues` | `getIssues` | Exact duplicate |

---

## Installation

1. Add `skills/mcp-deduplicator/SKILL.md` to your Claude project.
2. Either paste your server tool manifests directly, or list the server names you have connected (GitHub, GitLab, Jira, etc.) — the skill has a built-in registry for common servers.
3. Run `/scan` for a full duplicate report, then `/config` to generate the routing config.
4. Add the generated `.mcp-dedup.yml` to your project root.
