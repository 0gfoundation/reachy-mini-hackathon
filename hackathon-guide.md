# 🤖 Building for Reachy Mini — Hackathon Setup Sheet

You'll develop against a **simulator on your own laptop**, then deploy to the **one physical Reachy Mini (Wireless)** during demos. Code written for sim runs unchanged on the real robot — so build with confidence in sim.

---

## 1. Set up the simulator (no hardware needed)

You need Python 3.10+ and the SDK with the MuJoCo extra.

```bash
# Create and activate a virtual environment first, then:
pip install reachy-mini[mujoco]
```

> **macOS users — read this:**
> - Use **`pip`, not `uv`**, for the MuJoCo packages — `uv` has known compatibility issues with MuJoCo on macOS.
> - You must launch the sim with **`mjpython`** (not `python`) for the 3D GUI to work.

**Run the simulation daemon** (keep this terminal open):

```bash
# macOS:
mjpython -m reachy_mini.daemon --sim
# Linux:
python -m reachy_mini.daemon --sim
```

A 3D MuJoCo window opens. The sim behaves exactly like a real robot on `localhost` — drag to rotate, scroll to zoom.

✅ **Check it's working:** open `http://localhost:8000` — you should see the Reachy Dashboard.

---

## 2. Write your first behavior

The sim listens on localhost and runs any SDK script **without modification**. Same `ReachyMini()` constructor works on sim *and* hardware.

```python
from reachy_mini import ReachyMini
from reachy_mini.utils import create_head_pose

with ReachyMini() as mini:
    print("Connected!")
    # Look up + tilt head
    mini.goto_target(
        head=create_head_pose(z=20, roll=10, mm=True, degrees=True),
        duration=1.0,
    )
    # Wiggle antennas
    mini.goto_target(antennas=[0.6, -0.6], duration=0.3)
    mini.goto_target(antennas=[-0.6, 0.6], duration=0.3)
    # Reset
    mini.goto_target(head=create_head_pose(), antennas=[0, 0], duration=1.0)
```

The robot has a **camera, microphones, and speakers** too — explore the SDK docs for vision/audio and LLM integrations.

---

## 3. Using an AI coding agent? Bootstrap instantly

If you're on **Claude Code, Cursor, Codex, or Copilot**, paste this to your agent to get it building correctly from the start:

```
I'd like to create a Reachy Mini app. Start by reading
https://github.com/pollen-robotics/reachy_mini/blob/main/AGENTS.md
```

The repo's `skills/` directory has detailed guides the agent will use to scaffold a valid app.

---

## 4. Package your project as an app

To deploy to the robot, structure your project as a **Reachy Mini app** (inherits from `ReachyMiniApp`, exits cleanly via the provided `stop_event`). Start from the official template:

- Template: `huggingface.co/spaces/pollen-robotics/reachy_mini_template_app`

Typical layout:

```
my_app/
├── index.html          # landing page for your Space
├── pyproject.toml
├── my_app/
│   ├── __init__.py
│   └── main.py         # your ReachyMiniApp subclass
├── README.md
└── style.css
```

Install it locally to test against the sim:

```bash
pip install -e .
reachy-mini-app-assistant check   # validates your app
```

---

## 5. Publish to a Hugging Face Space

> ⚠️ **Do this early — day one.** Publishing requires a Hugging Face account and a **write token**. This is the #1 day-of blocker.

```bash
# Create a free HF account at huggingface.co, then:
hf auth login --token $HF_TOKEN --add-to-git-credential

# From your app directory:
reachy-mini-app-assistant publish
```

This creates a Space under your account. Your app then appears at
`huggingface.co/spaces/<your-username>/<your-app>`.

---

## 6. Deploy to the physical robot (demo time)

The robot is **Wireless** — its daemon is always running, and it's on the venue network. There are two ways to run your app on it:

**A) One-click install (recommended)**
1. Open your app's Hugging Face Space.
2. Click **Install to Robot**.
3. Enter the robot's dashboard URL: **`http://reachy-mini.local:8000`** (or the IP we'll post at the venue).
4. Start it from the dashboard.

**B) Run your SDK script over the network**
With your laptop on the same network as the robot, your script connects automatically (the constructor auto-detects Wireless and switches to network mode). For edge cases:
```python
mini = ReachyMini(connection_mode="network")
```

---

## ⚠️ Demo-day rules (important)

- **One team drives at a time.** The robot has one set of motors — only one app/script can control it simultaneously. We'll run demos as a queue: install/run → demo → stop → next team.
- **Connect by IP if `reachy-mini.local` doesn't resolve.** mDNS is flaky on some networks; the robot's IP will be posted at the demo station.
- **macOS Local Network permission:** if your laptop can't see the robot, check System Settings → Privacy & Security → Local Network and allow your browser/terminal.
- **Test everything in sim first.** Sim ↔ hardware parity is high, so a working sim demo should "just work" on the robot — but validate your full flow before your slot.

---

### Handy links
- SDK + docs: `github.com/pollen-robotics/reachy_mini`
- App publishing walkthrough: search "Make and Publish Your Reachy Mini Apps" (Pollen Robotics blog)
- Browse community apps for inspiration: Hugging Face Spaces → Reachy Mini

**Have fun — build something weird and expressive. 👋**
