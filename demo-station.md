# 🎛️ Reachy Mini — Demo Station Operator Card

**For the person running the robot during demos.** One robot, many teams. Your job: keep the queue moving and the robot un-stuck.

---

## Quick facts (write the real values in the blanks)

| Thing | Value |
|---|---|
| Dashboard URL | `http://reachy-mini.local:8000` |
| Robot IP (mDNS fallback) | `http://__________:8000` |
| REST API base | `http://reachy-mini.local:8000/api` |
| Live API explorer (Swagger) | `http://reachy-mini.local:8000/docs` |
| SSH into the robot | `ssh root@reachy-mini.local` · password: `root` |
| Battery | Wireless = onboard battery. **Keep it on the charger between slots.** |

> **Golden rule:** only **one app runs at a time**. Every demo = install/start → run → **stop** → next. Never let two teams connect at once.

---

## The demo queue (repeat per team)

1. **Confirm the robot is idle.** Dashboard shows no app running (or current-app-status is clear).
2. **Load the team's app** — either:
   - They click **Install to Robot** on their HF Space and enter the dashboard URL, **or**
   - You install it from the dashboard app list.
3. **Start it** from the dashboard.
4. **Team demos** (give them a fixed time — e.g. 3 min).
5. **Stop the app** (see below). The daemon sends a graceful shutdown and **returns the robot to its default position automatically** — so it self-resets between teams.
6. **Next team.**

---

## Stopping a running app

- **Normal:** hit **Stop** in the dashboard. The daemon sends SIGINT, the app shuts down cleanly, and the robot returns to default pose.
- **If the dashboard button doesn't respond:** use the REST API via the Swagger page (`/docs`) — find the apps **stop** endpoint and execute it. (You can also `POST` to the apps stop route directly with curl.)

---

## 🚑 Reset-if-stuck ladder

Work top to bottom — stop as soon as it's fixed.

1. **App won't stop / robot frozen mid-motion** → Dashboard **Stop**. Wait ~5s.
2. **Still stuck** → **Restart the daemon** from the dashboard (Settings), or via the daemon restart endpoint in `/docs`. This kills any running app and re-homes the robot.
3. **Dashboard unresponsive** → SSH in (`ssh root@reachy-mini.local`, pw `root`) and restart the service:
   ```bash
   sudo systemctl restart reachy-mini-daemon
   ```
4. **Total lockup** → **Power-cycle the robot.** Turn it off, wait ~10s, turn on. Daemon auto-starts on boot; allow ~1 min before it's reachable again.
5. **Comes back on its own hotspot (`reachy-mini-ap`) instead of venue Wi-Fi** → it lost the network. Reconnect it to venue Wi-Fi via the Reachy Mini Control app (or the Wi-Fi Reset Guide), then resume.

---

## Common failures → fast fixes

| Symptom | Fix |
|---|---|
| Team's laptop can't reach the robot | Both must be on the **same network**. Have them ping the robot IP. Venue Wi-Fi may block client-to-client — use the dedicated router/SSID. |
| `reachy-mini.local` won't resolve | Use the **IP** instead (top of card). Common on managed networks. |
| macOS browser/terminal can't see robot | System Settings → Privacy & Security → **Local Network** → allow the app. |
| Robot powered but not moving | Check an app is actually **started** (not just installed). Check it's awake, not asleep. Restart daemon if needed. |
| App installed but errors on start | Likely a dependency/packaging issue in their app — not your problem to debug live. **Skip, move on, come back.** |
| Audio not working | Confirm the robot's mics/speakers; some apps need audio enabled. Don't burn queue time on this. |

---

## Don't-panic reminders

- The robot **self-homes** after every app stops — you rarely need a manual reset.
- It's **beta** — occasional crashes are normal. The daemon restart (step 2–3) fixes almost everything.
- Keep it **on the charger** between slots so you never run out of battery mid-demo.
- When in doubt: **Stop → Restart daemon → next team.**
