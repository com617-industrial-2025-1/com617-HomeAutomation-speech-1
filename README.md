# MDAR VELBUS Voice-Assistive System

## Introduction

A voice-controlled, AI-enhanced, locally-running smart home system built on Home Assistant OS, Velbus hardware, and Node-RED automation flows. Designed for accessibility — enabling natural language control of lighting and background automation without cloud dependency.

See [Project Brief](./Sprint_1_Research_and_Discovery/ProjectBrief.md)

---

## Technology Stack

| Component | Technology |
|-----------|-----------|
| Platform | Home Assistant OS 17.1 on Raspberry Pi 5 |
| Hardware Protocol | Velbus (VMB4DC, VMB4RYLD, VMBPIRM, VMB7IN) |
| Speech-to-Text | Wyoming Whisper (base-en model) |
| Text-to-Speech | Wyoming Piper (en_GB-alan-medium) |
| LLM Conversation Agent | GPT-4o-mini via OpenRouter |
| Automation Layer | Node-RED |
| Messaging | Mosquitto MQTT Broker |

---

## Team Roles

| Role | Responsibilities |
|------|-----------------|
| Integrator | Hardware setup, Velbus programming, Node-RED flows, voice pipeline |
| Researcher | Accessibility research, requirements, AI model evaluation, MQTT design |
| Coordinator | Sprint planning, risk register, tutor liaison, AE2 pitch preparation |

---

## Project Structure

```
/
├── Project_Plan/                          — Master project plan and sprint schedule
├── Sprint_1_Research_and_Discovery/       — Foundations, technology research, solution proposal
├── Sprint_2_Proof_of_Concept/             — OS setup, Velbus integration, voice pipeline PoC
├── Bridging_Research/                     — AI agent research and automation architecture decisions
├── Sprint_3_MVP/                          — LLM integration, Node-RED flows, MQTT, final architecture
└── workup/                                — Exploratory VLM research (out of scope for MVP)
```

---

## Document Index

### Project Plan
- [PROJECT_PLAN.md](./Project_Plan/PROJECT_PLAN.md) — Full 15-week plan, role allocation, risk register

### Sprint 1 — Research & Discovery
- [01 Foundation and Requirements](./Sprint_1_Research_and_Discovery/01_Foundation_and_Requirements.md)
- [02 Technology Research](./Sprint_1_Research_and_Discovery/02_Technology_Research.md)
- [03 Solution Proposal](./Sprint_1_Research_and_Discovery/03_Solution_Proposal.md)
- [Point-by-Point Requirements](./Sprint_1_Research_and_Discovery/point_by_point.md)
- [AI Model Research](./Sprint_1_Research_and_Discovery/ai_model_research.md)

### Sprint 2 — Proof of Concept
- [01 OS and Platform Setup](./Sprint_2_Proof_of_Concept/01_OS_and_Platform_Setup.md)
- [02 Velbus Integration](./Sprint_2_Proof_of_Concept/02_Velbus_Integration.md)
- [03 Voice Pipeline](./Sprint_2_Proof_of_Concept/03_Voice_Pipeline.md)

### Bridging Research
- [01 AI Conversation Agent Research](./Bridging_Research/01_AI_Conversation_Agent_Research.md)
- [02 Node-RED vs Voice Architectural Decision](./Bridging_Research/02_NodeRED_vs_Voice_Architectural_Decision.md)

### Sprint 3 — MVP
- [01 AI Integration and Model Selection](./Sprint_3_MVP/01_AI_Integration_and_Model_Selection.md)
- [02 Node-RED Automation Implementation](./Sprint_3_MVP/02_NodeRED_Automation_Implementation.md)
- [03 MQTT Messaging Layer](./Sprint_3_MVP/03_MQTT_Messaging_Layer.md)
- [04 Final System Architecture](./Sprint_3_MVP/04_Final_System_Architecture.md) ← current system state
