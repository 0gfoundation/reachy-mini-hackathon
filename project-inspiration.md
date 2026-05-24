# 💡 Project Inspiration — Reachy Mini × 0G

Scoped for **1–2 days**. Every idea must use at least one of **0G Chain, Compute, or Storage**. Pick one, make it work, make it expressive. Mix and match freely.

> Reachy Mini gives you: head movement, antennas, a camera, microphones, and speakers. 0G gives you: decentralized **Compute** (LLM inference), **Storage** (persistent data/media), and **Chain** (identity, ownership, payments).

---

## 🧠 Powered by 0G Compute

- **1. Decentralized companion** — Reachy holds a conversation using an LLM served on 0G Compute instead of a centralized API. Talk to it, it talks back (with personality + antenna wiggles).
- **2. Verifiable answers** — Reachy answers questions via 0G Compute and shows it's running on verifiable/TEE-backed inference — "trustless robot." Nice for a clear, honest demo.
- **3. See-and-say** — use the camera + a vision model on 0G Compute so Reachy reacts to what it sees (objects, faces, gestures, mood) with movement and speech.

## 💾 Powered by 0G Storage

- **4. Robot with a memory** — Reachy remembers people and past conversations by persisting memory to 0G Storage, then recalls it next time it sees you. *(Decentralized agent memory — a real "MemoryVault" mini-demo.)*
- **5. Robot diary** — Reachy captures photos/audio moments from the event and writes them immutably to 0G Storage — a tamper-proof day-in-the-life archive you can play back.
- **6. Behavior library** — store community "behaviors/skills" on 0G Storage; Reachy downloads and runs them on demand. Bonus: let people contribute their own.

## ⛓️ Powered by 0G Chain

- **7. On-chain personality (iNFT)** — Reachy's character/persona lives as an NFT (ERC-7857 style) on 0G Chain; load a different NFT and its whole personality changes.
- **8. Tip-to-trick** — a smart contract on 0G Chain gates behaviors: send a micro-payment, Reachy performs (dance, joke, fortune). Live, on-chain, instant.
- **9. Voice-to-transaction** — speak a command, Reachy confirms with a nod and executes an on-chain action on 0G (mint / vote / transfer), then reports the result aloud.

## 🔀 Combo (for the ambitious)

- **10. Autonomous agent loop** — Reachy senses (Compute vision) → decides (Compute LLM) → remembers (Storage) → acts/settles (Chain). Even a thin slice of this end-to-end is a strong demo.

---

### Scoping tips
- **Build in the simulator first**, then deploy to the robot at demo time — sim ↔ hardware parity means it transfers cleanly.
- **One 0G service done well beats three done shallowly.** Judges reward 0G being *essential*, not decorative.
- **Make the robot the star.** If it could be a website, add movement, voice, or sensing until it couldn't.
- Keep the demo to a tight 2–3 minutes with one clear "wow" moment.
