# Sprint 1 — Document 3: Solution Proposal & Project Planning

## 1. Purpose

This document captures the formal solution proposal for the MDAR VELBUS Voice-Assistive System. It defines the user persona that drives the design, presents the proposed solution architecture, allocates roles to team members, and outlines the sprint planning rationale that was approved by the supporting tutor at the end of Sprint 1.

---

## 2. User Persona — "Arthur"

### 2.1 Background

Arthur is a 68-year-old retired engineer who lives alone in a one-bedroom flat in a sheltered housing complex managed by a local housing association. He has limited mobility due to arthritis and finds it difficult to reach light switches that are located on far walls or above his eye level. He has good cognitive function and is comfortable with simple voice commands but finds smartphone apps and complex menus frustrating.

### 2.2 Needs

- Control of all lighting in his flat from a seated or lying position.
- Voice-first interaction — no need to touch a phone, tablet, or wall switch.
- Reliable operation that works the same way every time.
- Privacy — Arthur does not want his voice recordings sent to large cloud companies.
- Background automation that takes care of obvious things (lights on when he enters a room, lights dim at night) without requiring a command every time.

### 2.3 Frustrations

- Existing voice assistants (Alexa, Google Home) are tied to large cloud ecosystems and Arthur is uncomfortable with the privacy implications.
- Smartphone apps for smart bulbs require him to find his phone, unlock it, find the app, and tap through menus — too slow and fiddly.
- Current physical switches in his flat are in inconvenient locations and cannot be relocated easily.

### 2.4 The Landlord

Arthur's flat is one of many managed by the housing association. The building manager needs visibility into the state of devices across all flats to identify maintenance issues (e.g., lights left on for days, sensors not reporting). This is not for surveillance but for operational efficiency.

---

## 3. Statement of Requirements (Point-by-Point)

| ID | Requirement |
|----|-------------|
| R1 | The system shall accept natural-language voice commands in British English. |
| R2 | The system shall respond to commands within 5 seconds end to end. |
| R3 | The system shall control all Velbus dimmer and relay channels installed in the flat. |
| R4 | The system shall process speech-to-text and text-to-speech locally on the Raspberry Pi. |
| R5 | The system shall support natural language understanding via an LLM conversation agent. |
| R6 | The system shall trigger lighting automatically when motion is detected. |
| R7 | The system shall apply a "Night Mode" lighting scene on a daily schedule. |
| R8 | The system shall publish device state reports over MQTT for landlord monitoring. |
| R9 | The system shall provide a manual "All Lights On" and "All Lights Off" trigger. |
| R10 | The system shall continue functioning if the internet connection is lost (with reduced AI capability). |

---

## 4. Proposed Solution Architecture

The proposed solution is a layered architecture running on a Raspberry Pi 5:

### 4.1 Hardware Layer
- Raspberry Pi 5 with 8GB RAM and a 32GB+ A2-class microSD card.
- Velbus modules (dimmers, relays, panels, motion sensor) connected to the Pi via the VMB1USB interface.

### 4.2 Operating System Layer
- Home Assistant OS providing the core platform, Supervisor, and Apps store.

### 4.3 Add-on Layer
- Mosquitto MQTT Broker for inter-device messaging.
- Wyoming Whisper for speech-to-text.
- Wyoming Piper for text-to-speech.
- Node-RED for visual automation flow building.

### 4.4 Integration Layer
- Velbus integration mapping every channel to a Home Assistant entity.
- Wyoming Protocol integrations connecting Whisper and Piper to the Assist pipeline.
- OpenRouter integration providing the LLM conversation agent (with local Ollama as a future fallback).

### 4.5 Logic Layer
- Home Assistant Assist pipeline (MDAR Assistant) orchestrating voice input, LLM, and voice output.
- Node-RED flows handling event-driven automation (motion lighting, scheduled scenes, MQTT reports).

### 4.6 User Interface Layer
- Lovelace dashboard with high-contrast Flash Cards for non-verbal interaction.
- TV-overlay notifications for important system messages.
- Voice interaction through the browser microphone (with future hardware microphone support).

---

## 5. Sprint Planning Rationale

The 15-week project was divided into three sprints based on the COM617 Project Milestones, with an additional Bridging Research phase:

- **Sprint 1 (Weeks 1–3) — Research & Discovery.** Three weeks is sufficient to research the problem space, evaluate technology options, and produce a concrete solution proposal. Less time would risk picking the wrong technology; more time would delay the practical work.
- **Sprint 2 (Weeks 4–7) — Proof of Concept.** Four weeks to install the OS, integrate the hardware, and prove that voice control of a single light is possible. This is the riskiest phase because it depends on external hardware and integration compatibility.
- **Bridging Research (between Sprints).** Allows team members to deepen their knowledge of areas they will lead in Sprint 3 without blocking each other's progress.
- **Sprint 3 (Weeks 8–15) — MVP.** The longest sprint because it covers AI integration, automation flows, MQTT messaging, dashboard design, testing, and final documentation. Eight weeks gives the team room to handle the unexpected complexity that always arises in integration projects.

---

## 6. Role Allocation

### Partner A — Integrator
- Pi 5 hardware configuration.
- Velbus module programming via VelbusLink.
- Node-RED automation flow development.
- Voice pipeline integration.

### Partner B — Researcher
- Accessibility research and persona development.
- Requirement documentation.
- MQTT logic design and message structure.
- AI model evaluation.

### Partner C — Project Coordinator
- Sprint planning and milestone tracking against COM617 deadlines.
- Maintains the master Risk Register and resolves blockers.
- Coordinates communication between team members and the supporting tutor/sponsor.
- Leads preparation for the AE2 Pitch and Presentation.
- Tracks individual contributions to ensure fair workload distribution.

---

## 7. Approval

The solution proposal, persona, requirements, and sprint plan were presented to the supporting tutor at the end of Week 3 and approved without major revisions. The project then proceeded into Sprint 2 with a clear technical baseline and divided responsibilities.
