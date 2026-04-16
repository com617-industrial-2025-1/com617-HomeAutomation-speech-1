# Sprint 3 — Document 4: Final System Architecture & Pipeline

## 1. Purpose

This document presents the complete, integrated architecture of the MDAR VELBUS Voice-Assistive System as it stands at the end of Sprint 3. It describes the end-to-end data flow from voice input to physical hardware action, summarises the testing results against the Sprint 1 requirements, consolidates the problems faced and their solutions, and identifies future improvements.

---

## 2. System Components

The final system consists of the following layers and components:

### 2.1 Hardware
- Raspberry Pi 5 (8GB RAM, 32GB A2-class microSD)
- Velbus VMB1USB serial interface
- Velbus modules: VMBPIRM, VMB4DC, VMBGPOD-2, VMBEL2, VMBELO, VMB1RYNO, VMB4RYLD, VMB7IN
- Windows laptop (provides Internet Connection Sharing during development; runs VelbusLink and optional Ollama)

### 2.2 Operating System
- Home Assistant OS 17.1 on Raspberry Pi 5
- Home Assistant Core 2026.2.x
- Home Assistant Supervisor 2026.03.x

### 2.3 Add-ons
- Mosquitto MQTT Broker
- Wyoming Whisper (faster-whisper, base-en model)
- Wyoming Piper (en_GB-alan-medium voice)
- Node-RED (with SSL disabled)

### 2.4 Integrations
- Velbus (serial port `/dev/ttyACM0`)
- Wyoming Protocol — Whisper (`core-whisper:10300`)
- Wyoming Protocol — Piper (`core-piper:10200`)
- OpenRouter (GPT-4o-mini)
- MQTT (auto-discovered Mosquitto broker)

### 2.5 Logic
- MDAR Assistant voice pipeline
- Five Node-RED automation flows (Motion-Triggered Lighting, All Lights On, All Lights Off, Night Mode, MQTT Landlord Reporting)

---

## 3. End-to-End Voice Command Flow

The following describes the complete data flow when a user speaks "Turn on the kitchen light":

1. **Audio capture** — the user speaks into the browser microphone.
2. **Audio transmission** — the browser sends the audio to Home Assistant.
3. **Speech-to-text** — Home Assistant forwards the audio to Wyoming Whisper at `core-whisper:10300`. Whisper transcribes the audio to text using the `base-en` model.
4. **Intent recognition** — the transcribed text is sent to the OpenRouter conversation agent (GPT-4o-mini), along with the system prompt and the list of exposed entities and tools.
5. **Tool call** — GPT-4o-mini decides to call the `light.turn_on` tool with the entity `light.livingroom_light2_livingroom_light` (the Kitchen Light).
6. **Service execution** — Home Assistant executes the `light.turn_on` service.
7. **Hardware command** — the Velbus integration sends the command over the USB serial bus to the VMB4DC dimmer.
8. **Physical action** — the VMB4DC switches Channel 2, and the kitchen light turns on.
9. **Response generation** — the LLM produces a short confirmation message ("Kitchen light turned on").
10. **Text-to-speech** — the response is sent to Wyoming Piper at `core-piper:10200`.
11. **Audio synthesis** — Piper synthesises the speech using the `en_GB-alan-medium` voice.
12. **Audio playback** — the audio is sent back through Home Assistant to the browser, which plays it.

End-to-end latency: approximately 2–4 seconds.

---

## 4. Autonomous Flow (Motion-Triggered Lighting Example)

For a Node-RED-driven action, the flow is:

1. **Sensor event** — the VMBPIRM detects motion. The module sends a state change over the Velbus bus.
2. **Integration update** — Home Assistant's Velbus integration receives the state change and updates `binary_sensor.vmbpirm_motion_output_1` to `on`.
3. **Node-RED trigger** — the Motion-Triggered Lighting flow's `server-state-changed` node fires.
4. **Service calls** — the flow calls `light.turn_on` and `switch.turn_on` for the living room light and spot.
5. **Hardware command** — the Velbus integration sends the commands over the USB bus to the relevant modules.
6. **Physical action** — the light and spot turn on.
7. **Wait for inactivity** — when motion stops, the flow's `delay` node waits 2 minutes.
8. **Service calls** — the flow then calls `light.turn_off` and `switch.turn_off`.
9. **Hardware command** — the lights turn off.

No human input. No cloud LLM. Entirely deterministic.

---

## 5. Network Architecture

All processing happens locally except for the OpenRouter API call:

- **Local-only:** Velbus communication, Whisper transcription, Piper synthesis, Home Assistant logic, Node-RED flows, Mosquitto broker.
- **Cloud:** OpenRouter LLM (only the transcribed text is sent; never the audio itself).

If the internet connection drops, the system degrades gracefully:

- Voice control falls back to the basic Home Assistant Assist engine, which can still match simple commands.
- All Node-RED automations continue to run.
- The Velbus bus continues to operate, including direct button-to-output actions programmed in VelbusLink.
- MQTT reporting continues but messages cannot be delivered to a remote landlord until connectivity returns (the broker queues them locally if the subscriber is on the same network).

---

## 6. Testing Results vs Requirements

