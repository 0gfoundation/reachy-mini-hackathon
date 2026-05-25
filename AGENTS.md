# AGENTS.md

Guidance for AI coding agents (and humans) working in this repo. Portable across agent tooling — Claude Code, Cursor, Aider, Codex, etc.

## What this repo is

Run-of-show docs for a hackathon where teams build apps for the **Reachy Mini robot** that integrate with the **0G stack** (Chain, Compute, Storage). This repo is **docs only — no application code**. One physical Reachy Mini (Wireless) on site; most teams build in simulation and deploy at demo time.

## Where to find what

This repo is content, not code. Before answering a technical question or making edits, **read the relevant doc** — its content is authoritative; this file is just a map.

| File | Read it for… |
|---|---|
| `hackathon-guide.md` | Attendee setup. Section 0 = 0G key prep (the #1 day-of blocker). Section 1A = JS / WebRTC path. Section 1B = Python / on-robot path. Plus publishing + deployment + demo-day rules. |
| `starter-snippets.md` | Working 0G API helpers (chat, Whisper STT) for JS and Python, plus a reference implementation Space. |
| `demo-station.md` | Operator card for whoever runs the robot during demos — queue, stop/reset, fixes. |
| `judging-rubric.md` | Fast 0G-centered scoring sheet (out of 25). |
| `project-inspiration.md` | 1–2 day project ideas, grouped by 0G service. |

For Reachy Mini SDK questions, go upstream: `github.com/pollen-robotics/reachy_mini`. Pollen's own [AGENTS.md](https://github.com/pollen-robotics/reachy_mini/blob/main/AGENTS.md) is the source of truth for SDK conventions (e.g. JS as the default app path, Python for on-robot/hardware work). The SDK is in active beta — verify against live docs/`--help` when something doesn't match.

## Conventions for editing these docs

- Keep them **short and scannable** — they're used live, under time pressure.
- The judging rubric is deliberately fast (~2 min/team); don't add criteria without removing others.
- Fill bracketed/blank values (robot IP, etc.) before the event rather than hardcoding guesses.
- **Code-block format**: every installation/setup snippet must be a **single-line fenced code block with no inline comments**, so GitHub's copy button produces a clean copy. Put OS/shell labels and explanations in **prose above** the block, not as `# Linux:` comments inside it. When the same step differs by OS or shell, give one block per variant (e.g., separate blocks for bash, PowerShell, cmd).
- Cover **Linux, macOS, and native Windows** for all setup instructions. Use the correct shell syntax per OS (`source venv/bin/activate` vs `venv\Scripts\Activate.ps1`; `$VAR` vs `$env:VAR` vs `%VAR%`).

## Keeping AGENTS.md and CLAUDE.md in sync

`CLAUDE.md` imports this file via `@AGENTS.md` and adds only Claude-specific instructions on top. When updating agent guidance:
- Put portable instructions **here** (AGENTS.md), not in CLAUDE.md.
- Put Claude-Code-specific instructions (slash commands, hooks, harness behavior, tool preferences) in CLAUDE.md.
- Never duplicate content between the two files — if something belongs in both, it belongs in AGENTS.md.
