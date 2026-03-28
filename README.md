# com617-HomeAutomation-speech-1


# Realistic Project Plan: MDAR VELBUS Voice-Assistive System

## 1. Project Overview

This project is divided into three distinct sprints, following the **COM617 Project Milestones**. The goal is to move from theoretical requirements to a fully functional Minimal Viable Product (MVP) for assistive building automation.

---

## 2. Sprint Schedule

### **Sprint 1: Gather Requirements (Weeks 1-3)**

**Goal:** Define the technical scope and human needs.

* **Week 1:** Establish GitHub repository and conduct AI research (Whisper vs. Piper).
* **Week 2:** Create the "Arthur" User Persona and draft the Point-by-Point Statement of Requirements.
* **Week 3:** Finalize the Outline Solution Proposal and present to the supporting tutor.

### **Sprint 2: Initial Proof of Concept (Weeks 4-7)**

**Goal:** Achieve basic hardware-software connectivity.

* **Week 4:** Installation of Home Assistant OS and Wyoming Protocol on the Raspberry Pi 5.
* **Week 5:** Integrate Velbus hardware via USB/TCP and establish "Hello World" (one light toggled via voice).
* **Week 6:** Initial design of the MQTT reporting logic and Node-RED flows.
* **Week 7:** Demonstrate PoC and initial design documentation to the project tutor and sponsor.

### **Sprint 3: Minimal Viable Product (Weeks 8-15)**

**Goal:** Finalize the accessibility features and system evaluation.

* **Week 8-9:** Develop the high-contrast "Flash Card" dashboard for non-verbal interaction.
* **Week 10-11:** Implement status inquiry logic (e.g., "Tell me the temperature") and TV-Overlay notifications.
* **Week 12:** Rigorous testing against the Point-by-Point Requirements (Latency and Accuracy tests).
* **Week 13:** Complete the final Project Initiation Document (PID) and Evaluation of Solution.
* **Week 14:** Final polish and preparation for the AE2 Pitch/Presentation.
* **Week 15:** Submission of AE1 (PID) and delivery of AE2 Pitch.

---

## 3. Role Allocation

* **Researcher (Partner B):** Responsible for accessibility research, persona development, requirement documentation, and MQTT logic.
* **Integrator (Partner A):** Responsible for Pi 5 hardware configuration, Velbus module programming, and Node-RED automation.

---

## 4. Identified Issues Needing Resolved (Risk Register)

1. **Audio Hardware:** Determining if the Raspberry Pi 5 internal audio or a specialized I2S microphone HAT is needed for clear STT in a lab environment.
2. **Network Autonomy:** Ensuring the Pi 5 can handle local speech processing while simultaneously managing MQTT traffic without latency spikes.
3. **Velbus Link Access:** Obtaining the correct module addresses from the VelbusLink configuration tool in the university lab.

---
