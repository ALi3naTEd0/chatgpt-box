# ChatGPT Desktop Overlay for Hyprland (Wayland)

This project provides a **ChatGPT Desktop–like experience on Arch Linux** using **Hyprland + Wayland**, replicating the behavior of the official ChatGPT desktop apps on macOS/Windows:

* Global shortcut (`Alt + Space`)
* Floating, borderless window
* Centered overlay
* Persistent session
* Single-instance behavior
* No Electron, no heavy wrapper

Functionally, this behaves as an **independent desktop application**, not as a browser tab.

---

## Target Environment

* **OS**: Arch Linux (or Arch-based)
* **Display server**: Wayland
* **Window manager**: Hyprland
* **Browser runtime**: Chromium (app mode)

> If you use another WM/DE (Sway, i3, KDE, GNOME), the approach is still valid but rules and bindings must be adapted.

---

## Functional Goals

* Global shortcut: **Alt + Space**
* Floating window by default
* Manual toggle to tiling via WM (`SUPER + F`)
* No window decorations
* Fixed size
* Centered on screen
* Always on top
* Uses the **official ChatGPT web interface**
* Prevents multiple instances

---

## Step 1 — Launcher Script

Create a launcher script that **opens ChatGPT or focuses it if already running**.

### Create the script

```bash
mkdir -p ~/.local/bin
nano ~/.local/bin/chatgpt-box
```

### Script contents

```bash
#!/usr/bin/env bash

CLASS="chatgpt"

if hyprctl clients | grep -q "class: $CLASS"; then
  hyprctl dispatch focuswindow class:$CLASS
else
  chromium \
    --app=https://chat.openai.com \
    --class=$CLASS \
    --user-data-dir=$HOME/.config/chatgpt-app \
    --window-size=480,640 &
fi
```

### Make it executable

```bash
chmod +x ~/.local/bin/chatgpt-box
```

### What this script does well

* Prevents duplicate windows
* Keeps a persistent login session
* Runs independently from your default browser
* Mimics a real desktop application lifecycle

---

## Step 2 — Hyprland Window Rules

Edit your Hyprland configuration:

```bash
nano ~/.config/hypr/hyprland.conf
```

Add the following rules:

```conf
windowrulev2 = float,class:^(chatgpt)$
windowrulev2 = size 480 640,class:^(chatgpt)$
windowrulev2 = center,class:^(chatgpt)$
windowrulev2 = noborder,class:^(chatgpt)$
windowrulev2 = stayfocused,class:^(chatgpt)$
windowrulev2 = pin,class:^(chatgpt)$
```

### Result

* Always floating by default
* Clean, borderless UI
* Overlay-style behavior
* Focus remains on the ChatGPT window

The user can still manually tile it using Hyprland’s `togglefloating` binding.

---

## Step 3 — Global Shortcut (Alt + Space)

In the same `hyprland.conf` file:

```conf
bind = ALT, Space, exec, ~/.local/bin/chatgpt-box
```

Reload Hyprland:

```bash
hyprctl reload
```

---

## Step 4 — Close with Escape (Optional but Recommended)

To match Spotlight / ChatGPT Desktop behavior:

```conf
bind = , Escape, exec, hyprctl dispatch closewindow class:chatgpt
```

---

## Final User Experience

* **Alt + Space** → ChatGPT overlay appears
* Immediate typing focus
* **Escape** → window closes
* No browser chrome
* No Electron
* No interference with normal workflow

In practice, this feels **equal to or better than the official ChatGPT desktop app**.

---

## Is This a “Real” Desktop App?

Technically:

* It is a **web app encapsulated in a dedicated WebView**
* Uses an isolated Chromium profile
* Does **not** rely on your default browser
* Does **not** open as a tab
* Has its own lifecycle and session

This is the same architectural model used by:

* Slack
* Discord
* Notion
* VS Code
* Spotify (Linux)

---

## Optional Improvements

Possible next steps:

* True toggle behavior (show/hide instead of close)
* Entry in application menu (`.desktop` file)
* Custom icon
* Dark mode enforcement
* Scrollbar removal
* Entry animation
* Migration to WebKitGTK (lighter runtime)
* Tauri-based native wrapper
* Replace ChatGPT with **Ollama (local LLM)**

---

## License

MIT (or choose your preferred license)

---
