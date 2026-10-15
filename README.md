# Scarlett PC Agent

Lets Scarlett run commands and see screenshots on your Windows PC — only when
you ask her to. No remote-desktop software, no open ports, no always-on
service. It's a small Python script you start when you want it and stop
whenever you like.

## How it works

- Scarlett drops a command file into the **private** GitHub repo
  `ohrmich-png/scarlett-pc-bridge` (folder `cmd/`).
- The agent on your PC checks that folder every 5 seconds, runs the command,
  and writes the result back (folder `res/`).
- Everything the agent receives is **printed in its console window** — you see
  every command before/alongside it running. Nothing is hidden.

## Setup (one time, ~10 minutes)

**1. Install Python 3**
- Download from https://www.python.org/downloads/ (Windows installer).
- ⚠️ On the first installer screen, tick **"Add python.exe to PATH"**, then
  Install Now.

**2. Install the two libraries** (open Command Prompt / PowerShell and run):
```
pip install requests mss
```

**3. Get the agent files**
- Copy `pc-agent.py` and `agent.env.example` into a folder, e.g.
  `Documents\scarlett-pc\`.
- Rename `agent.env.example` → `agent.env`.

**4. Create a GitHub token** (this is the agent's key — it lives only on your PC)
- Go to https://github.com/settings/tokens?type=beta → **Generate new token**.
- Token name: `scarlett-pc-agent`. Expiration: 90 days is fine.
- **Repository access** → *Only select repositories* → choose
  `scarlett-pc-bridge`.
- **Permissions** → *Repository permissions* → **Contents** → *Read and write*.
- Click **Generate token**, copy it, and paste it into `agent.env` replacing
  `PASTE_YOUR_TOKEN_HERE`. Save the file.
- (The token can only touch that one private repo — nothing else on your
  GitHub.)

**5. Run it**
```
cd Documents\scarlett-pc
python pc-agent.py
```
You'll see `Scarlett PC agent started.` — leave the window open while you
want her connected. Tell her it's running and she'll take it from there.

## Stopping it (the kill switch)

Any one of these stops it dead:
- Press **Ctrl+C** in its window, or just **close the window**.
- Right-click its tray icon (if in tray mode) → **Quit**.
- Ask Scarlett to send the `quit` command.
- Revoke the token at https://github.com/settings/tokens (instant remote kill).
- Delete the `scarlett-pc-bridge` repo (nuclear option).

The agent does **not** start with Windows and installs nothing — when the
window is closed, it's fully off.

## Tray mode (no console window)

Prefer the agent tucked next to the clock instead of on the taskbar?

1. `pip install pystray pillow`
2. Make sure `scarlett.ico` sits next to `pc-agent.py` (it's in this repo).
3. Launch with `pythonw.exe` instead of `python.exe`:
   ```
   pythonw pc-agent.py
   ```
   No console, no taskbar button — just a Scarlett tray icon with a
   right-click → Quit menu. Point a shortcut (and the Startup-folder copy)
   at `pythonw.exe` to make it the default.
4. To force the classic console window even with pystray installed, set
   `SCARLETT_NO_TRAY=1` before launching.

## Two computers at once

Give each machine its own name in `agent.env`:

```
MACHINE=desktop   # on your PC
MACHINE=laptop    # on your laptop
```

Both agents can stay live on the same bridge — each one only runs commands
tagged for its own name and quietly ignores the rest. (Without this, every
command would run twice — once per machine.) Tell Scarlett which machine
you're on and she'll address her commands to it; a command sent while its
machine is offline simply waits in the bridge until that agent starts.

## What Scarlett can do with it

| Command | What it does |
|---|---|
| `ping` | Checks the agent is alive |
| `shell` | Runs a command (e.g. `whoami`, `dir`) and returns the output |
| `screenshot` | Captures your primary monitor and sends Scarlett the image |
| `quit` | Shuts the agent down |

She will only use these when you ask her to do something on your PC.

## Privacy notes (read once)

- The bridge repo is **private** — only you (and the token) can see it.
  Scarlett reaches it through your connected GitHub account.
- Command files are deleted after being read; result files are deleted after
  Scarlett picks them up. Screenshots are deleted from the repo right after
  she downloads them.
- Traffic goes to `api.github.com` over HTTPS. GitHub (the company)
  technically could see the bytes in transit/at rest like any cloud service —
  so don't use this for passwords or banking sessions. Everyday PC help is
  the intended use.
- Your token never leaves your PC. Never paste it in chat — if Scarlett ever
  needs it re-issued, you generate a new one yourself in the same place.

## Troubleshooting

- `ERROR: no GitHub token found` → `agent.env` is missing or still has the
  placeholder. Check the file name (not `agent.env.example`) and the token.
- `poll error: 401/403` → token is wrong, expired, or missing Contents
  read/write on the repo. Generate a new one.
- `mss not installed` warning → run `pip install mss` (screenshots need it).
- Scarlett says "timeout waiting for the PC agent" → the agent window isn't
  running, or it's stuck on an old error — restart it.
- `'python' is not recognized` → reinstall Python and tick "Add python.exe
  to PATH".