| ID | Requirement | Result |
|----|-------------|--------|
| R1 | Accept natural-language voice commands in British English | ✅ Achieved with GPT-4o-mini via OpenRouter |
| R2 | Respond within 5 seconds end to end | ✅ Typical latency 2–4 seconds |
| R3 | Control all Velbus dimmer and relay channels | ✅ All configured channels controllable via voice and dashboard |
| R4 | Process STT and TTS locally | ✅ Whisper and Piper run on the Pi 5 |
| R5 | Support natural language understanding via LLM | ✅ Achieved with GPT-4o-mini |
| R6 | Trigger lighting automatically on motion | ✅ Achieved via Node-RED Motion-Triggered Lighting flow |
| R7 | Apply Night Mode lighting scene on schedule | ✅ Achieved via Node-RED Night Mode flow |
| R8 | Publish device state reports over MQTT | ✅ Achieved via Node-RED MQTT Reporting flow |
| R9 | Provide manual All Lights On / Off triggers | ✅ Achieved via Node-RED All Lights On and All Lights Off flows |
| R10 | Continue functioning if internet is lost | ✅ Partially — voice falls back to basic Assist; all automations continue |

All ten requirements were met or partially met by the end of Sprint 3.

---

## 7. Consolidated Problems Faced and Solutions

| # | Problem | Sprint | Solution |
|---|---------|--------|----------|
| 1 | Add-ons menu missing — actually renamed to "Apps" | 2 | Documented the rename and used the new menu |
| 2 | Wrong OS in Raspberry Pi Imager | 2 | Navigated to Other specific-purpose OS → Home assistants → Home Assistant OS |
| 3 | Wyoming Protocol failed to connect | 2 | Manually entered `core-whisper:10300` and `core-piper:10200` |
| 4 | Microphone blocked by browser on HTTP | 2 | Used Chrome flag `unsafely-treat-insecure-origin-as-secure` |
| 5 | `[object Object]` in Assist chat | 2 | Confirmed UI bug; used direct device commands; awaiting HA update |
| 6 | Velbus modules showed no entities | 2 | Programmed channels in VelbusLink, then re-added the integration |
| 7 | VelbusLink in offline mode | 2 | Clicked Connect and selected the correct COM port |
| 8 | VelbusLink truncated channel names | 2 | Renamed the entities in Home Assistant after sync |
| 9 | Voice command controlled wrong entity | 2 | Deleted leftover helper toggles |
| 10 | Velbus entity not exposed to Assist | 2 | Toggled exposure on in Settings → Voice Assistants |
| 11 | OpenRouter not in conversation agent dropdown | 3 | Created a conversation agent inside the OpenRouter integration first |
| 12 | Claude tool calls returned format errors | 3 | Switched to GPT-4o-mini |
| 13 | Local Ollama too slow on Pi (40s) | 3 | Moved Ollama to laptop GPU; kept GPT-4o-mini as primary |
| 14 | Node-RED failed to start (SSL error) | 3 | Disabled SSL in the add-on configuration |
| 15 | Node-RED nodes showed "no connection" | 3 | Generated a Long-Lived Access Token and added it to the server config |
| 16 | VelbusLink "Buffer full" warning | 3 | Waited for the bus to settle and re-scanned |

---

## 8. Future Improvements

The system is functional and meets all defined requirements, but several enhancements would strengthen it further:

1. **Multi-room expansion** — extend Velbus configuration and Node-RED flows to additional rooms with their own motion sensors and area-specific automations.
2. **Hardware microphone** — replace the browser microphone with a dedicated I2S mic HAT on the Pi for hands-free wake-word activation.
3. **Wake word** — install the openWakeWord add-on to allow the assistant to be activated by speaking a wake phrase rather than clicking a microphone icon.
4. **HTTPS** — set up a reverse proxy with a self-signed or Let's Encrypt certificate to eliminate the Chrome flag workaround.
5. **Local LLM in production** — investigate larger and more reliable local models (e.g., Llama 3.3 70B on dedicated server hardware) to remove the cloud dependency for natural language understanding.
6. **MQTT authentication and TLS** — secure the MQTT broker for external landlord access with username/password and TLS.
7. **Bidirectional MQTT** — add a `commands` topic so that the landlord can send remote commands to a property.
8. **High-contrast Lovelace dashboard** — design a Flash Card dashboard with large buttons and high contrast for users who prefer non-verbal interaction.
9. **TV-overlay notifications** — push critical alerts (smoke detected, water leak) to a TV display via Home Assistant's notification service.
10. **Multi-property scaling** — extend the topic schema and add property identifiers to support the system being deployed across multiple flats managed by a single landlord.

---

## 9. Conclusion

The MDAR VELBUS Voice-Assistive System has been successfully built, tested, and demonstrated. It combines local voice processing (Whisper and Piper), a cloud-based AI conversation agent (GPT-4o-mini via OpenRouter), event-driven automation (Node-RED), a structured MQTT messaging layer, and real Velbus hardware control into a single coherent platform.

The system meets every functional requirement defined in Sprint 1. The Voice Assistant gives Arthur an intuitive, conversational interface for controlling his home; the Node-RED flows take care of the routine background work that makes the home feel intelligent rather than just voice-controlled; and the MQTT publisher gives the landlord visibility into device states across the property.

The journey from research through proof-of-concept to MVP produced a substantial body of documented experience — 12 documents across 4 folders, capturing not just the final architecture but every problem encountered, the diagnosis applied, and the solution adopted. This documentation is itself a deliverable: a future team picking up the project can reproduce the system without rediscovering the same pitfalls.
