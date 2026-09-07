# Reachy Mini + 0G Voice Chat (JS Starter)

A single-file web app that connects Reachy Mini to 0G decentralized compute for AI chat. Zero build step, works in any browser.

## What it does

- **Chat** with the robot via text or voice (push-to-talk)
- **AI responses** powered by 0G Compute (any OpenAI-compatible model)
- **Speech-to-text** via 0G Whisper (optional)
- **Text-to-speech** via Web Speech API (free, no key needed)
- **Expressive movement** — antennas wiggle and head tilts while speaking
- **3D simulator** included — no robot needed to develop

## Quick start

### 1. Get your 0G API keys

Go to [pc.0g.ai](https://pc.0g.ai), switch to Advanced mode, fund each provider you use, and mint one `app-sk-...` key per provider. Chat and Whisper use different provider keys.

```bash
0g-compute-cli inference get-secret --provider <PROVIDER_ADDRESS>
```

### 2. Serve locally

Linux / macOS:

```bash
python3 -m http.server 8765
```

Windows:

```powershell
py -m http.server 8765
```

### 3. Open and configure

Open `http://localhost:8765`, paste your `app-sk-...` key, and click **Start**.

The robot (or simulator) will respond to your messages with AI-generated text, spoken aloud with expressive animations.

## Publish to Hugging Face

Create a **Static** Space at [huggingface.co/new-space](https://huggingface.co/new-space), then initialize git:

```bash
git init
```

Stage your files:

```bash
git add .
```

Commit:

```bash
git commit -m "Initial Reachy Mini app"
```

Make sure the branch is named `main`:

```bash
git branch -M main
```

Set the Space remote:

```bash
git remote add origin https://huggingface.co/spaces/YOUR_USER/YOUR_APP
```

Push:

```bash
git push -u origin main
```

Your app is live at `https://YOUR_USER-YOUR_APP.static.hf.space/`.

## Customize

This is a starting point. Fork it and build your hackathon project:

- Change the system prompt in `conversationHistory` to give Reachy a different personality
- Add new motion animations in the `ROBOT ANIMATIONS` section
- Wire up the camera for vision (see the `remoteVideo` element)
- Add 0G Storage for persistent memory
- Add 0G Chain for on-chain interactions

## Files

Just one: `index.html`. That's it. No build tools, no npm, no bundler.
