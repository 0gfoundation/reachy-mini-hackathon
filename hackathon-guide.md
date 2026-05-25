# 🤖 Building for Reachy Mini × 0G — Hackathon Setup Sheet

Develop against a **simulator**, then deploy to the **one physical Reachy Mini (Wireless)** at demo time. Same code runs on sim and hardware.

Works on **Linux, macOS, and native Windows** (no WSL required).

**Contents:** [Pick your path](#-pick-your-path) · [1. Get 0G API keys](#1-get-your-0g-api-keys-do-this-before-the-event) · [2A. JS path](#2a-path-a--js--static-web-app) · [2B. Python path](#2b-path-b--python-on-robot) · [3. AI coding agent](#3-using-an-ai-coding-agent-bootstrap-instantly) · [Demo-day rules](#-demo-day-rules)

---

## 🛤️ Pick your path

Two ways to build a Reachy Mini app. Pick one and stick with it. Most teams should pick **Path A**.

| | **Path A: JS / Static Web App** | **Path B: Python (on-robot)** |
|---|---|---|
| **Where it runs** | Browser, talks to robot via WebRTC | On the robot's Pi as a registered app |
| **Install** | None. Static HF Space, opens on any device | venv + `pip install reachy-mini[mujoco]` + daemon |
| **Sim** | 3D sim runs in the browser (URDF + Three.js) | MuJoCo desktop sim |
| **Demo flow** | Open URL on the laptop, connect to robot, run | "Install to Robot" from HF Space, daemon runs it |
| **Best for** | LLM apps, UIs, voice loops, anything that fits the WebRTC model. **Recommended by upstream Pollen.** | Heavy compute, deterministic control loops, hardware-side state machines |
| **Starter** | Clone `cduss/webrtc_example` on Hugging Face | `reachy-mini-app-assistant create <name> <path>` |
| **Reference** | [`gathint/reachy-blocks`](https://huggingface.co/spaces/gathint/reachy-blocks) — Scratch-style builder with 0G chat + Whisper already wired up | [`pollen-robotics/reachy_mini_template_app`](https://huggingface.co/spaces/pollen-robotics/reachy_mini_template_app) |

> Pollen's own [AGENTS.md](https://github.com/pollen-robotics/reachy_mini/blob/main/AGENTS.md) says: **"Default to a JS/Web app unless you explicitly need on-robot Python."** Static HF Spaces are zero-install, mobile-friendly, and the same URL works on any laptop at the venue.

Skip to **[Section 2A](#2a-path-a--js--static-web-app)** for the JS path, or **[Section 2B](#2b-path-b--python-on-robot)** for Python.

---

## 1. Get your 0G API keys (do this BEFORE the event)

**This is the #1 day-of blocker.** 0G's Compute marketplace uses **per-provider API keys in Direct mode** — one `app-sk-…` per model you call. The chat key won't work for Whisper, and neither will work without a funded sub-account.

### A. Create a wallet + deposit

1. Open https://pc.0g.ai and connect a wallet.
2. Browse **AI Models** and pick the providers you'll use. Common picks:
   - **Chat**: `deepseek/deepseek-chat-v3-0324`, `zai-org/GLM-5-FP8`, or `qwen3.6-plus` (each is a separate provider).
   - **Speech-to-text**: `openai/whisper-large-v3`.
   - **Vision / image**: `qwen/qwen3-vl-30b-a3b-instruct`, `z-image`.
3. Click into each provider you want → deposit at least **1 0G** to that provider's sub-account. **Each model = its own deposit.** A balance on the chat provider does NOT cover Whisper.

### B. Mint per-provider API keys

For every provider you funded, generate its `app-sk-…` token:

```bash
0g-compute-cli inference get-secret --provider <PROVIDER_ADDRESS>
```

The provider address is shown on the model card at pc.0g.ai. The token format is `app-sk-<base64(rawMessage:signature)>` — it's bound to that one provider's address. Keep these secret; they're per-account.

### C. Know your endpoint URLs

Each provider lives on a **different `compute-network-N` host**. Examples (verify on pc.0g.ai, they can change):

- Chat (deepseek/GLM): `https://compute-network-1.integratenetwork.work/v1/proxy/chat/completions`
- Whisper STT: `https://compute-network-16.integratenetwork.work/v1/proxy/audio/transcriptions`

If you hit the wrong host with a key, the proxy returns `400 Provider proxy: validate session: missing or invalid Authorization header`. That error means **wrong key for this host**, not a malformed token.

### D. Smoke-test with curl before you write any app code

```bash
curl -sS https://compute-network-1.integratenetwork.work/v1/proxy/chat/completions \
  -H "Authorization: Bearer $OG_CHAT_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"zai-org/GLM-5-FP8","messages":[{"role":"user","content":"hi"}],"max_tokens":5}'
```

200 with a JSON response = key + deposit + endpoint all good. Anything else, fix it before building.

> Working JS + Python helpers (chat, whisper STT, error handling) are in **[starter-snippets.md](starter-snippets.md)**. Copy them — don't reinvent.

---

## 2A. Path A — JS / Static Web App

### A. Clone the template

The canonical JS template is the WebRTC example Space:

```bash
git clone https://huggingface.co/spaces/cduss/webrtc_example my-reachy-app
cd my-reachy-app
```

For a production-grade reference that already wires 0G chat, Whisper STT, TTS, and a 3D simulator — see **[`gathint/reachy-blocks`](https://huggingface.co/spaces/gathint/reachy-blocks)**. Forking that is the fastest way to ship.

### B. Pin the SDK version

In `index.html`, the SDK is imported from jsdelivr. **Pin to a version tag**, never `@main`:

```html
<script type="module">
  import { ReachyMini } from "https://cdn.jsdelivr.net/gh/pollen-robotics/reachy_mini@v1.7.1/js/reachy-mini.js";
</script>
```

Check the latest tag: `git ls-remote --tags https://github.com/pollen-robotics/reachy_mini | grep -E 'v[0-9]' | tail`.

### C. Local dev

Serve over HTTP (not `file://`):

```bash
python3 -m http.server 8765
```

Open `http://localhost:8765/?hf=<your-read-token>`. **Important:** the JS SDK does **not** auto-read the `?hf=` URL param — `authenticate()` only checks OAuth callbacks and sessionStorage. You must parse it yourself and pass to `robot.connect(token)`. Reachy Blocks has this pattern; copy it.

### D. Run against the on-stage robot

When connected to venue Wi-Fi, the SDK auto-discovers the robot via the relay. The same code runs against the on-stage Wireless without changes.

### E. Publish to a HF Space (do this day one)

Create a static Space at https://huggingface.co/new-space (SDK: Static), then:

```bash
git remote set-url origin https://huggingface.co/spaces/<your-user>/<your-app>
git push -u origin main
```

Your app is now at `https://<your-user>-<your-app>.static.hf.space/` — share that URL during your demo slot. OAuth (`robot.login()`) only works from the deployed `*.static.hf.space` origin, not from localhost.

### JS gotchas (read these, save hours)

- **Do NOT call `navigator.mediaDevices.getUserMedia()`.** Robot camera + mic stream over the WebRTC peer connection automatically. `getUserMedia` grabs the user's laptop devices, which is wrong.
- **Do NOT call `robot.setMicMuted(false)`.** That unmutes the *outgoing* laptop-mic track, and the daemon plays unmuted outgoing audio through the robot's speaker (loopback that won't stop until you mute again).
- **For STT: grab the robot's incoming WebRTC audio track**, not laptop mic. The Reachy Blocks `listenForSpeech()` is a clean reference.
- **`await robot.ensureAwake()` after `startSession()`** or motion commands silently no-op.

---

## 2B. Path B — Python (on-robot)

### A. Create and activate a virtualenv

**Linux / macOS:**

```bash
python3 -m venv venv
```

```bash
source venv/bin/activate
```

**Windows (PowerShell)** — if activation is blocked, run once as Admin: `Set-ExecutionPolicy RemoteSigned`.

```powershell
python -m venv venv
```

```powershell
venv\Scripts\Activate.ps1
```

### B. Install the SDK with the MuJoCo extra

**Linux / macOS** — use `pip`. On macOS, `uv` has known MuJoCo compatibility issues, so avoid it here too.

```bash
pip install "reachy-mini[mujoco]"
```

**Windows (PowerShell)** — `uv` works fine.

```powershell
uv pip install "reachy-mini[mujoco]"
```

### C. Start the daemon (keep this terminal open)

**Linux / Windows:**

```bash
reachy-mini-daemon --sim
```

**macOS** — needs `mjpython` to drive the GUI.

```bash
mjpython -m reachy_mini.daemon.app.main --sim
```

✅ Open `http://localhost:8000` — the Reachy Dashboard should load.

### D. Write your first behavior

```python
from reachy_mini import ReachyMini
from reachy_mini.utils import create_head_pose

with ReachyMini() as mini:
    mini.goto_target(head=create_head_pose(z=20, roll=10, mm=True, degrees=True), duration=1.0)
    mini.goto_target(antennas=[0.6, -0.6], duration=0.3)
    mini.goto_target(antennas=[-0.6, 0.6], duration=0.3)
    mini.goto_target(head=create_head_pose(), antennas=[0, 0], duration=1.0)
```

### E. Package as an app

Subclass `ReachyMiniApp`, exit cleanly via the provided `stop_event`. Start from the template:

- `huggingface.co/spaces/pollen-robotics/reachy_mini_template_app`

Install in editable mode, then validate against the sim:

```bash
pip install -e .
```

```bash
reachy-mini-app-assistant check
```

### F. Publish to a Hugging Face Space

> ⚠️ **Do this day one.** Requires a Hugging Face account + a **write token**. This is the #1 day-of blocker.

Log in with your token:

```bash
hf auth login --token $HF_TOKEN --add-to-git-credential
```

(Windows PowerShell: `$env:HF_TOKEN`; Windows cmd: `%HF_TOKEN%`.)

Then from your app directory:

```bash
reachy-mini-app-assistant publish
```

Your app appears at `huggingface.co/spaces/<your-username>/<your-app>`.

### G. Deploy to the physical robot

Two ways:

**A) One-click install (recommended)** — on your Space, click **Install to Robot** → enter `http://reachy-mini.local:8000` (or the IP posted at the venue) → start it from the dashboard.

**B) Run your SDK script directly** — with your laptop on the same network, the constructor auto-detects Wireless and connects. Force it if needed:

```python
mini = ReachyMini(connection_mode="network")
```

---

## 3. Using an AI coding agent? Bootstrap instantly

On **Claude Code, Cursor, Codex, or Copilot**, paste this to get your agent building correctly from the start:

```
I'd like to create a Reachy Mini app. Start by reading
https://github.com/pollen-robotics/reachy_mini/blob/main/AGENTS.md
then check the 0G hackathon guide at
https://github.com/0gfoundation/reachy-mini-hackathon/blob/main/hackathon-guide.md
```

---

## ⚠️ Demo-day rules

- **One team at a time.** One set of motors. We run demos as a queue: install → demo → stop → next.
- **Use the IP if `reachy-mini.local` doesn't resolve.** mDNS is flaky and the IP will be posted at the demo station. Windows usually needs **Bonjour** installed for `.local` names; using the IP avoids that.
- **OS network permissions:**
  - **macOS:** System Settings → Privacy & Security → Local Network → allow your browser/terminal.
  - **Windows:** if Windows Defender Firewall prompts on first connection, allow your browser/Python on **Private** networks.
  - **Linux:** usually nothing extra; if `ufw`/`firewalld` is on, allow outbound to port 8000.
- **Test the full flow in sim first.** Parity is high, but validate before your slot.

---

### Handy links

- SDK: [github.com/pollen-robotics/reachy_mini](https://github.com/pollen-robotics/reachy_mini)
- Docs: [huggingface.co/docs/reachy_mini](https://huggingface.co/docs/reachy_mini)
- Pollen's [AGENTS.md](https://github.com/pollen-robotics/reachy_mini/blob/main/AGENTS.md) (load into your coding agent)
- Publishing walkthrough: [Make and Publish Your Reachy Mini Apps](https://huggingface.co/blog/pollen-robotics/make-and-publish-your-reachy-mini-apps) (Pollen Robotics blog)
- Community apps: [Reachy Mini Spaces](https://huggingface.co/spaces/pollen-robotics/Reachy_Mini)
- 0G compute marketplace: https://pc.0g.ai

**Have fun — build something weird and expressive. 👋**
