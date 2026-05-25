# 📋 0G x Reachy Starter Snippets

Copy-paste helpers that hit 0G Compute correctly the first time. Pair these with the [hackathon guide](hackathon-guide.md) and the API key setup (Section 1).

> All snippets assume you've completed **[Section 1 of the hackathon guide](hackathon-guide.md#1-get-your-0g-api-keys-do-this-before-the-event)**: funded provider sub-accounts and minted per-provider `app-sk-…` keys.

---

## JS / browser (Path A)

### Chat completion

OpenAI-compatible, works for `deepseek/deepseek-chat-v3-0324`, `zai-org/GLM-5-FP8`, `qwen3.6-plus`, and any other 0G chat model.

```js
async function askAI(prompt, { signal } = {}) {
  const res = await fetch(
    'https://compute-network-1.integratenetwork.work/v1/proxy/chat/completions',
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${OG_CHAT_KEY}`,   // app-sk-... for your chat provider
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        model: 'zai-org/GLM-5-FP8',
        messages: [{ role: 'user', content: prompt }],
        max_tokens: 256,
      }),
      signal,
    }
  );
  if (!res.ok) throw new Error(`Chat ${res.status}: ${(await res.text()).slice(0, 200)}`);
  const data = await res.json();
  return (data.choices?.[0]?.message?.content || '').trim();
}
```

### Whisper STT (speech-to-text)

**Important:** 0G's `openai/whisper-large-v3` wrapper rejects raw browser webm/opus blobs with `"Invalid or unsupported audio file"`. Decode + re-encode to WAV (PCM) first using the Web Audio API.

```js
// 1. Capture audio (from robot mic via WebRTC, or laptop mic):
//    const recorder = new MediaRecorder(audioStream, { mimeType: 'audio/webm;codecs=opus' });
//    Collect chunks into a Blob: `rawBlob` (type: audio/webm;codecs=opus)

async function webmBlobToWav(blob) {
  const buf = await blob.arrayBuffer();
  const ctx = new (window.AudioContext || window.webkitAudioContext)();
  const audio = await ctx.decodeAudioData(buf);
  try { ctx.close(); } catch (_) {}
  const ch0 = audio.getChannelData(0);
  const len = ch0.length;
  const sampleRate = audio.sampleRate;
  const out = new ArrayBuffer(44 + len * 2);
  const v = new DataView(out);
  const ws = (off, s) => { for (let i = 0; i < s.length; i++) v.setUint8(off + i, s.charCodeAt(i)); };
  ws(0, 'RIFF'); v.setUint32(4, 36 + len * 2, true); ws(8, 'WAVE');
  ws(12, 'fmt '); v.setUint32(16, 16, true);
  v.setUint16(20, 1, true); v.setUint16(22, 1, true);
  v.setUint32(24, sampleRate, true); v.setUint32(28, sampleRate * 2, true);
  v.setUint16(32, 2, true); v.setUint16(34, 16, true);
  ws(36, 'data'); v.setUint32(40, len * 2, true);
  let off = 44;
  for (let i = 0; i < len; i++) {
    const s = Math.max(-1, Math.min(1, ch0[i]));
    v.setInt16(off, s < 0 ? s * 0x8000 : s * 0x7FFF, true);
    off += 2;
  }
  return new Blob([out], { type: 'audio/wav' });
}

async function transcribe(rawWebmBlob, { signal } = {}) {
  const wav = await webmBlobToWav(rawWebmBlob);
  const form = new FormData();
  form.append('file', wav, 'audio.wav');
  form.append('model', 'openai/whisper-large-v3');
  form.append('response_format', 'json');
  const res = await fetch(
    'https://compute-network-16.integratenetwork.work/v1/proxy/audio/transcriptions',
    {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${OG_STT_KEY}` },   // app-sk-... for the whisper provider
      body: form,
      signal,
    }
  );
  if (!res.ok) throw new Error(`STT ${res.status}: ${(await res.text()).slice(0, 200)}`);
  return (await res.json()).text || '';
}
```

### Using the robot's mic (not the laptop's)

When a Reachy session is live, capture audio from the WebRTC incoming track instead of `getUserMedia`:

```js
const remoteVideo = document.getElementById('remoteVideo');   // wherever attachVideo was called
const robotAudio = remoteVideo?.srcObject?.getAudioTracks?.() || [];
const audioStream = robotAudio.length
  ? new MediaStream(robotAudio)
  : await navigator.mediaDevices.getUserMedia({ audio: true });  // sim-only fallback

const recorder = new MediaRecorder(audioStream, { mimeType: 'audio/webm;codecs=opus' });
// ... capture chunks, then transcribe(blob) above
```

**Never call `robot.setMicMuted(false)`** — it unmutes the outgoing laptop-mic track and the robot's speaker echoes whatever your laptop hears.

---

## Python (Path B)

### Chat completion

The `openai` SDK works as-is against 0G's OpenAI-compatible proxy:

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://compute-network-1.integratenetwork.work/v1/proxy",
    api_key=OG_CHAT_KEY,    # app-sk-... for your chat provider
)

resp = client.chat.completions.create(
    model="zai-org/GLM-5-FP8",
    messages=[{"role": "user", "content": "Hi Reachy"}],
    max_tokens=256,
)
print(resp.choices[0].message.content)
```

### Whisper STT

Same SDK, different base URL (each 0G provider has its own host):

```python
from openai import OpenAI

stt_client = OpenAI(
    base_url="https://compute-network-16.integratenetwork.work/v1/proxy",
    api_key=OG_STT_KEY,     # app-sk-... for the whisper provider (DIFFERENT from chat key)
)

with open("recording.wav", "rb") as f:
    result = stt_client.audio.transcriptions.create(
        model="openai/whisper-large-v3",
        file=f,
        response_format="json",
    )
print(result.text)
```

Notes:
- WAV / mp3 / m4a / aiff all work.
- Don't reuse the chat client — `api_key` and `base_url` are both wrong for the whisper provider.

---

## Reference implementation

A full, shipped application that uses every snippet above (plus a 3D sim, a Scratch-style block editor, share/import, TTS routed through the robot speaker, AI-in-use panel, missing-key warnings) is **[`gathint/reachy-blocks`](https://huggingface.co/spaces/gathint/reachy-blocks)** ([source](https://huggingface.co/spaces/gathint/reachy-blocks/tree/main)). Fork it to get a working baseline; replace the block UI with your own app if you want a different shape.

Key files to study in that source:
- `index.html` → `askAI()`, `listenForSpeech()`, `say()`, `_webmBlobToWav()`, key/provider settings
- `sim.js` → the in-browser 3D simulator (Three.js + URDF loader)

---

## Common 400/401 you'll hit

| Error response | Cause | Fix |
|---|---|---|
| `Provider proxy: ... missing or invalid Authorization header, must be Bearer app-sk-<base64(rawMessage:signature)>` | Wrong key for this host (e.g. chat key sent to whisper endpoint) | Use the per-provider key matching the host |
| `Provider proxy: ... insufficient balance` (paraphrased) | Sub-account on that provider isn't funded | Deposit at least 1 0G to that provider on pc.0g.ai |
| `Invalid or unsupported audio file` (from Whisper) | Browser sent webm/opus directly | Convert to WAV with `webmBlobToWav` above before POST |
| CORS preflight fails | Hitting a path that doesn't exist (e.g. base URL missing `/proxy`) | Check the full URL ends with `/v1/proxy` then the endpoint |
| `app-sk-` token rejected after working earlier | `get-secret` tokens have a TTL | Re-mint with `0g-compute-cli inference get-secret --provider <addr>` |
