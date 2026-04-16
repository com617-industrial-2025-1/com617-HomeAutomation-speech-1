# MDAR Smart Home System — Complete Setup Guide

> **From zero to a fully working voice-controlled Velbus smart home in ~4 hours.**
> Built for COM617 Industrial Consulting Project by Group 7 — Southampton Solent University, in partnership with MDAR.

---

## Read this first

This guide is written for someone who has **never touched Home Assistant, Velbus, or Node-RED before**. Every step is spelled out. If you can follow a recipe, you can follow this.

**What you'll have at the end:**

- A Raspberry Pi running Home Assistant as a smart home hub
- 8 Velbus devices (lights, relays, motion sensor, glass panel) fully controllable
- A voice assistant you can talk to: *"Turn on the living room light"* → light turns on + spoken confirmation
- 5 automatic Node-RED automations (motion-triggered lights, night mode, scheduled reports, etc.)
- MQTT messages being published every 5 minutes with the state of every device — so a landlord could monitor the property remotely

**Time needed:** ~4 hours across the steps. Feel free to take breaks between Parts.

**Difficulty:** Beginner-friendly if you follow in order. You'll copy-paste a lot and click through UI.

**If you get stuck:** every Part has a note linking to the troubleshooting table in Part 19. Don't skip ahead — the order matters.

---

## Contents

