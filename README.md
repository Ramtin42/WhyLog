# WhyLog

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE) [![Zero Dependencies](https://img.shields.io/badge/dependencies-zero-brightgreen.svg)](#) [![Claude Code](https://img.shields.io/badge/Claude_Code-plugin-blueviolet.svg)](#)

**Decision capture for AI-generated codebases.**

A free, zero-dependency Claude Code plugin that ensures every architecture and design decision made during a coding session gets documented with its reasoning: the tradeoffs, the alternatives considered, and why the chosen approach won.

**The problem:** AI-native teams (80%+ AI-generated code) move fast but lose the "why" behind their codebase. Three months later, nobody knows why Redis was chosen over Postgres for caching, why the auth flow works the way it does, or why that weird pattern in the API layer exists. Engineers delete code that was there for a reason nobody documented. The AI approaches the same problem differently across sessions, creating patchwork architecture. Docs drift, and the codebase becomes a black box that only works by accident.

**The solution:** A CLAUDE.md snippet + a single markdown file. The instructions tell Claude to read `docs/decisions.md` before making changes and append new ADRs after. That's the entire mechanism. No dependencies, no services, no setup beyond copy-paste.

## Table of Contents

- [Quick Start](#quick-start)
- [File 1: docs/decisions.md](#file-1-docsdecisionsmd)
- [File 2: Append to your CLAUDE.md](#file-2-append-to-your-claudemd)
- [ADR Format](#adr-format)
- [Optional: Enforcement Hook](#optional-enforcement-hook)
- [Optional: strategy.md](#optional-strategymd)
- [Why This Exists](#why-this-exists)
- [Contributing](#contributing)

## Quick Start

1. Create `docs/decisions.md` in your project root (copy [File 1](#file-1-docsdecisionsmd) below)
2. Append [File 2](#file-2-append-to-your-claudemd) to your project's `CLAUDE.md`
3. Done

---

## File 1: `docs/decisions.md`

Create this file at `docs/decisions.md`:

```markdown
# Architecture Decision Records

> Append new ADRs at the bottom. Never edit past ADRs; supersede them with a new entry.
> Format: terse, concrete, no filler. Capture the *why* from the conversation.

---
```

---

## File 2: Append to your `CLAUDE.md`

```markdown
## Decision Records (docs/decisions.md)

Architecture decisions are tracked in `docs/decisions.md`. This is the memory of *why* things are the way they are.

Succinctness is a core principle. ADRs must be the minimum words needed to preserve the reasoning. No "we carefully considered"; just what was considered and what won. No restating the obvious. No filler, no philosophizing. If it reads like a blog post, it failed. If it reads like a commit message with tradeoffs, it worked.

### Before any architecture/design change, deletion of existing code, or new approach to a problem:
1. Read `docs/decisions.md`. Has this been decided before? Is there reasoning you'd be overriding?
2. If conflicting with a past ADR: flag it, explain why you're diverging, add a new ADR that supersedes the old one (set old status to `Superseded by ADR-NNN`)
3. Never delete code that exists for a documented reason without updating the ADR
4. After compaction, re-read `docs/decisions.md` before making design choices to avoid duplicating entries

### After making a design/architecture choice in this session:
- Append a new ADR to `docs/decisions.md` when the decision is final, after discussion concludes and implementation is done, not mid-deliberation
- Capture the reasoning from our conversation: tradeoffs weighed, alternatives rejected and why
- If the human overrode your recommendation, document their reasoning for doing so. That override IS the decision
- Deciding NOT to change something after evaluating alternatives is also a decision worth documenting
- Link the ADR to the files/modules it affects (e.g. "Applies to: src/auth/")
- If this ADR depends on or extends a previous one, reference it (e.g. "Builds on ADR-001")
- Format: `## ADR-NNN: [title]`. Only Date, Status, and Decision are required. Add Context, Alternatives, Consequences, Applies to, Builds on, or Notes when they add value. Add a Notes field for meaningful context that doesn't fit elsewhere
- Status lifecycle: Proposed → Accepted → Superseded by ADR-NNN | Deprecated

### What IS an ADR:
- Decisions that affect multiple files or modules
- Choices that would surprise a new engineer reading the code
- Anything that constrains future choices (picked a framework, chose a pattern, set a convention)

### What is NOT an ADR:
- Routine bug fixes, formatting, dependency bumps, anything where "why" is self-evident
- Function-level implementation details (use code comments for those)
- Choices with only one reasonable option
```

---

## ADR format

Only **Date**, **Status**, and **Decision** are required. Include the other fields when they add value; skip them when they don't. A two-line ADR that captures the right "why" beats a fully filled template that says nothing useful.

Add a **Notes** field for anything meaningful that doesn't fit the standard fields: links to relevant discussions, performance benchmarks, compliance requirements, migration concerns, etc.

```markdown
## ADR-001: Supabase over raw Postgres
- **Date**: 2025-03-28
- **Status**: Accepted
- **Decision**: Use Supabase as primary database and auth provider.
- **Context**: Need Postgres, auth, and realtime capabilities.
- **Alternatives**: Raw Postgres (RDS/Neon), self-hosted Supabase
- **Consequences**: Vendor dependency. Get auth, RLS, realtime out of the box.
- **Applies to**: src/db/, src/auth/
- **Notes**: Free tier sufficient for current scale. Re-evaluate at ~10k MAU.
---
```

---

## Optional: Enforcement hook

The CLAUDE.md instructions get you ~80% compliance. If you want ~95%, add a Stop hook that blocks Claude from finishing when it makes an undocumented decision.

Add to `.claude/settings.json`:

```json
{
  "hooks": {
    "Stop": [
      {
        "type": "prompt",
        "model": "haiku",
        "prompt": "Review the assistant's last message. Did it make or implement an architecture/design decision, e.g. chose a technology, designed a data model, structured code a specific way over alternatives, deleted or replaced existing patterns, or approached a problem in a way that involved tradeoffs? If YES: check if docs/decisions.md was updated with an ADR capturing the reasoning. If a decision was made but NOT documented, respond with {\"ok\": false, \"reason\": \"You made a design decision but didn't document the reasoning in docs/decisions.md. Append an ADR now. Capture the why, tradeoffs, and alternatives from this conversation.\"}. If NO design decision was made, or it was already documented, respond with {\"ok\": true}."
      }
    ]
  }
}
```

Fires on every Claude response. Haiku evaluates and passes silently on routine code. Adds small latency. Try it; remove it if it's too noisy.

---

## Optional: `strategy.md`

If your team doesn't already have an `architecture.md`, `plan.md`, or similar:

```markdown
# Architecture Strategy

> How this codebase is built. Updated only when decisions.md establishes or changes a pattern.
> Last updated: YYYY-MM-DD

## Stack
- **Backend**:
- **Frontend**:
- **Database**:
- **Auth**:
- **Hosting**:

## Principles
-

## Conventions
-

## Boundaries (things we explicitly don't do)
-
```

Add to CLAUDE.md: `Read docs/strategy.md before making structural choices to ensure consistency.`

---

## Why this exists

AI-native teams generate code fast but lose the reasoning behind it. Decisions get made in conversation, implemented immediately, and the reasoning evaporates when the session ends.

WhyLog captures the in-session reasoning, the ~40-50% that happens inside AI coding conversations. For the rest (cross-tool reasoning from Slack, PRs, design docs, engineer knowledge), there's [ReasonForge](https://reasonforge.ai).

---

## Compatibility

Works with Claude Code (any version that supports CLAUDE.md and hooks).

## Contributing

Issues and PRs welcome.

## License

MIT

*By [ReasonForge](https://reasonforge.ai): stop losing the reasoning behind your engineering decisions.*
