# 🤖 Building for Reachy Mini — Hackathon Setup Sheet

Develop against a **simulator on your laptop**, then deploy to the **one physical Reachy Mini (Wireless)** at demo time. The same code runs on sim and hardware — build in sim with confidence.

Works on **Linux, macOS, and native Windows** (no WSL required). Python 3.10+ everywhere.

---

## 1. Set up the simulator

Create and activate a fresh virtualenv, then install the SDK + MuJoCo extra:

| OS | Activate venv | Install |
|---|---|---|
| **Linux / macOS** | `source venv/bin/activate` | `pip install "reachy-mini[mujoco]"` |
| **Windows (PowerShell)** | `venv\Scripts\Activate.ps1` | `uv pip install "reachy-mini[mujoco]"` |

> **Per-OS gotchas:**
> - **macOS:** use **`pip`, not `uv`** — `uv` has known MuJoCo compatibility issues on macOS.
> - **Windows:** if PowerShell blocks the activate script, run once as Admin: `Set-ExecutionPolicy RemoteSigned`.
> - **Linux:** `pip` or `uv` both fine.

**Start the daemon** (keep this terminal open):

```bash
# Linux / Windows:
reachy-mini-daemon --sim

# macOS (needs mjpython for the GUI):
mjpython -m reachy_mini.daemon.app.main --sim
```

✅ Open `http://localhost:8000` — the Reachy Dashboard should load.

---

## 2. Write your first behavior

The sim runs any SDK script unchanged — the same `ReachyMini()` works on hardware.

```python
from reachy_mini import ReachyMini
from reachy_mini.utils import create_head_pose

with ReachyMini() as mini:
    mini.goto_target(head=create_head_pose(z=20, roll=10, mm=True, degrees=True), duration=1.0)
    mini.goto_target(antennas=[0.6, -0.6], duration=0.3)
    mini.goto_target(antennas=[-0.6, 0.6], duration=0.3)
    mini.goto_target(head=create_head_pose(), antennas=[0, 0], duration=1.0)
```

The robot also has a camera, microphones, and speakers — see the SDK docs for vision, audio, and LLM integrations.

---

## 3. Using an AI coding agent? Bootstrap instantly

On **Claude Code, Cursor, Codex, or Copilot**, paste this to get your agent building correctly from the start:

```
I'd like to create a Reachy Mini app. Start by reading
https://github.com/pollen-robotics/reachy_mini/blob/main/AGENTS.md
```

---

## 4. Package as an app

To deploy to the robot, structure your project as a **Reachy Mini app** (subclass `ReachyMiniApp`, exit cleanly via the provided `stop_event`). Start from the template:

- `huggingface.co/spaces/pollen-robotics/reachy_mini_template_app`

Test it locally against the sim:

```bash
pip install -e .
reachy-mini-app-assistant check   # validates your app
```

---

## 5. Publish to a Hugging Face Space

> ⚠️ **Do this on day one.** Requires a Hugging Face account + a **write token**. This is the #1 day-of blocker.

Log in with your token (syntax differs by shell):

```bash
# Linux / macOS (bash/zsh):
hf auth login --token $HF_TOKEN --add-to-git-credential

# Windows (PowerShell):
hf auth login --token $env:HF_TOKEN --add-to-git-credential

# Windows (cmd):
hf auth login --token %HF_TOKEN% --add-to-git-credential
```

Then from your app directory:

```bash
reachy-mini-app-assistant publish
```

Your app appears at `huggingface.co/spaces/<your-username>/<your-app>`.

---

## 6. Deploy to the physical robot (demo time)

The Wireless robot's daemon is always running on the venue network. Two ways to run your app on it:

**A) One-click install (recommended)** — on your Space, click **Install to Robot** → enter `http://reachy-mini.local:8000` (or the IP posted at the venue) → start it from the dashboard.

**B) Run your SDK script directly** — with your laptop on the same network, the constructor auto-detects Wireless and connects. Force it if needed:

```python
mini = ReachyMini(connection_mode="network")
```

---

## ⚠️ Demo-day rules

- **One team at a time.** One set of motors — we run demos as a queue: install → demo → stop → next.
- **Use the IP if `reachy-mini.local` doesn't resolve.** mDNS is flaky and the IP will be posted at the demo station. Note: Windows usually needs **Bonjour** installed for `.local` names — using the IP avoids that entirely.
- **OS network permissions:**
  - **macOS:** System Settings → Privacy & Security → Local Network → allow your browser/terminal.
  - **Windows:** if Windows Defender Firewall prompts on first connection, allow your browser/Python on **Private** networks.
  - **Linux:** usually nothing extra; if `ufw`/`firewalld` is on, allow outbound to port 8000.
- **Test the full flow in sim first.** Parity is high, but validate before your slot.

---

### Handy links
- SDK + docs: `github.com/pollen-robotics/reachy_mini`
- Publishing walkthrough: "Make and Publish Your Reachy Mini Apps" (Pollen Robotics blog)
- Community apps: Hugging Face Spaces → Reachy Mini

**Have fun — build something weird and expressive. 👋**
