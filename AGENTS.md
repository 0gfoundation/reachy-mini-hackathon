# AGENTS.md

Guidance for AI coding agents (and humans) working in this repo. Portable across agent tooling — Claude Code, Cursor, Aider, Codex, etc.

## What this repo is

Organizing materials for a **hackathon where teams build apps for the Reachy Mini robot that integrate with the 0G stack** (Chain, Compute, Storage). This repo holds the run-of-show docs, not application code. The event has **one physical Reachy Mini (Wireless)**; many teams develop in simulation and deploy to that single robot during demos.

## Contents

| File | Purpose |
|---|---|
| `hackathon-guide.md` | Attendee setup sheet — sim install, first script, app publishing, deployment. |
| `demo-station.md` | Operator card for whoever runs the robot during demos — queue, stop/reset, fixes. |
| `judging-rubric.md` | Fast 0G-centered scoring sheet (out of 25). |
| `project-inspiration.md` | 1–2 day project ideas, grouped by 0G service. |

## Key technical facts (verify against current beta before relying on these)

The Reachy Mini SDK is in active beta; CLI flags and service names shift between releases. Treat the below as the working model, but check the live docs/`--help` when something doesn't match.

**Development model**
- SDK: `reachy-mini` (Python 3.10+). Simulator uses MuJoCo: `pip install reachy-mini[mujoco]`.
- **Sim ↔ hardware parity is the core principle**: the same `ReachyMini()` code runs against the simulator and the real robot unchanged. Build in sim, deploy to hardware.
- macOS gotchas: use **`mjpython`** (not `python`) to launch the sim GUI; use **`pip`, not `uv`**, for MuJoCo packages.
- AI agents can bootstrap by reading `https://github.com/pollen-robotics/reachy_mini/blob/main/AGENTS.md` (the repo's `skills/` dir has app-building guides).

**Architecture**
- A **daemon** (FastAPI server on port 8000) bridges code and hardware. On **Wireless**, it runs onboard the Raspberry Pi and is reachable at `http://reachy-mini.local:8000` (or the robot's IP). On **Lite**, it runs on the host computer at `localhost:8000`.
- REST API at `/api`; interactive Swagger at `/docs`.
- **Apps**: subclass `ReachyMiniApp`, exit via the provided `stop_event`. Created/packaged with the `reachy-mini-app-assistant` CLI, **published to a Hugging Face Space** (requires a HF account + write token), then **installed onto the robot via the dashboard** ("Install to Robot" → enter dashboard URL) or REST.
- **Only one app runs at a time.** The daemon launches the app as a subprocess; on stop it sends SIGINT and **returns the robot to its default pose** automatically.
- Hardware exposes: head pose, antennas, camera, microphones, speakers.

**The 0G angle (this event's whole point)**
- Projects must meaningfully use ≥1 of: **0G Compute** (decentralized/TEE LLM + vision inference), **0G Storage** (persistent memory, media, behavior libraries), **0G Chain** (agent identity / iNFTs, ownership, micro-payments).
- Common patterns: Compute for the robot's conversation/vision, Storage for agent memory, Chain for identity or pay-per-interaction.

## Demo-day operational notes
- Get the robot on a **controlled network** (bring a travel router / dedicated SSID). Venue Wi-Fi often blocks client-to-client traffic and breaks `.local` mDNS resolution — fall back to the robot's IP.
- macOS clients may need Local Network permission (System Settings → Privacy & Security → Local Network).
- Reset ladder when stuck: dashboard **Stop** → restart daemon → SSH (`root@reachy-mini.local`) `systemctl restart reachy-mini-daemon` → power-cycle.

## Conventions for editing these docs
- Keep them **short and scannable** — these are used live, under time pressure.
- The judging rubric is deliberately fast (~2 min/team); don't add criteria without removing others.
- Fill bracketed/blank values (robot IP, etc.) before the event rather than hardcoding guesses.

## Keeping AGENTS.md and CLAUDE.md in sync

`CLAUDE.md` imports this file via `@AGENTS.md` and adds only Claude-specific instructions on top. When updating agent guidance:
- Put portable instructions **here** (AGENTS.md), not in CLAUDE.md.
- Put Claude-Code-specific instructions (slash commands, hooks, harness behavior, tool preferences) in CLAUDE.md.
- Never duplicate content between the two files — if something belongs in both, it belongs in AGENTS.md.
