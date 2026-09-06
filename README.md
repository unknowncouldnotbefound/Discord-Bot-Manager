# Discord Bot Manager

A desktop tool for running and monitoring multiple Discord bots (or any Python
scripts) from one window — with live green/red status, auto-restart on crash,
and launch-on-Windows-startup.

## Setup

1. Put `discord_bot_manager.py` anywhere you want (e.g. `C:\Tools\BotManager`).
   It'll create a `data` folder next to itself for config + logs — everything
   is local, nothing gets uploaded anywhere.

2. Install the one required dependency:
   ```
   py -m pip install customtkinter
   ```
   (Using `py -m pip` instead of plain `pip` since that's what worked around
   the PATH issues on your machine before.)

3. Optional, only if you want the true "minimize to system tray" behavior
   instead of just minimizing to the taskbar:
   ```
   py -m pip install pystray pillow
   ```

4. Run it:
   ```
   py discord_bot_manager.py
   ```

## Using it

- **+ Add Bot** → point it at the folder your bot lives in. It'll scan for
  `.py` files and guess the right one (looks for `main.py`, `bot.py`,
  `run.py`, `app.py`, `start.py` first). It also auto-detects a `venv`/`.venv`
  folder inside the bot's directory and uses that Python interpreter if one
  exists — otherwise it uses whatever Python is running the manager. You can
  always override the interpreter manually if a bot needs a specific one.

- Each bot gets a **status dot**: green = running, red = stopped/crashed,
  yellow = starting or waiting to auto-restart, gray = disabled.

- **Start / Stop / Restart** per bot, or **Start All / Stop All** from the
  top bar.

- **Log** button opens a live-updating log viewer for that bot (its console
  output is captured to `data/logs/<id>.log`). Logs auto-trim once they pass
  the size limit you set in Settings, so they won't grow forever.

- **Edit** lets you change a bot's folder, entry file, interpreter, args, or
  toggle auto-start / auto-restart / enabled without deleting and re-adding it.

## Settings

- **Launch this manager when Windows starts** — adds a small `.bat` shortcut
  to your Startup folder (`shell:startup`). No admin rights needed, no
  registry edits. Unchecking it removes the same file.
- **Auto-start bots on launch** — when the manager itself opens, it starts
  every bot that has "auto-start" checked.
- **Minimize to tray** — closing the window hides it to the system tray
  instead of quitting (requires `pystray` + `pillow`; falls back to a normal
  minimize if those aren't installed).
- **Auto-restart crashed bots** — if a bot's process ends unexpectedly (not
  because you clicked Stop), it'll be relaunched after a delay, up to a max
  number of attempts, both configurable.
- **Check interval** — how often (ms) the manager polls process status. 2000ms
  is the default and is basically free CPU-wise; you can raise it if you're
  running this on something very low-powered.
- **Show console windows for bots** — off by default so bots run fully
  hidden in the background; turn on if you want to see a bot's console pop
  up for debugging.
- **Stop all bots on exit** — if off (default), fully closing the manager
  leaves bots running independently; if on, closing the manager stops them
  too.

## Notes on performance

- Bots run as normal hidden subprocesses (`CREATE_NO_WINDOW`) — the manager
  isn't doing anything heavier than checking `process.poll()` every couple
  seconds per bot, so it should sit at basically 0% CPU when idle.
  Console output is piped straight to a log file by the OS, not read/copied
  in a loop, so it doesn't add overhead even for chatty bots.

## Starting it hidden at boot

The Startup shortcut launches it with `--minimized`, which skips showing the
main window and goes straight to the tray icon (or a minimized taskbar entry
if you didn't install the tray dependencies) and auto-starts your bots.
