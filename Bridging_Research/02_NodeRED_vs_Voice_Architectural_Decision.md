# Bridging Research — Document 2: Node-RED vs Voice Chat — Architectural Decision

## 1. Purpose

The PoC at the end of Sprint 2 demonstrated that a user could control a light by speaking to the assistant. But Arthur's daily life involves repetitive patterns: lights on when he enters the living room, lights dimmed at night, status reports sent to the landlord. Asking the assistant to perform every one of these actions manually would defeat the point of an assistive system.

This document captures the team's architectural reasoning about why an automation layer is necessary alongside the conversational layer, why Node-RED was selected, and how the two layers complement each other in the final system design.

---

## 2. The Limitation of a Conversational-Only System

Voice control is reactive — it responds to a user's immediate input. It is excellent for:

- One-off, on-demand actions ("turn off the kitchen light").
- Spontaneous queries ("what is the temperature in the bedroom?").
- Situations where the user knows exactly what they want.

However, voice control breaks down when:

- The user is asleep, away from home, or otherwise unable to speak.
- The desired behaviour is repetitive and predictable (same dim level every night).
- The trigger is not a human action but an environmental event (motion detected, time of day reached, sensor threshold crossed).
- The action is invisible to the user (sending a state report to a landlord).

For Arthur, the most valuable smart home features are precisely the ones that happen **without** him having to ask. Walking into the living room and having the lights come on automatically is far more accessible than reaching for a phone or speaking a command.

---

## 3. The Need for an Event-Driven Automation Layer

What the system needs is a separate layer that:

- Continuously listens for events (sensor state changes, time triggers, MQTT messages).
- Executes pre-defined logic when those events occur.
- Runs entirely without human input or LLM cost.
- Is reliable, predictable, and easy to inspect or modify.

This is fundamentally different from the voice pipeline. The voice pipeline is **interactive** (user → assistant → action). The automation layer is **autonomous** (event → logic → action).

---

## 4. Why Node-RED?

Several options exist for building automation logic on Home Assistant:

### 4.1 Home Assistant Automations (YAML or UI)

Home Assistant has a built-in automation engine that can trigger on state changes, times, and events. It is configured either via YAML or a visual editor.

**Pros:** Built-in, no extra add-on needed, tightly integrated.
**Cons:** Limited visual representation. Complex flows with multiple branches and delays become hard to read. Debugging is awkward.

### 4.2 AppDaemon

AppDaemon is a Python-based automation framework for Home Assistant.

**Pros:** Full Python power, very flexible.
**Cons:** Requires writing code. Higher barrier to entry. Less suitable for visual flow design.

### 4.3 Node-RED

Node-RED is a flow-based programming tool originally developed by IBM for IoT applications. It uses a visual editor where logic is built by connecting nodes representing inputs, transformations, and outputs.

**Pros:** Visual representation makes complex flows easy to understand and modify. Excellent debugging via the built-in debug panel. Wide library of community nodes for everything from MQTT to HTTP. The Home Assistant Node-RED add-on provides direct integration with HA entities and services.
**Cons:** Adds a layer of indirection — flows live in Node-RED rather than HA itself. Requires a separate add-on.

### 4.4 Decision

**Node-RED** was selected because:

1. The project brief explicitly references Node-RED as a starting point.
2. The visual flow paradigm makes it easier to demonstrate the system to non-technical stakeholders (the supporting tutor and sponsor).
3. Complex flows like the MQTT landlord reporting (which involves polling several entities and constructing a JSON document) are much easier to read in Node-RED than in YAML.
4. Node-RED has first-class MQTT support, which is essential for the landlord reporting requirement.

---

## 5. The Two-Layer Architecture

The final system architecture has two complementary layers:

### 5.1 Layer 1 — Conversational (Voice Pipeline)

- Wyoming Whisper (STT) → OpenRouter LLM (intent) → Home Assistant (action) → Wyoming Piper (TTS)
- Triggered by: user speech.
- Used for: spontaneous, on-demand control and queries.

### 5.2 Layer 2 — Autonomous (Node-RED Flows)

- Event sources: sensor state changes, scheduled times, MQTT messages, manual triggers.
- Logic: predefined flows in Node-RED.
- Targets: Home Assistant services (light.turn_on, switch.turn_off, etc.) and MQTT publish.
- Triggered by: events, not humans.
- Used for: motion-triggered lighting, scheduled scenes, periodic reports.

The two layers operate independently but share the same underlying entity model in Home Assistant. A light can be controlled by both a voice command and a motion-triggered automation, with no conflict — each layer simply calls the appropriate Home Assistant service.

---

## 6. Planned Node-RED Flows for Sprint 3

Based on the requirements documented in Sprint 1, the team planned the following flows for implementation:

| Flow | Trigger | Action |
|------|---------|--------|
| Motion-Triggered Lighting | `binary_sensor.vmbpirm_motion_output_1` changes to `on` | Turn on living room light and spot. After 2 minutes of no motion, turn them off. |
| All Lights On | Manual inject button | Turn every light to 100% and turn on every spot |
| All Lights Off | Manual inject button | Turn off every light and every spot |
| Night Mode | Cron schedule at 22:00 daily | Dim all lights to 20%, turn off all spots. At 07:00, restore lights to 100%. |
| MQTT Landlord Reporting | Cron schedule every 5 minutes | Poll all entity states, construct a JSON report, publish to `mdar/landlord/report` |

These flows together cover requirements R6, R7, R8, and R9 from Sprint 1.

---

## 7. Conclusion

The decision to layer Node-RED on top of the voice pipeline was driven by a clear understanding that conversational control alone cannot deliver an accessible smart home. Arthur needs both — the voice assistant for spontaneous interaction, and the autonomous flows for the routine background operation that makes the home feel "smart" rather than just "voice-controlled".

This architectural decision shaped all of Sprint 3 and resulted in a system that is genuinely useful even when the user is silent.
