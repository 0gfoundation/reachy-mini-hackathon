# 🤖 Building for Reachy Mini × 0G — Hackathon Setup Sheet

Develop against a **simulator**, then deploy to the **one physical Reachy Mini (Wireless)** at demo time. Same code runs on sim and hardware.

Works on **Linux, macOS, and native Windows** (no WSL required).

**Contents:**

- [Pick your path](#-pick-your-path)
- [1. Get 0G API keys](#1-get-your-0g-api-keys-do-this-before-the-event)
- [2A. JS path](#2a-path-a--js--static-web-app)
- [2B. Python path](#2b-path-b--python-on-robot)
- [3. Showcase your project](#3-showcase-your-project)
- [4. AI coding agent](#4-using-an-ai-coding-agent-bootstrap-instantly)
- [Demo-day rules](#-demo-day-rules)

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
| **Starter** | [`starters/js-voice-chat/`](starters/js-voice-chat/) in this repo — single-file voice chat with 0G Compute, Whisper STT, TTS, and a 3D sim. Zero build step. | `reachy-mini-app-assistant create <name> <path>` |
| **Example app** | [`gathint/reachy-blocks`](https://huggingface.co/spaces/gathint/reachy-blocks) — Scratch-style builder with 0G chat + Whisper, as a polished published reference | [`pollen-robotics/reachy_mini_template_app`](https://huggingface.co/spaces/pollen-robotics/reachy_mini_template_app) |

> Pollen's own [AGENTS.md](https://github.com/pollen-robotics/reachy_mini/blob/main/AGENTS.md) says: **"Default to a JS/Web app unless you explicitly need on-robot Python."** Static HF Spaces are zero-install, mobile-friendly, and the same URL works on any laptop at the venue.

Skip to **[Section 2A](#2a-path-a--js--static-web-app)** for the JS path, or **[Section 2B](#2b-path-b--python-on-robot)** for Python.

---

## 1. Get your 0G API keys (do this BEFORE the event)

**This is the #1 day-of blocker.** 0G's Compute marketplace uses **per-provider API keys** — one `app-sk-…` per model you call. **You must mint the key in Advanced mode**.

The walkthrough below uses the **pc.0g.ai** web UI. If you prefer the CLI, expand the section below for the full equivalent flow.

<details>
<summary><b>Prefer the CLI?</b> Full equivalent flow using the 0G Compute SDK</summary>

See the [0G Compute TS SDK README](https://github.com/0gfoundation/0g-compute-ts-sdk) for full setup details.

Install the SDK CLI globally:

```bash
npm install @0gfoundation/0g-compute-ts-sdk -g
```

One-time network setup and wallet login:

```bash
0g-compute-cli setup-network
```

```bash
0g-compute-cli login
```

Deposit 0G to your main account:

```bash
0g-compute-cli deposit --amount 1
```

List available providers (output includes each one's address and endpoint URL):

```bash
0g-compute-cli inference list-providers
```

Transfer funds from your main account to a provider's sub-account:

```bash
0g-compute-cli transfer-fund --provider <PROVIDER_ADDRESS> --amount 1
```

Acknowledge the provider before first use:

```bash
0g-compute-cli inference acknowledge-provider --provider <PROVIDER_ADDRESS>
```

Mint the `app-sk-…` token for the provider:

```bash
0g-compute-cli inference get-secret --provider <PROVIDER_ADDRESS>
```

</details>

### A. Create a wallet + deposit

1. Open [pc.0g.ai](https://pc.0g.ai), connect a wallet and switch to Advanced mode.
2. Head to the Playground section to browse **AI Models** and pick the providers you'll use. Common picks:
   - **Chat**: `deepseek/deepseek-chat-v3-0324`, `zai-org/GLM-5-FP8`, or `qwen3.6-plus` (each is a separate provider).
   - **Speech-to-text**: `openai/whisper-large-v3`.
   - **Vision / image**: `qwen/qwen3-vl-30b-a3b-instruct`, `z-image`.
3. Click "Use" on each provider you want, then select "Fund" to deposit 1 0G to that provider's sub-account. **Each model = its own deposit.**

> ⚠️ **Top up small amounts at a time.** Withdrawing funds back out of a provider sub-account takes time, so don't lock up all your 0G upfront — start small and add more only if you need it.

> 💧 **Testing on testnet?** Grab free 0G tokens from the faucet at [faucet.0g.ai](https://faucet.0g.ai) — enough to fund a few provider sub-accounts for development.

### B. Mint per-provider API keys

For every provider you funded, generate its `app-sk-…` token under the API Reference section. **You must mint the key in Advanced mode** — keys generated any other way won't authorize against the proxy.

The provider address is shown on this same page. The token format is `app-sk-<base64(rawMessage:signature)>` — it's bound to that one provider's address. Keep these secret; they're per-account.

### C. Know your endpoint URLs

Each provider lives on a **different `compute-network-N` host**. You can find this listed under the API Reference section. For example:

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

### A. Get the JS starter

The default JS starter is **[`starters/js-voice-chat/`](starters/js-voice-chat/)** in this repo — a single-file voice chat app with 0G Compute chat, Whisper STT, Web Speech TTS, expressive movement, and a 3D simulator. Zero build step.

Clone this repo, then copy the starter to a fresh directory.

```bash
git clone https://github.com/0gfoundation/reachy-mini-hackathon
```

**Linux / macOS:**

```bash
cp -r reachy-mini-hackathon/starters/js-voice-chat my-reachy-app
```

**Windows (PowerShell):**

```powershell
Copy-Item -Recurse reachy-mini-hackathon\starters\js-voice-chat my-reachy-app
```

Then:

```bash
cd my-reachy-app
```

```bash
git init
```

See `starters/js-voice-chat/README.md` for run + configure details.

For an example of a polished, published app on this stack, see **[`gathint/reachy-blocks`](https://huggingface.co/spaces/gathint/reachy-blocks)** — a Scratch-style block editor for choreographing Reachy with 0G chat and Whisper already wired in.

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

> **Why required:** your Space stays live after the event — judges revisit, organizers showcase winners, and you keep a portfolio piece. It also unlocks one-click install on the robot and OAuth (`robot.login()`), which only works from `*.static.hf.space`.

Create a static Space at https://huggingface.co/new-space (SDK: Static), then:

```bash
git remote set-url origin https://huggingface.co/spaces/<your-user>/<your-app>
git push -u origin main
```

Your app is now at `https://<your-user>-<your-app>.static.hf.space/` — share that URL during your demo slot. OAuth (`robot.login()`) only works from the deployed `*.static.hf.space` origin, not from localhost.

Once published, see **[Section 3](#3-showcase-your-project)** to get your Space into the official 0G hackathon collection.

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
>
> **Why required:** your Space stays live after the event — judges revisit, organizers showcase winners, and you keep a portfolio piece. It also unlocks the dashboard's one-click "Install to Robot" flow.

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

Once published, see **[Section 3](#3-showcase-your-project)** to get your Space into the official 0G hackathon collection.

### G. Deploy to the physical robot

Two ways:

**A) One-click install (recommended)** — on your Space, click **Install to Robot** → enter `http://reachy-mini.local:8000` (or the IP posted at the venue) → start it from the dashboard.

**B) Run your SDK script directly** — with your laptop on the same network, the constructor auto-detects Wireless and connects. Force it if needed:

```python
mini = ReachyMini(connection_mode="network")
```

---

## 3. Showcase your project

Once your Space is published, two short steps get it into the official **0G × Reachy Mini Hackathon** collection on Hugging Face:

- **Tag it.** Add `0g-hackathon` to your Space's `README.md` YAML frontmatter (alongside `title`, `sdk`, etc.).
- **Submit it.** Share your Space URL with organizers (Discord / event form) so it's added to the curated 0G collection.

Example frontmatter:

```yaml
tags:
  - 0g-hackathon
```

Tagged Spaces are also self-discoverable at [huggingface.co/spaces?other=0g-hackathon](https://huggingface.co/spaces?other=0g-hackathon), even if not added to the curated collection.

---

## 4. Using an AI coding agent? Bootstrap instantly

On **Claude Code, Cursor, Codex, or Copilot**, paste this to get your agent building correctly from the start:

```
I'd like to create a Reachy Mini app on the 0G stack. Start by reading
https://github.com/pollen-robotics/reachy_mini/blob/main/AGENTS.md
then the 0G AI context at
https://docs.0g.ai/ai-context
then the 0G hackathon guide at
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
- 0G AI context (load into your coding agent): [docs.0g.ai/ai-context](https://docs.0g.ai/ai-context)
- 0G compute marketplace: https://pc.0g.ai

**Have fun — build something weird and expressive. 👋**
