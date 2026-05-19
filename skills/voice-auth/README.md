# Voice Authenticator

Makes AI output sound like the actual user — not generic, not polished-AI, not
"in today's rapidly evolving landscape."

---

## The Problem

AI writing has a smell. It opens with context-setting instead of a point. It uses
Latinate vocabulary where plain words work. It structures everything as three-part
lists. It ends with summaries no one asked for.

Editing the AI smell out takes 30–45 minutes per output. And often the audience can
still tell.

This skill fingerprints your voice from a writing sample, then generates content
that matches your structural habits, rhythm, and vocabulary — not the AI defaults.

---

## Workflow

```text
User provides a writing sample (min 2 paragraphs)
    │
    ├─ Extract voice fingerprint:
    │   sentence rhythm, vocabulary/register,
    │   structural habits, punctuation patterns
    │
    ├─ Run anti-AI checklist — identify what to kill
    │
    ├─ Generate or rewrite using the fingerprint as the structure model
    │
    ├─ Self-check: AI smell score (0-10) + fingerprint match (0-10)
    │   Rewrite if smell score > 5
    │
    └─ On feedback: update the fingerprint model, not just the output
```

---

## Slash Commands

| Command | Action |
| --- | --- |
| `/fingerprint` | Analyze a writing sample and output the voice fingerprint |
| `/write <topic>` | Write content using the stored fingerprint |
| `/rewrite <text>` | Rewrite AI output using the stored fingerprint |
| `/check <text>` | Score text for AI smell and flag specific violations |
| `/calibrate <feedback>` | Update the voice model based on feedback |
| `/diff <original> <rewrite>` | Show what changed and why |
| `/anti-ai` | Run the AI cliché checklist on pasted text |
| `/reset` | Clear the stored fingerprint |

---

## Anti-AI Checklist (sample)

**Banned openers:** "In today's rapidly evolving landscape," "It goes without saying,"
"Now more than ever," "At the intersection of"

**Corporate verbs to replace:** leverage → use, utilise → use, facilitate → help,
commence → start, endeavour → try, demonstrate → show, synergise → work together

**Structural tells:** Three-part lists as default structure, "dive into / delve into",
"realm" and "landscape" as domain metaphors, excessive bolding

---

## Installation

Add to your `.claude/skills/` directory or reference in your `CLAUDE.md`:

```yaml
skills:
  - name: voice-auth
    path: skills/voice-auth/SKILL.md
    use-when: >
      User wants AI writing to match their voice. Activate on: "make this sound
      like me", "remove the AI smell", "too formal", "too polished", "sounds like
      ChatGPT", "write in my voice", "match my style", "this doesn't sound like me".
```

---

## Example

**Original AI output:**
> It's important to note that when building a product roadmap, there are several
> key considerations to keep in mind. First, you'll want to ensure alignment
> with stakeholder expectations. Second, prioritising customer feedback is crucial.

**Fingerprint:** Short paragraphs. Problem-first openers. Contractions frequent.
Em-dashes for asides. No bullets in prose.

**Rewritten:**
> Roadmaps fail when they're built for stakeholders instead of customers.
> It's an easy trap — the people in the room have loud opinions, and the
> people paying have quiet ones.
>
> Start with what customers told you they couldn't live without. Everything
> else is negotiable.
