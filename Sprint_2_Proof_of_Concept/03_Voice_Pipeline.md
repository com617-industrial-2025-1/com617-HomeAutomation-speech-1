# Sprint 2 — Document 3: Voice Pipeline (Whisper + Piper)

## 1. Purpose

This document covers the construction of the local voice pipeline that connects the Whisper speech-to-text engine, the Home Assistant Assist conversation engine, and the Piper text-to-speech engine into a unified voice assistant called **MDAR Assistant**. It also documents the browser microphone permission issues encountered during testing and the workaround used to resolve them. By the end of this phase, a single voice command could toggle a real Velbus light — the "Hello World" milestone for the project.

---

## 2. Wyoming Protocol Integration

The Wyoming Whisper and Wyoming Piper add-ons installed in Document 1 expose their services on internal network endpoints. To make them usable by the Home Assistant Assist pipeline, two **Wyoming Protocol** integrations were added in Home Assistant.

The integrations are added at:

**Settings → Devices & Services → Add Integration → Wyoming Protocol**

For each one, a Host and a Port are entered. By default, both fields are empty and the connection fails — this caught the team out initially. The correct values are:

| Service | Host | Port |
|---------|------|------|
| Wyoming Whisper (STT) | `core-whisper` | `10300` |
| Wyoming Piper (TTS) | `core-piper` | `10200` |

After both integrations were configured, Home Assistant exposed them as available STT and TTS providers in the Voice Assistants configuration.

---

## 3. Building the MDAR Assistant Pipeline

A custom voice assistant was created at:

**Settings → Voice Assistants → Add Assistant**

The configuration was:

| Field | Value |
|-------|-------|
| Name | MDAR Assistant |
| Language | English |
| Conversation Agent | Home Assistant (built-in Assist, replaced in Sprint 3) |
| Speech-to-Text | faster-whisper (via Wyoming Protocol) |
| Text-to-Speech | piper (via Wyoming Protocol) |
| Voice | en_GB-alan-medium |

The "Conversation Agent" was initially set to the built-in Assist engine. In Sprint 3, this was replaced with an OpenRouter-backed LLM to provide natural language understanding.

---

## 4. Exposing Entities to Assist

By default, Home Assistant entities are not exposed to voice assistants. To allow Assist to recognise and control devices, each entity must be explicitly exposed. The team did this for every Velbus light and switch entity:

**Settings → Voice Assistants → Expose** → search for the entity → toggle it on.

Aliases were also added for entities whose technical names did not match natural speech. For example, the entity `light.livingroom_light_livingroom_light` was given the alias **Living Room Light** so that the command "turn on the living room light" would match cleanly.

---

## 5. Browser Microphone Permission Issue

The first attempt to use the voice assistant from the browser produced a red exclamation mark on the microphone icon and the Assist dialog showed `Failed to connect`. Investigation revealed that:

- Home Assistant was being accessed over `http://homeassistant.local:8123` (an insecure origin from the browser's perspective).
- Modern browsers, including Chrome, **block microphone access on insecure origins** for security reasons.
- The browser's permission settings showed "Microphone: Blocked" with the dropdown greyed out — meaning the user could not even change it through the UI.

### 5.1 Workaround

The team used a Chrome experimental flag to mark the local Home Assistant URL as a trusted secure origin:

1. Opened a new Chrome tab to: `chrome://flags/#unsafely-treat-insecure-origin-as-secure`
2. Added `http://homeassistant.local:8123` to the comma-separated list of allowed origins.
3. Set the flag's dropdown from **Disabled** to **Enabled**.
4. Clicked **Relaunch** to restart Chrome.
5. Returned to Home Assistant and clicked the microphone icon — Chrome now prompted for microphone permission, which was granted.

This workaround is acceptable because the system runs entirely on the local network — there is no risk from a malicious website abusing the permission. A proper long-term solution would be to set up HTTPS on Home Assistant using a self-signed certificate or a reverse proxy, but this was deferred as out of scope for the PoC.

---

## 6. Testing the Pipeline

With the microphone permission resolved, the full pipeline was tested. The team spoke the command:

> "Turn on living room light"

The pipeline executed as follows:

1. The browser captured the audio and sent it to Home Assistant.
2. Home Assistant forwarded the audio to Wyoming Whisper, which transcribed it to text.
3. The Assist engine matched the text against the exposed entity "Living Room Light" in the Living Room area.
4. Assist called the `light.turn_on` service for `light.livingroom_light_livingroom_light`.
5. The Velbus integration sent the command over the bus to the VMB4DC dimmer.
6. The physical light turned on.
7. Assist generated a confirmation response, which was sent to Wyoming Piper.
8. Piper synthesised the response audio, which was played back through the browser.

This was the **"Hello World" milestone** of the project — proof that the entire voice pipeline worked end to end with real hardware.

---

## 7. Identified Weaknesses

While the pipeline worked, several limitations were apparent immediately:

### 7.1 Latency

End-to-end latency from speech to physical light response was approximately 3–5 seconds. The bottleneck was Whisper transcription on the Pi 5 CPU. This is within the project's 5-second requirement (R2) but leaves little headroom for slower hardware or larger models.

### 7.2 Basic Command Matching

The built-in Assist engine matches commands using sentence templates and exposed entity names. It does not understand natural language nuance. For example:

- "Turn on the living room light" → works.
- "It's a bit dark in the living room" → fails (Assist does not infer that this means "turn on the light").
- "What devices can you see?" → fails (Assist returned the time as the closest matching response).

This limitation was the primary motivator for adding an LLM-based conversation agent in Sprint 3.

### 7.3 No Natural Conversation

The built-in Assist engine cannot have a conversation, ask follow-up questions, or provide information. It is purely a command parser. For a true voice assistant experience, a more capable conversation agent was needed.

### 7.4 [object Object] Display Bug

When asking general questions, the chat UI sometimes displayed `[object Object]` instead of a readable response. This was identified as a known UI rendering bug in the version of Home Assistant being used. It was expected to be fixed in a future update (2026.2.3 was pending at the time).

---

## 8. Verification and Outcome

By the end of Sprint 2, the system met the following criteria:

- A voice command spoken into a browser microphone successfully controlled a real Velbus light.
- The pipeline used entirely local processing for speech transcription and synthesis.
- Latency was within the project's 5-second requirement.
- The team had a clear understanding of the limitations of the built-in Assist engine, which informed the planning for Sprint 3.

The PoC was demonstrated to the supporting tutor and sponsor in Week 7. Approval was given to proceed to Sprint 3, with the explicit goal of replacing the basic Assist engine with an LLM-based conversation agent and adding the Node-RED automation layer.