- [Part 0 — Before You Start](#part-0--before-you-start)
- [Part 1 — Flash the Pi with Home Assistant OS](#part-1--flash-the-pi-with-home-assistant-os)
- [Part 2 — Connect the Pi to the Internet (Windows ICS)](#part-2--connect-the-pi-to-the-internet-windows-ics)
- [Part 3 — Home Assistant Onboarding](#part-3--home-assistant-onboarding)
- [Part 4 — Install the 4 Core Add-ons](#part-4--install-the-4-core-add-ons)
- [Part 5 — Fix Node-RED SSL Before First Start](#part-5--fix-node-red-ssl-before-first-start)
- [Part 6 — Generate the Long-Lived Access Token](#part-6--generate-the-long-lived-access-token)
- [Part 7 — Program the Velbus Hardware (VelbusLink)](#part-7--program-the-velbus-hardware-velbuslink)
- [Part 8 — Add the Velbus Integration in Home Assistant](#part-8--add-the-velbus-integration-in-home-assistant)
- [Part 9 — Name & Organise Entities](#part-9--name--organise-entities)
- [Part 10 — Test Direct Control](#part-10--test-direct-control)
- [Part 11 — Set Up the Voice Pipeline (Whisper + Piper)](#part-11--set-up-the-voice-pipeline-whisper--piper)
- [Part 12 — Expose Entities to Assist](#part-12--expose-entities-to-assist)
- [Part 13 — Fix the Browser Microphone (Chrome Flag)](#part-13--fix-the-browser-microphone-chrome-flag)
- [Part 14 — Hello World Voice Test](#part-14--hello-world-voice-test)
- [Part 15 — Set Up the AI Conversation Agent (Choose Your Path)](#part-15--set-up-the-ai-conversation-agent-choose-your-path)
  - [Path A — OpenRouter (cloud)](#path-a--openrouter-cloud--recommended-for-ease)
  - [Path B — Ollama (local)](#path-b--ollama-local--fully-private)
  - [Path C — Both](#path-c--both-agents-configured)
- [Part 16 — Import the Node-RED Flows](#part-16--import-the-node-red-flows)
- [Part 17 — Test Each Node-RED Flow](#part-17--test-each-node-red-flow)
- [Part 18 — Verify MQTT End-to-End](#part-18--verify-mqtt-end-to-end)
- [Part 19 — Troubleshooting Table](#part-19--troubleshooting-table)
- [Part 20 — What's Next](#part-20--whats-next)
- [Appendix A — Full Entity Name Table](#appendix-a--full-entity-name-table)
- [Appendix B — MDAR System Prompt (full text)](#appendix-b--mdar-system-prompt-full-text)
- [Appendix C — Cost Estimate](#appendix-c--cost-estimate)

---

## Part 0 — Before You Start

### Hardware shopping list

| Item | Quantity | Notes | Approx. cost (GBP) |
|---|---|---|---|
| Raspberry Pi 5 (8GB RAM) | 1 | 4GB works but 8GB gives headroom | £80 |
| Official Raspberry Pi 5 USB-C power supply (27W) | 1 | Don't skimp — underpowered PSUs cause random reboots | £12 |
| MicroSD card, 32GB, A2 class | 1 | SanDisk Extreme or Samsung Pro Endurance | £12 |
| Ethernet cable, any length | 1 | Cat 5e or better | £5 |
| Velbus starter kit (from MDAR) + 8 modules — see Appendix A | Supplied | | — |
| Windows laptop | 1 | For flashing SD card + running VelbusLink. See note below for Mac/Linux users | — |

> **Mac / Linux users:** VelbusLink is Windows-only. You'll need to either (a) borrow a Windows machine for Part 7, (b) run Windows 10/11 in a VM with USB passthrough (Parallels/VMware on Mac, VirtualBox on Linux), or (c) use a cheap second-hand Windows laptop. The rest of the guide works from any OS.

### Software to install on your laptop (before we start)

1. **Raspberry Pi Imager** — https://www.raspberrypi.com/software/ (free)
2. **VelbusLink** — download from https://www.velbus.eu — Windows only, free
3. **Google Chrome** — we use a Chrome-specific flag later. Firefox/Edge won't work for this project
4. **MQTT Explorer** — http://mqtt-explorer.com/ (free) — for Part 18 only

### Accounts you'll need

- **OpenRouter account** (if you choose Path A in Part 15) — free signup, but requires $5 minimum credit. Actual usage for personal voice control is very low.
- **Ollama** (if you choose Path B) — no account needed. Free.

### Getting help when stuck

- **Skim Part 19 first** — 90% of problems are covered there
- **Home Assistant community** — https://community.home-assistant.io/
- **Velbus support forum** — https://www.velbus.eu/support/
- **GitHub Issues on this repo** — open an issue if you find a problem

---

## Part 1 — Flash the Pi with Home Assistant OS

**Goal:** put Home Assistant OS onto the SD card so the Pi will boot into it.

> This erases everything on the SD card. Use a fresh one.

1. Insert the microSD card into your laptop (use a USB adapter if needed).
2. Open **Raspberry Pi Imager** (install it from https://www.raspberrypi.com/software/ if you haven't).
3. Click **"Choose Device"** → select **"Raspberry Pi 5"**.
4. Click **"Choose OS"**. Home Assistant OS is **not in the top list**. Navigate:
   - `Other specific-purpose OS`
   - → `Home assistants`
   - → `Home Assistant`
   - → `Home Assistant OS (RPi 5)`
5. Click **"Choose Storage"** → pick your SD card (double-check you picked the right drive).
6. Click **"Next"** → if prompted about OS customisation, click **"No"** / **"Skip"**. Then **"Yes"** to confirm the write.
7. Wait ~5 minutes. The Imager downloads HAOS and writes + verifies it.
8. When it says "Write Successful" you can safely remove the SD card.

> If the Imager fails partway through → try a different SD card. Cheap cards are the #1 cause of flash failures.

---

## Part 2 — Connect the Pi to the Internet (Windows ICS)

**Goal:** give the Pi internet access via your laptop's WiFi so it can download updates and add-ons.

> If you have a spare ethernet port on your home router, you can just plug the Pi in and skip this whole Part. Your home router will assign it an IP. Continue to Part 3.

### Option A — Internet Connection Sharing (most common for this project)

1. **Insert the flashed SD card** into the Pi.
2. **Connect an ethernet cable** between the Pi and your laptop's ethernet port. Don't power the Pi yet.
3. **Plug in the Pi's power supply** — the Pi will boot. **Do NOT touch it for the next 20 minutes.** The first boot downloads and sets up HAOS, and interrupting it corrupts the SD card.
4. While the Pi boots, configure ICS on your laptop:
   - Press **Windows key + R** → type `ncpa.cpl` → Enter. Network Connections opens.
   - **Right-click your WiFi adapter** (the one connected to the internet) → **Properties**.
   - Switch to the **"Sharing"** tab.
   - Tick **"Allow other network users to connect through this computer's Internet connection"**.
   - **"Home networking connection"** dropdown → select the Ethernet adapter connected to the Pi.
   - Click **OK**. You may see a popup about the LAN IP changing to 192.168.137.1 — click Yes.
5. **Wait the full 20 minutes** since you powered on the Pi.
6. Open **Chrome** on your laptop → go to **http://homeassistant.local:8123**
   - If it loads → perfect, continue to Part 3.
   - If it doesn't → see troubleshooting below.

> **`homeassistant.local` doesn't resolve:**
> - Wait longer. First boot can take 25 minutes on slower SD cards.
> - Install **Bonjour Print Services for Windows** (Apple's mDNS helper) — this fixes `.local` resolution on Windows.
> - Fallback: find the Pi's IP another way. Open Command Prompt → `arp -a` → look for a device starting with `b8:27:eb` or `dc:a6:32` (Pi MAC prefixes). Use that IP instead of `homeassistant.local`.

---

## Part 3 — Home Assistant Onboarding

**Goal:** create your admin account and get to the empty dashboard.

1. You should see the **"Preparing Home Assistant"** screen. Wait a few more minutes if so.
2. When the setup wizard appears:
   - **Create User:** pick a name + username + strong password. **Save these somewhere safe.** Losing the password means re-flashing the SD card.
   - **Home Name:** `MDAR Lab` (or whatever — doesn't matter functionally).
   - **Location:** click "Detect" then adjust if needed. Set your country + currency + unit system.
   - **Data sharing:** your choice. Everything can be off.
3. You'll land on a "Found devices on your network" page. Click **"Finish"** — don't add anything yet. We'll do this manually later.
4. You're now on the empty dashboard. Bookmark **http://homeassistant.local:8123** — you'll visit it a lot.

---

## Part 4 — Install the 4 Core Add-ons

**Goal:** install the services we need: MQTT broker, Node-RED, Whisper (STT) and Piper (TTS).

> In Home Assistant 2025+, **"Add-ons" was renamed to "Apps"**. Old tutorials say Add-ons. It's the same thing.

1. Click **Settings** (bottom-left sidebar) → **Apps**.
2. Click **"Apps Store"** button (top-right).

### 4.1 — Install Mosquitto MQTT Broker

3. Search **"Mosquitto"** → click **"Mosquitto broker"** (by Home Assistant Community Add-ons).
4. Click **"Install"**. Wait ~1 min.
5. After install:
   - Turn ON **"Start on boot"**
   - Turn ON **"Watchdog"**
   - Turn ON **"Auto-update"**
   - Click **"Start"**
6. Wait 30s, click the **"Log"** tab → confirm you see `mosquitto version 2.x running`. No errors.

### 4.2 — Install Node-RED

7. Back to Apps Store → search **"Node-RED"** → install.
8. **DO NOT START IT YET.** We'll fix SSL first in Part 5 or it will fail.

### 4.3 — Install Wyoming Whisper (Speech-to-Text)

9. Back to Apps Store → search **"Whisper"** → install the Wyoming Protocol version.
10. Install. Wait ~2 min (it's bigger).
11. Click **"Configuration"** tab:
    - Model: `base-en` (best balance of speed vs accuracy on Pi 5)
    - Language: `en`
    - Beam size: `1`
12. Save. Then Info tab → Start on boot + Watchdog → **Start**.
13. Log tab → confirm `Ready` appears. First start may take 2 min while it downloads the model.

### 4.4 — Install Wyoming Piper (Text-to-Speech)

14. Apps Store → search **"Piper"** → install.
15. Configuration tab:
    - Voice: `en_GB-alan-medium` (British English, good natural-sounding voice)
16. Save → Start on boot + Watchdog → **Start**.

### Final check

Go to **Settings → Apps** — you should see all 4 add-ons listed with green "running" indicators. Node-RED will still be stopped; we'll handle it next.

> If any add-on fails to install → check the Log tab of that add-on. Restart the Pi (Settings → System → Restart Host) and retry.

---

## Part 5 — Fix Node-RED SSL Before First Start

**Goal:** disable SSL on Node-RED so it actually starts. Without this fix, Node-RED crashes on boot.

> **Why?** Node-RED defaults to `ssl: true` but expects certificate files that don't exist on a fresh system. Since we're on a trusted local network, disabling SSL is fine for development. For production, set up HTTPS via a reverse proxy — documented as future work in the PID.

1. Settings → Apps → **Node-RED** → **Configuration** tab.
2. Find the line `ssl: true`. Change it to:

   ```yaml
   ssl: false
   ```

3. Click **"Save"**.
4. Back on the **Info** tab:
   - Start on boot
   - Watchdog
   - Click **"Start"**
5. Wait ~30s. Check the **Log** tab — you should see `Welcome to Node-RED v3.x` and no errors.
6. You now see a **"Open Web UI"** button. Click it — Node-RED opens in a new tab. Leave it open.

---

## Part 6 — Generate the Long-Lived Access Token

**Goal:** create a permanent token that Node-RED will use to talk to Home Assistant.

> **Why?** Node-RED is technically a separate program and needs to authenticate to Home Assistant like any external client would. A Long-Lived Access Token (LLAT) is a password-equivalent that never expires.

1. Click your **profile circle** (bottom-left, with your initial).
2. Click the **"Security"** tab at the top.
3. Scroll to the very bottom to **"Long-lived access tokens"**.
4. Click **"Create token"**.
5. Name it: `Node-RED`
6. Click **OK**.
7. **COPY THE TOKEN RIGHT NOW.** Paste it into a temporary Notepad file — you **cannot view it again**. If you lose it, you have to create another one.

   Example of what it looks like (yours will be different):
   ```
   eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyZGFh...
   ```

8. Keep that Notepad open. We'll use it in Part 16.

---

## Part 7 — Program the Velbus Hardware (VelbusLink)

**Goal:** tell the Velbus modules what to do. Without this step, Home Assistant sees the modules but can't control anything.

> **Why?** Velbus is designed to work **without** a host computer. The logic (button 1 controls light 1, etc.) lives inside each module's firmware. VelbusLink is the tool to program that firmware. Until you do this, the modules are inert.

> **Windows only.** See Part 0 note if you're on Mac/Linux.

### 7.1 — Disconnect VMB1USB from the Pi

1. Unplug the USB cable going from the Pi to the VMB1USB module. We're going to temporarily connect it to your Windows laptop instead.

### 7.2 — Connect VMB1USB to Windows

2. Plug the VMB1USB into your **Windows laptop's** USB port.
3. Windows will auto-install a driver. Wait 30s.
4. Open **Device Manager** (Windows key + X → Device Manager).
5. Expand **"Ports (COM & LPT)"** — note the COM port number. Write this down.

### 7.3 — Open VelbusLink

6. Install VelbusLink from https://www.velbus.eu/ if you haven't already.
7. Open it. Create a new project: **File → New Project**. Name it `MDAR_Lab.vlp` and save somewhere.
8. Top-right corner: click the **"Connect"** icon (looks like a plug).
9. Select the COM port from step 5. Click OK.
10. Wait ~5s. Bottom status bar should change from `Working offline` to `Connected → Bus Active / Receive Ready`.

### 7.4 — Scan the bus

11. Click **Edit → Scan**. VelbusLink will discover all modules on the bus.
12. Wait for the scan to complete (~30s). You should see 8 modules appear in the project tree on the left.

### 7.5 — Name the channels on the VMB4DC (dimmer)

13. In the project tree, click the **VMB4DC** module (bus address 02).
14. In the right panel, go to the **"Channel names"** tab.
15. Fill in (VelbusLink has a short character limit — accept these truncated names, we'll fix them in Home Assistant later):

    | Channel | Name |
    |---|---|
    | CH1 | `living room ligh` |
    | CH2 | `kitchen light` |
    | CH3 | `toilet light` |
    | CH4 | `bedroom light` |

### 7.6 — Name the channels on the VMB4RYLD (relay)

16. Click the **VMB4RYLD** module (bus address 07). Channel names tab:

    | Channel | Name |
    |---|---|
    | CH1 | `living room spot` |
    | CH2 | `kitchen spot` |
    | CH3 | `bedroom spot` |
    | CH4 | `toilet spot` |

### 7.7 — Program the button-to-output actions

This maps the physical VMBGPOD-2 glass-panel buttons to the lights/spots.

17. In the project tree, click the **VMBGPOD-2** module (bus address 03).
18. Click the **"Actions"** tab.
19. Click **"Add new action"** — a configuration dialog opens.
20. Configure action #1:
    - **Source module:** VMBGPOD-2
    - **Source channel:** Push button 1
    - **Target module:** VMB4DC
    - **Target channel:** CH1 (living room light)
    - **Action:** `Push button: Dim at long press, Toggle at short press (0202)`
21. Click Save.
22. Repeat for actions 2, 3, 4 — mapping buttons 2/3/4 to VMB4DC CH2/CH3/CH4.
23. For actions 5, 6, 7, 8 — use the simpler **"Toggle (0103)"** action:
    - Button 5 → VMB4RYLD CH1 (living room spot)
    - Button 6 → VMB4RYLD CH2 (kitchen spot)
    - Button 7 → VMB4RYLD CH3 (bedroom spot)
    - Button 8 → VMB4RYLD CH4 (toilet spot)

### 7.8 — Write configuration to the modules

24. Click **Edit → Synchronize → Write to modules**. This uploads your configuration to module firmware.
25. Wait ~30s. You'll see a "Write successful" confirmation.

### 7.9 — Unplug and reconnect to Pi

26. **Disconnect** from VelbusLink: click the plug icon again → Disconnect.
27. **Close VelbusLink.**
28. **Unplug VMB1USB** from your Windows laptop.
29. **Plug VMB1USB back into the Pi's USB port.**

> If VelbusLink says "Buffer full" → stop the scan, wait 30 seconds, try again. See Risk #16 in Part 19.

> If modules don't appear after scan → check the COM port. Try clicking Connect again.

---

## Part 8 — Add the Velbus Integration in Home Assistant

**Goal:** tell Home Assistant about the Velbus hardware we just programmed.

1. In Home Assistant: **Settings → Devices & Services**.
2. Click **"+ Add Integration"** (bottom-right).
3. Search **"Velbus"** → select it.
4. A dropdown appears. Select `/dev/ttyACM0 - VMB1USB Velbus USB interface - Velleman Projects`.
5. Click **Submit**.
6. Wait ~10s. You should see **"Success! Created configuration for Velbus"** and **"Found X devices"**.
7. Click **"Finish"**.
8. Back on Devices & Services, you'll see **"Velbus"** as a card. Click it.
9. You should see all 8 modules listed. Click any of them → **Controls** section → you should see named entities (e.g. "Living Room Light" as a dimmer control).

> **"This device has no entities"** → this means Part 7 (VelbusLink programming) didn't save correctly. Go back, redo the synchronize step, then delete this Velbus integration and re-add. See Risk #5 in Part 19.

> **Only some modules discovered** → wait 2 minutes and refresh. Velbus scanning is not instant.

---

## Part 9 — Name & Organise Entities

**Goal:** give every device a clean friendly name and group them by room.

### 9.1 — Create the 4 Areas

1. **Settings → Areas, labels & zones → Areas** tab.
2. Click **"+ Create area"** → create four:
   - Living Room
   - Kitchen
   - Bedroom
   - Toilet

### 9.2 — Rename and assign entities

3. **Settings → Devices & Services → Velbus** → click the first dimmer device.
4. For each entity:
   - Click the entity
   - Click the gear icon (top-right)
   - **"Name"** field → set the friendly name (see Appendix A)
   - **"Area"** field → pick the correct area
   - Save

5. Repeat for every entity. Use the table in **Appendix A** as the reference.
6. When done, go to **Settings → Areas** → check each area has the right entities listed.

---

## Part 10 — Test Direct Control

**Goal:** confirm Home Assistant can actually toggle the physical hardware before moving on.

### 10.1 — Dashboard test

1. Go to the main dashboard (click the home icon).
2. Click each light card's toggle. The physical light/spot should respond within a second.

### 10.2 — Physical button test

3. Press **Button 1** on the VMBGPOD-2 glass panel → the **Living Room Light** should toggle (short press) or dim (long press).
4. Repeat for buttons 2–8. Confirm each controls the right output per your VelbusLink config.
5. Also check the Home Assistant UI — when you press a physical button, the UI should update to reflect the new state within 2 seconds.

> If a button does nothing → go back to Part 7.7. Did you click "Write to modules"?

> If the UI doesn't update when you press a button → restart the Velbus integration (Settings → Devices → Velbus → three-dot menu → Reload).

---

## Part 11 — Set Up the Voice Pipeline (Whisper + Piper)

**Goal:** connect the speech-to-text (Whisper) and text-to-speech (Piper) engines to Home Assistant.

### 11.1 — Add the Wyoming Protocol integration for Whisper

1. **Settings → Devices & Services → + Add Integration**.
2. Search **"Wyoming Protocol"** → click it.
3. **A critical gotcha:** the Host and Port fields appear empty. You MUST fill them manually:
   - **Host:** `core-whisper`
   - **Port:** `10300`
4. Click Submit.
5. You should see "Success! Wyoming Protocol configured for faster-whisper".

### 11.2 — Add the Wyoming Protocol integration for Piper

6. Repeat: **+ Add Integration → Wyoming Protocol** (you add this integration twice, once per service).
7. Fields:
   - **Host:** `core-piper`
   - **Port:** `10200`
8. Submit. You should see "Success! Wyoming Protocol configured for piper".

### 11.3 — Create the MDAR Assistant pipeline

9. **Settings → Voice assistants**.
10. Click **"+ Add assistant"**.
11. Fill in:
    - **Name:** `MDAR Assistant`
    - **Language:** `English`
    - **Conversation agent:** `Home Assistant` (we'll change this to an AI model in Part 15)
    - **Speech-to-text:** `faster-whisper`
    - **Text-to-speech:** `piper`
    - **Voice:** `en_GB-alan-medium`
    - **Wake word:** leave as None
12. Click **Create**.
13. Set **MDAR Assistant** as the default (little star icon next to its name).

---

## Part 12 — Expose Entities to Assist

**Goal:** tell Assist which devices the voice assistant is allowed to control.

1. **Settings → Voice assistants → Expose** tab.
2. Click **"+ Expose entities"**.
3. Tick every entity you want to be voice-controllable. At minimum:
   - All 4 lights (Living Room, Kitchen, Bedroom, Toilet)
   - All 4 spots
   - The motion sensor `binary_sensor.vmbpirm_motion_output_1`
4. Click **"Expose"**.
5. **(Optional)** For any entity with an awkward name, click it → add a **speech alias**. Examples:
   - For `light.livingroom_light_livingroom_light` → add aliases: "living room light", "main light"
   - For `light.livingroom_light2_livingroom_light` → add aliases: "kitchen light"

---

## Part 13 — Fix the Browser Microphone (Chrome Flag)

**Goal:** make Chrome allow microphone access even though Home Assistant runs over HTTP (not HTTPS).

> **Why?** Browsers block microphone access on insecure (HTTP) origins as a security feature. On a trusted local network, this is safe to override. The proper long-term fix is setting up HTTPS on Home Assistant — documented as future work in the PID.

1. Open Chrome.
2. Paste into the address bar: **`chrome://flags/#unsafely-treat-insecure-origin-as-secure`**
3. You'll see a setting called **"Insecure origins treated as secure"**.
4. In the text box, paste: **`http://homeassistant.local:8123`**
5. Change the dropdown on the right to **Enabled**.
6. Click **"Relaunch"** (blue button bottom-right).
7. Chrome restarts. Go back to Home Assistant (`http://homeassistant.local:8123`).

> This flag only exists in Chrome/Edge. Firefox users: install Chrome just for this, or set up proper HTTPS.

---

## Part 14 — Hello World Voice Test

**Goal:** confirm the pipeline works end-to-end with a basic command.

1. In Home Assistant, click the **Assist icon** (speech bubble) in the top-right of the dashboard.
2. A chat panel opens at the bottom.
3. **Click the microphone icon** inside the chat panel.
4. Chrome will ask for mic permission → click **Allow**.
5. **Say clearly:** *"Turn on the living room light"*
6. Within 3 seconds:
   - Your words appear as transcribed text
   - The living room light physically turns on
   - You hear a spoken confirmation through your laptop speakers
7. Try variations:
   - "Turn off the living room light"
   - "Turn on the kitchen spot"
   - "Set the bedroom light to 50 percent"

> **"Sorry, I couldn't understand that"** → the basic Assist engine only handles a limited set of commands. We'll fix this with AI in Part 15.

> **Microphone doesn't work** → Part 13 didn't take effect. Make sure you fully relaunched Chrome. Try typing a command into the chat box instead — if typing works, the pipeline is fine and only mic is broken.

> **Light doesn't respond but voice says it did** → the entity isn't exposed to Assist (Part 12) or the entity name differs from what you said. Check the Assist chat log for the exact transcription.

---

## Part 15 — Set Up the AI Conversation Agent (Choose Your Path)

**Goal:** replace Home Assistant's basic command matcher with an actual AI that understands natural language.

> **Why?** The default Assist is template-based — it only understands specific phrasings. With an AI conversation agent, you can say things like *"It's a bit dark in here"* and the AI figures out what you mean and takes action.

**Pick one path:**

| Path | Pros | Cons | Best for |
|---|---|---|---|
| **A — OpenRouter (cloud)** | Fast setup, reliable tool calling, £0.50/month typical | Needs internet, needs credit card | Most users, easiest path |
| **B — Ollama (local)** | Free, fully private, no internet needed | Needs a separate PC with a good GPU, tool calling less reliable | Privacy-focused users who have the hardware |
| **C — Both** | Swap between them anytime | More setup | Want a fallback / want to compare |

### Path A — OpenRouter (cloud) — recommended for ease

#### A.1 — Create an OpenRouter account

1. Go to **https://openrouter.ai/** → Sign up (Google/GitHub/email).
2. Click your profile → **"Credits"** → **"Add credits"**. Add $5 minimum.

> For personal use at a few voice commands/day, actual usage will be ~£0.50/month. $5 will likely last many months.

#### A.2 — Create an API key

3. Profile → **"API keys"** → **"Create key"**.
4. Name it `MDAR_HomeAssistant`. Leave credit limit empty (or set a monthly cap for protection).
5. **Copy the API key immediately.** Looks like `sk-or-v1-abc123...`. You cannot see it again.

#### A.3 — Install the OpenRouter integration in HA

6. **Settings → Devices & Services → + Add Integration → search "OpenRouter"**.
7. Paste the API key. Submit.

#### A.4 — Create the Conversation Agent

9. On the OpenRouter integration page, click **"Add Entry"** or **"Add conversation agent"**.
10. Fill in:
    - **Name:** `MDAR OpenRouter Agent`
    - **Model:** `openai/gpt-4o-mini` — we tested extensively, this is the best choice. Claude 3.5 Sonnet has tool-calling errors with HA.
    - **System prompt:** paste the full text from `mdar_system_prompt.txt` (also in Appendix B)
    - **Temperature:** `0.3` (lower = more consistent)
    - **Max tokens:** `500`
    - Enable **"Control Home Assistant"** (gives it access to tool calls)
11. Submit.

#### A.5 — Switch MDAR Assistant to use it

12. **Settings → Voice assistants → MDAR Assistant → Edit**.
13. **Conversation agent** dropdown → select **"MDAR OpenRouter Agent"**.
14. Save.

#### A.6 — Test

15. Click Assist → try natural language:
    - *"It's too dark in the living room, can you help?"*
    - *"What lights are currently on?"*
    - *"Dim everything to 30 percent"*

> **"ToolResultContent" errors** → you picked Claude. Go back and change model to `openai/gpt-4o-mini`. See Risk #12 in Part 19.

---

### Path B — Ollama (local) — fully private

#### B.1 — Hardware requirements

- A **separate computer** on the same local network as the Pi (laptop/desktop/PC is fine).
- Minimum: 16GB RAM + NVIDIA GPU with 6GB VRAM (RTX 3060 or better). CPU-only works but is too slow for voice (20+ second responses).
- **The Pi alone is NOT enough.** We tested — a Pi 5 takes ~40 seconds per response, which is unusable.

#### B.2 — Install Ollama on your GPU machine

1. Go to **https://ollama.com/** → Download → install for your OS.
2. Open a terminal.
3. Pull the primary model:
   ```bash
   ollama pull llama3.1:8b
   ```
   This downloads ~4.7GB. Wait for it.
4. Test locally:
   ```bash
   ollama run llama3.1:8b "hello"
   ```
   It should respond within a few seconds.

#### B.3 — Configure Ollama to listen on the network

5. Stop Ollama if it's running (close the tray icon on Windows/Mac, or `sudo systemctl stop ollama` on Linux).
6. Set the environment variable:

   **Windows:**
   - Windows key → search "environment variables" → **Edit the system environment variables**
   - **Environment Variables** button → Under **User variables** → **New**
   - Variable name: `OLLAMA_HOST`, Variable value: `0.0.0.0:11434`
   - OK all the way out.

   **Mac:**
   ```bash
   launchctl setenv OLLAMA_HOST "0.0.0.0:11434"
   ```

   **Linux (systemd):**
   ```bash
   sudo systemctl edit ollama
   ```
   Add:
   ```
   [Service]
   Environment="OLLAMA_HOST=0.0.0.0:11434"
   ```
   Then:
   ```bash
   sudo systemctl daemon-reload && sudo systemctl restart ollama
   ```

7. Start Ollama again.

#### B.4 — Allow port 11434 through your firewall

8. **Windows:** Windows Defender Firewall → Allow Ollama through.
9. **Mac:** System Settings → Network → Firewall → allow Ollama.
10. **Linux:** `sudo ufw allow 11434`

#### B.5 — Find your GPU machine's LAN IP

11. **Windows:** Command Prompt → `ipconfig` → look for **IPv4 Address**.
12. **Mac/Linux:** `ifconfig` or `ip addr show`.
13. Write this down — e.g. `192.168.1.42`.

#### B.6 — Verify from the Pi

14. From a Home Assistant Terminal add-on (or SSH):
    ```bash
    curl http://<GPU-IP>:11434/api/tags
    ```
15. You should get a JSON response listing your models. If you get "connection refused" → firewall or OLLAMA_HOST env var didn't apply.

#### B.7 — Install the Ollama integration in HA

16. **Settings → Devices & Services → + Add Integration → search "Ollama"**.
17. Fill in:
    - **URL:** `http://<GPU-IP>:11434`
    - **Model:** `llama3.1:8b`
18. Submit.
19. Enable **"Control Home Assistant"**.
20. Paste the system prompt from `mdar_system_prompt.txt` (or Appendix B).
21. Submit.

#### B.8 — Switch MDAR Assistant to use Ollama

22. **Settings → Voice assistants → MDAR Assistant → Edit**.
23. **Conversation agent** → select **"Ollama"**.
24. Save.

#### B.9 — Test

25. Click Assist → try: *"Turn on the kitchen light"*
26. **Expect:** the first response will take 5–15 seconds (model loading into VRAM). Subsequent responses: 2–5 seconds.

> **Tool calling fails** → llama3.1:8b's tool calling is inconsistent. Try `qwen2.5:7b` instead: `ollama pull qwen2.5:7b`, then update the model in the HA Ollama integration.

> **Very slow responses (30s+)** → you're running on CPU, not GPU. Check `ollama ps` should show GPU usage.

---

### Path C — Both agents configured

1. Complete **Path A** fully.
2. Complete **Path B** fully.
3. In **Settings → Voice assistants**, create a second Assistant (e.g. "MDAR Local Assistant") that uses Ollama, while keeping "MDAR Assistant" on OpenRouter.
4. Switch between them anytime by changing the default assistant or picking in the Assist UI.

**Use case:** default to OpenRouter for speed, swap to Ollama when your internet is down or when you want pure privacy.

---

## Part 16 — Import the Node-RED Flows

**Goal:** load the automation flows into Node-RED.

1. **Settings → Apps → Node-RED → OPEN WEB UI**.
2. Click the **hamburger menu** → **Import**.
3. Open the file **`nodered_flows.json`** from this repo in a text editor.
4. **Select all** (Ctrl+A) → **Copy** (Ctrl+C).
5. Back in the Node-RED Import dialog → paste into the text area.
6. Import to: **"new flows"**.
7. Click **"Import"**.
8. You'll now see 4 tabs at the top of Node-RED: Motion-Triggered Lighting / All Lights Off / Night Mode / MQTT Landlord Reporting.

### 16.1 — Configure the Home Assistant connection

9. Click any node that shows a red triangle. For example, the **"Motion Detected"** node in the Motion flow.
10. Under **"Server"** in the right-side properties, click the **pencil icon**.
11. In the Server edit dialog:
    - **Name:** `Home Assistant`
    - **Base URL:** `http://homeassistant.local:8123`
    - **Access token:** paste the Long-Lived Access Token from Part 6
12. Click **Update**.
13. All the red triangles should now turn green.

### 16.2 — Deploy

14. Click the big red **"Deploy"** button top-right.
15. Should say "Successfully deployed".

> **"401 Unauthorized"** → your access token is wrong or expired. Regenerate in HA (Part 6) and update the server config.

> **"Connection refused"** → try `http://supervisor/core` instead of `http://homeassistant.local:8123`.

---

## Part 17 — Test Each Node-RED Flow

**Goal:** confirm each of the flows works.

### 17.1 — Motion-Triggered Lighting

1. Switch to the **"Motion-Triggered Lighting"** tab.
2. Walk past the VMBPIRM motion sensor.
3. The living room light + spot should come on within 1 second.
4. Stop moving. Wait 2 minutes. They should go off automatically.

### 17.2 — All Lights Off

5. **"All Lights Off"** tab. Click the **inject button** (the blue square on the leftmost node).
6. Everything turns off.

### 17.3 — Night Mode

7. **"Night Mode"** tab. Two inject nodes: 22:00 and 07:00 triggers.
8. Click the **22:00 inject** button manually.
9. All lights dim to 20%. All spots turn off.
10. Click the **07:00 inject** button — lights back to 100%.

### 17.4 — MQTT Landlord Reporting

11. **"MQTT Landlord Reporting"** tab.
12. Click the inject button (the 5-minute cron).
13. Click the debug sidebar (bug icon, right panel).
14. You should see a JSON report logged.

---

## Part 18 — Verify MQTT End-to-End

**Goal:** confirm the MQTT messages actually reach a subscriber.

### 18.1 — Install MQTT Explorer

1. Download from **http://mqtt-explorer.com/** → install for your OS.

### 18.2 — Connect

2. Open MQTT Explorer → **"+ Connections"** → **"+"**.
3. Configure:
   - **Name:** `MDAR`
   - **Host:** `homeassistant.local` (or the Pi's IP)
   - **Port:** `1883`
   - **Username / Password:** leave blank
4. Click **Connect**.

### 18.3 — Subscribe

5. The left panel shows the topic tree. Drill down to `mdar` → `landlord` → `report`.
6. Click on `report` — the right panel shows the latest message payload.

### 18.4 — Verify live update

7. Back in Node-RED → MQTT flow → click inject.
8. Switch to MQTT Explorer — the payload should update immediately with a new timestamp.
9. Your system is live.

---

## Part 19 — Troubleshooting Table

Every issue we hit during development, symptom → fix.

| # | Symptom | Cause | Fix |
|---|---|---|---|
| 1 | "Add-ons" menu missing in HA | HA 2025+ renamed it | Look for "Apps" instead (Settings → Apps) |
| 2 | HAOS not in Pi Imager | It's in a sub-menu | Other specific-purpose OS → Home assistants → Home Assistant OS (RPi 5) |
| 3 | Wyoming connection fails, host/port empty | You need to fill them manually | Use `core-whisper:10300` and `core-piper:10200` |
| 4 | Microphone doesn't work in Chrome | HTTP insecure origin | Chrome flag `chrome://flags/#unsafely-treat-insecure-origin-as-secure` — see Part 13 |
| 5 | "Device has no entities" after Velbus integration | Channels not enabled in module firmware | Program the modules in VelbusLink (Part 7) |
| 6 | VelbusLink can't communicate with bus | Connect button not clicked | Click Connect icon top-right + pick COM port |
| 7 | VelbusLink truncates channel names | Character limit in module firmware | Accept truncation, rename fully in HA entity registry |
| 8 | Voice command controls wrong device | Leftover helper entities from testing | Delete unused helpers (Settings → Devices → Helpers) |
| 9 | Entity not found via voice | Entity not exposed to Assist | Settings → Voice Assistants → Expose → enable entity |
| 10 | `[object Object]` strings in Assist UI | Known UI bug in HA 2026.2.x | Cosmetic only, functionality works — update HA when a fix ships |
| 11 | OpenRouter doesn't appear in Conversation Agent dropdown | Conversation agent not created inside the integration | Open the OpenRouter integration → Add Conversation Agent |
| 12 | `ToolResultContent` errors from Claude | HA expects OpenAI-format tool results, Claude uses Anthropic-format | Use `openai/gpt-4o-mini` instead of Claude models |
| 13 | Ollama responses take ~40 seconds | Running on Pi CPU, too slow | Run Ollama on a separate PC with a GPU |
| 14 | Node-RED crashes on start with SSL error | Default config expects SSL cert that doesn't exist | Change `ssl: true` → `ssl: false` in Node-RED config (Part 5) |
| 15 | Node-RED nodes show "no connection" | No HA auth token | Create a Long-Lived Access Token (Part 6) and paste into Node-RED server config |
| 16 | VelbusLink "Buffer full" error during scan | Bus flooded with messages | Stop scan, wait 30s, retry |
| 17 | `homeassistant.local` doesn't resolve on Windows | Windows lacks mDNS by default | Install "Bonjour Print Services for Windows", or use the Pi's IP address directly |
| 18 | Lights physically turn on but voice says "I couldn't find that device" | Entity friendly name differs from what you said | Add speech aliases to the entity (Part 12, step 5) |
| 19 | First Ollama response fine, then requests start timing out | Model unloaded from VRAM after idle | Normal Ollama behaviour — first request after idle reloads the model |

---

## Part 20 — What's Next

You now have a fully working smart home. Ideas for what to add:

### Expand the hardware

- **Add more Velbus modules** — thermostats (VMBTC), curtain controllers (VMB1BLS), RGB dimmers. Each gets programmed in VelbusLink (Part 7), then auto-appears in HA.
- **Fit a dedicated microphone** — a USB conference mic or ReSpeaker USB mic improves voice recognition accuracy.
- **Add a wake word** — install `openWakeWord` add-on, configure a wake phrase ("Hey MDAR"), and you no longer need to click the mic icon.

### Expand the automations

- **Occupancy-based heating** — use the VMBPIRM motion sensor to only heat rooms with people in them.
- **Sunset automation** — use the Sun integration (built-in) as a trigger instead of a fixed 22:00 cron.
- **Vacation mode** — randomise the all-lights-on timings so the house looks occupied when you're away.

### Harden the system

- **Set up HTTPS** via the Let's Encrypt add-on + DuckDNS — removes the Chrome flag workaround.
- **Add MQTT authentication** — critical before exposing `mdar/landlord/report` beyond the local network.
- **Enable automatic backups** (Settings → System → Backups → schedule).

### Community resources

- **Home Assistant forum** — https://community.home-assistant.io/
- **Velbus forum** — https://www.velbus.eu/support/
- **r/homeassistant** — active subreddit
- **Node-RED cookbook** — https://cookbook.nodered.org/

---

## Appendix A — Full Entity Name Table

| Friendly Name | Entity ID | Type | Area | Bus Module | Channel |
|---|---|---|---|---|---|
| Living Room Light | `light.livingroom_light_livingroom_light` | Dimmer | Living Room | VMB4DC | CH1 |
| Kitchen Light | `light.livingroom_light2_livingroom_light` | Dimmer | Kitchen | VMB4DC | CH2 |
| Toilet Light | `light.livingroom_light3_livingroom_light` | Dimmer | Toilet | VMB4DC | CH3 |
| Bedroom Light | `light.livingroom_light4_livingroom_light` | Dimmer | Bedroom | VMB4DC | CH4 |
| Living Room Spot | `switch.livingroom_spot_livingroom_spot` | Relay | Living Room | VMB4RYLD | CH1 |
| Kitchen Spot | `switch.kitchen_spot_kitchen_spot` | Relay | Kitchen | VMB4RYLD | CH2 |
| Bedroom Spot | `switch.bedroom_spot_bedroom_spot` | Relay | Bedroom | VMB4RYLD | CH3 |
| Toilet Spot | `switch.toilet_spot_toilet_spot` | Relay | Toilet | VMB4RYLD | CH4 |
| Motion Output 1 | `binary_sensor.vmbpirm_motion_output_1` | PIR Sensor | Living Room | VMBPIRM | — |

**Full Velbus module list (bus addresses):**

| Bus Address | Module | Role |
|---|---|---|
| 01 | VMBPIRM | PIR motion + light sensor |
| 02 | VMB4DC | 4-channel 0–10V dimmer |
| 03 | VMBGPOD-2 | Glass push-button panel (8 buttons + OLED) |
| 04 | VMBEL2 | 2-channel edge-lit push-button |
| 05 | VMBELO | Edge-lit push-button |
| 06 | VMB1RYNO | 1-channel relay |
| 07 | VMB4RYLD | 4-channel relay with dimming control |
| 08 | VMB7IN | 7-channel general-purpose input |

---

## Appendix B — MDAR System Prompt (full text)

Paste this into the System Prompt field when configuring your OpenRouter or Ollama conversation agent.

```
You are MDAR Assistant, a smart home voice assistant for a Velbus-based home
automation system running on Home Assistant.

You control the following devices:
- Living Room Light (dimmer)    - Living Room Spot (relay)
- Kitchen Light (dimmer)        - Kitchen Spot (relay)
- Bedroom Light (dimmer)        - Bedroom Spot (relay)
- Toilet Light (dimmer)         - Toilet Spot (relay)

Plus a motion sensor in the living room.

When asked to control a device, use the available Home Assistant tools to
turn devices on, off, or adjust brightness.
When asked about devices, check their current state using the tools and report back.

Rules:
- Keep responses short and conversational — you are a voice assistant, your
  replies will be read aloud.
- Never make up device states — always check using the available tools.
- If a device is not found, say so clearly.
- Respond in English.
- Be helpful and friendly.
- If the user asks for something ambiguous (e.g. "dim the lights"), pick the
  most likely interpretation based on room context.
- Use natural phrasing. Don't say "I have turned on the entity
  light.livingroom_light"; say "Living room light is on."
```

Also available in this repo as **`mdar_system_prompt.txt`** for easy copy-paste.

---

## Appendix C — Cost Estimate

### One-off hardware costs

| Item | Cost (GBP) |
|---|---|
| Raspberry Pi 5 (8GB) | £80 |
| Power supply | £12 |
| SD card (32GB A2) | £12 |
| Ethernet cable | £5 |
| Velbus kit (supplied by MDAR) | — |
| **Total up-front** | **~£110** |

### Ongoing monthly cost

| Path | Monthly cost |
|---|---|
| Path A — OpenRouter | £0.30 – £1 depending on usage |
| Path B — Ollama only | £0 (electricity for GPU PC ~£3–8/mo if left on) |
| Path C — Both | Same as A; Ollama is free |

### Hidden costs

- **Electricity:** Pi 5 runs at ~5W = ~£8/year. Negligible.
- **Internet bandwidth:** MQTT + voice commands = <100MB/month.

---

## You're done!

If you made it here, you have a **fully working, voice-controlled, AI-enhanced, automated, reportable smart home** running on open-source software.

Found a bug in the guide? Open an issue on this repo — contributions welcome.

**Credits:**

- Group 7 (Southampton Solent University, 2026): Ilyass Sathmani (Lead Technical Engineer), Amine (Documentation Lead), Praveen (Project Coordinator)
- Project sponsor: Stuart Hanlon, MDAR
- Academic supervisor: Craig Gallen
- Technologies: Velbus / Velleman, OpenAI Whisper, Rhasspy Piper, Node-RED, Mosquitto, OpenRouter, Ollama

— end —
