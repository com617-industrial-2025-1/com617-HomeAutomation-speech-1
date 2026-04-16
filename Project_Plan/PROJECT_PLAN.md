# MDAR VELBUS Voice-Assistive System — Project Plan

## 1. Project Overview

This project is divided into three distinct sprints, following the **COM617 Project Milestones**, with an **Bridging Research phase** between Sprint 2 and Sprint 3. The goal is to move from theoretical requirements to a fully functional Minimum Viable Product (MVP) for assistive building automation.

The end deliverable is a voice-controlled, AI-enhanced, locally-running smart home system with automation flows that can be controlled both interactively (via natural language) and autonomously (via event-driven triggers).

---

## 2. Sprint Schedule

### Sprint 1: Research & Discovery (Weeks 1–3)

**Goal:** Define the technical scope, understand the underlying technologies, and document the human needs.

- **Week 1:** Establish the GitHub repository. Define foundational concepts: IoT, STT, TTS, what a Raspberry Pi is, what Velbus is (link + kit). Research Raspberry Pi capabilities and OS options.
- **Week 2:** Evaluate Home Assistant against alternative platforms. Research and compare Whisper and Piper. Review GitHub repositories for TTS/STT solutions. Begin drafting the "Arthur" user persona.
- **Week 3:** Finalise the Outline Solution Proposal. Complete the Point-by-Point Statement of Requirements. Assign roles. Present to the supporting tutor.

### Sprint 2: Proof of Concept (Weeks 4–7)

**Goal:** Achieve basic hardware-to-software connectivity and demonstrate a working voice pipeline.

- **Week 4:** Install Home Assistant OS on the Raspberry Pi 5. Set up Internet Connection Sharing from the laptop. Complete onboarding. Install the Wyoming Whisper and Piper add-ons.
- **Week 5:** Connect Velbus hardware via USB. Program the modules in VelbusLink (channel naming and button-to-output actions). Integrate the Velbus bus with Home Assistant. Resolve entity discovery issues.
- **Week 6:** Build the MDAR Assistant voice pipeline. Expose entities. Resolve browser microphone permission issues. Achieve "Hello World" — toggle a single light entirely via voice.
- **Week 7:** Document the weaknesses of the basic pipeline (latency, basic command matching, lack of natural language understanding). Demonstrate the PoC to the tutor and sponsor.

### Bridging Research Phase (between Sprint 2 and Sprint 3)

**Goal:** Bridge the gap between a basic PoC and a polished MVP through targeted research.

- Research AI conversation agents (OpenAI, OpenRouter, local Ollama).
- Identify the difference between conversational AI (chat-based) and event-driven automation (Node-RED).
- Research code patterns and node types in Node-RED.
- Identify MQTT as the messaging layer for landlord reporting.
- Plan the automation flows that will be implemented in Sprint 3.

### Sprint 3: Minimum Viable Product (Weeks 8–15)

**Goal:** Finalise the accessibility features, complete the full automation pipeline, and prepare the system for evaluation.

- **Week 8:** Implement the OpenRouter integration. Test multiple LLM models (Claude 3 Haiku, GPT-4o-mini, Gemini). Test local Ollama with llama3.1:8b and dolphin-llama3 on a laptop GPU. Settle on GPT-4o-mini as the production model for reliability and cost.
- **Week 9:** Install and configure Node-RED. Resolve the SSL startup issue. Connect Node-RED to Home Assistant using a Long-Lived Access Token.
- **Week 10:** Build the automation flows: Motion-Triggered Lighting, All Lights On, All Lights Off, Night Mode (auto-dim and restore), MQTT Landlord Reporting.
- **Week 11:** Configure the Mosquitto MQTT broker. Design the JSON report structure. Test the publish/subscribe pipeline end to end.
- **Week 12:** Develop a high-contrast "Flash Card" dashboard for non-verbal interaction. Implement status inquiry logic ("what is the temperature"). Add TV-overlay notifications.
- **Week 13:** Conduct rigorous testing — latency benchmarks, accuracy tests, full pipeline end-to-end against requirements.
- **Week 14:** Complete the Project Initiation Document (PID) and the Evaluation of Solution. Finalise documentation across all sprint folders.
- **Week 15:** Submit AE1 (PID). Deliver the AE2 Pitch and Presentation.

---

## 3. Role Allocation

### Partner A — Integrator
- Raspberry Pi 5 hardware configuration and OS setup.
- Velbus module programming via VelbusLink (channel naming, action mapping, synchronisation).
- Node-RED automation flow development.
- Voice pipeline integration (Whisper + Piper + LLM conversation agent).

### Partner B — Researcher
- Accessibility research and user persona development.
- Point-by-Point Statement of Requirements.
- MQTT logic design and message structure.
- AI model evaluation (cloud vs local, cost vs reliability).

### Partner C — Project Coordinator
- Sprint planning and milestone tracking against COM617 deadlines.
- Maintains the master Risk Register and resolves blockers.
- Coordinates communication between team members and the supporting tutor/sponsor.
- Leads preparation for the AE2 Pitch and Presentation.
- Tracks individual contributions to ensure fair workload distribution.

---

## 4. Risk Register

### Resolved during Sprint 2

| Risk | Resolution |
|------|------------|
| Add-ons menu renamed to "Apps" in Home Assistant 2025+ | Identified the new menu location and documented the change |
| Browser microphone blocked on HTTP origins | Used Chrome flag `unsafely-treat-insecure-origin-as-secure` to allow microphone on the local HA URL |
| Velbus modules showed no entities after first integration | Programmed channels in VelbusLink and re-added the integration to force a fresh entity scan |
| Wyoming Protocol connection failed with empty fields | Manually entered `core-whisper:10300` and `core-piper:10200` |
| Helper toggles conflicted with real Velbus entities in Assist | Deleted the simulated helpers after real hardware was connected |

### Resolved during Sprint 3

| Risk | Resolution |
|------|------------|
| Claude models on OpenRouter had tool-calling errors (`ToolResultContent`) | Switched to GPT-4o-mini, which handles HA tool calling reliably |
| Local Ollama too slow on Raspberry Pi (40-second response times) | Moved Ollama to laptop with NVIDIA RTX 3070 Ti GPU. Tool calling still unreliable, so kept GPT-4o-mini as the production agent |
| Node-RED add-on failed to start | Disabled SSL in the add-on configuration (`ssl: false`) since the system runs on a trusted local network |
| Node-RED nodes showed "no connection" to Home Assistant | Generated a Long-Lived Access Token from the HA user profile and pasted it into the Node-RED server config |
| VelbusLink truncated channel names | Accepted the truncated names in VelbusLink and renamed the entities in Home Assistant where there is no character limit |

### Ongoing / Monitored

1. **Audio Hardware:** The Pi 5's internal audio versus a dedicated I2S microphone HAT may affect STT quality in noisy environments. To be evaluated during Sprint 3 testing.
2. **Network Autonomy:** The current production pipeline depends on OpenRouter (cloud LLM). If the internet drops, voice control degrades to basic Assist matching. A future improvement is a more capable local LLM running on dedicated hardware.
3. **VelbusLink Lab Access:** Module addresses must be obtained from the VelbusLink configuration tool in the university lab. Access is dependent on lab availability.

---

## 5. Documentation Structure

The complete documentation is organised into four folders, each containing dedicated markdown documents:

- **Sprint 1 — Research & Discovery:** 3 documents
- **Sprint 2 — Proof of Concept:** 3 documents
- **Bridging Research:** 2 documents
- **Sprint 3 — Minimum Viable Product:** 4 documents

See the root [`README.md`](../README.md) for the full folder index.
