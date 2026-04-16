# Sprint 2 — Document 1: Operating System & Platform Setup

## 1. Purpose

This document describes the installation of Home Assistant OS on the Raspberry Pi 5, the network configuration that allowed the Pi to connect to the internet through a laptop, the initial onboarding process, and the installation of the core add-ons that the rest of the system depends on. It is the first practical document in Sprint 2 and represents the moment when the project moved from research into hands-on implementation.

---

## 2. Hardware Preparation

The hardware used was:

- A Raspberry Pi 5 with 8GB of RAM.
- A 32GB A2-class microSD card.
- The official Raspberry Pi 5 USB-C power supply.
- A standard ethernet cable.
- A Windows laptop used both for flashing the SD card and for sharing internet to the Pi.

---

## 3. Flashing Home Assistant OS

The team used the official Raspberry Pi Imager (v2.0.6) to flash the SD card. A common point of confusion was that the default OS list shows Raspberry Pi OS variants, not Home Assistant OS. The correct path within the Imager is:

**Other specific-purpose OS → Home assistants → Home Assistant → Home Assistant OS (RPi 5)**

Once the correct image was selected and the SD card was chosen as the target, the Imager downloaded and wrote the HAOS image directly. Home Assistant OS replaces Raspberry Pi OS entirely — it is not an application that runs on top of Raspberry Pi OS. This was an important clarification during setup, as the team initially considered installing both.

---

## 4. Network Configuration via Internet Connection Sharing

The Pi was connected to the laptop via an ethernet cable. To give the Pi internet access through the laptop's WiFi, Windows Internet Connection Sharing (ICS) was configured:

1. Opened **Network Connections** on Windows.
2. Right-clicked the **Wi-Fi** adapter and opened **Properties**.
3. Switched to the **Sharing** tab.
4. Enabled **"Allow other network users to connect through this computer's internet connection"**.
5. Set the **Home networking connection** dropdown to the **Ethernet** adapter (the one connected to the Pi).

After this configuration, the laptop's ethernet adapter received the IP address `192.168.137.1` and the Pi was assigned an IP from the same subnet (e.g., `192.168.137.57`).

A common mistake during this step was enabling sharing on the wrong adapter. Sharing must be enabled on the adapter that **has** internet (WiFi) and the dropdown must point to the adapter that should **receive** the shared connection (Ethernet).

---

## 5. First Boot and Onboarding

After flashing and inserting the SD card, the Pi was powered on with the ethernet cable connected. The first boot took approximately 20 minutes as Home Assistant OS downloaded and installed its components.

Once boot was complete, the Home Assistant web UI was accessed from the laptop's browser at:

```
http://homeassistant.local:8123
```

The onboarding wizard prompted the team to:

1. Create an administrator user account.
2. Name the home installation (chosen: **MDAR Lab**).
3. Set the geographic location for sunrise/sunset automations.
4. Skip the initial device discovery (devices would be added manually later).

A key observation during onboarding: the Pi's local console (visible if a monitor is attached) shows a `homeassistant login:` prompt. This is **not** the main interface — it is a recovery/diagnostic console. All real interaction with Home Assistant happens through the web UI in a browser.

---

## 6. The "Apps" Menu (formerly "Add-ons")

In Home Assistant 2025+, the menu item previously called "Add-ons" was renamed to **"Apps"**. This caused initial confusion when following older documentation that referenced "Add-ons". The functionality is identical — only the menu label changed. The team confirmed this by checking the official Home Assistant documentation.

The Apps store is accessed via **Settings → Apps**.

---

## 7. Core Add-on Installation

Four core add-ons were installed in sequence, each verified to be running before moving to the next:

| Add-on | Purpose | Configuration |
|--------|---------|---------------|
| Mosquitto MQTT Broker | Message broker for landlord reporting and inter-component communication | Default settings; Start on boot enabled |
| Node-RED | Visual flow-based automation builder | Default settings; SSL disabled (see Sprint 3) |
| Wyoming Whisper | Speech-to-text engine | Model: `base-en`; Start on boot enabled |
| Wyoming Piper | Text-to-speech engine | Voice: `en_GB-alan-medium`; Start on boot enabled |

Each add-on download took several minutes, with Whisper being the longest because it also downloads the AI model weights on first run. Each was started and confirmed to display a green "Running" status before proceeding.

---

## 8. Verification

At the end of this phase, the system was in the following state:

- Home Assistant OS booted and accessible via the web UI.
- Internet connectivity working through the laptop's shared connection.
- Administrator account created.
- All four core add-ons installed, configured, and running.
- The platform ready for hardware integration (Document 2) and voice pipeline configuration (Document 3).

This phase took approximately one week of effort, with most time spent on the network sharing configuration and waiting for downloads. Once the system was online, subsequent work was significantly faster.
