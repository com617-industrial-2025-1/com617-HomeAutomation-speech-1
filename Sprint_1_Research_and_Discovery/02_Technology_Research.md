# Sprint 1 — Document 2: Technology Research & Framework Selection

## 1. Purpose

With the foundational concepts established in Document 1, the next step was to evaluate specific technologies and select a framework that could support the project requirements. This document covers the operating system evaluation, the smart home platform comparison, the deep dive on Whisper and Piper, and the GitHub repository review for voice processing solutions.

---

## 2. Operating System Evaluation

The choice of operating system for the Raspberry Pi was the first major decision. Three options were considered:

### 2.1 Raspberry Pi OS

The default OS for the Raspberry Pi, based on Debian Linux. It offers a familiar desktop environment and full general-purpose Linux capability. However, it would require manual installation and configuration of every smart home component, with no built-in framework for device discovery, automation, or voice control.

### 2.2 Home Assistant OS (HAOS)

A purpose-built operating system that runs Home Assistant Core along with the Supervisor service, an integrated add-on store, and built-in backup management. HAOS replaces a general-purpose OS entirely — it boots directly into Home Assistant.

### 2.3 Home Assistant Container

Home Assistant Core running as a Docker container on top of a separate host OS. This option offers more flexibility but lacks the integrated add-on store and Supervisor features that simplify smart home setup.

### 2.4 Decision

**Home Assistant OS** was selected because:

- It includes the integrated Apps store (formerly called "Add-ons") that makes installing Whisper, Piper, Node-RED, and Mosquitto a one-click operation.
- It manages backups, updates, and recovery automatically through the Supervisor.
- It is the recommended platform for new Home Assistant deployments.
- It is the most beginner-friendly while still being production-capable.

---

## 3. Smart Home Platform Comparison

Two open-source smart home platforms were evaluated:

### 3.1 Home Assistant

A Python-based open-source home automation platform with:

- Native Wyoming Protocol support for Whisper and Piper.
- A built-in Assist conversational engine for command parsing.
- A drag-and-drop dashboard editor (Lovelace).
- Native Velbus integration with full device discovery.
- A large community add-on ecosystem.
- Lighter resource usage on a Raspberry Pi 5.

### 3.2 openHAB

A Java-based open-source home automation platform. It supports Velbus through an official binding and has MQTT capability, but it lacks:

- Native voice pipeline support equivalent to the Wyoming Protocol.
- A built-in intent recognition engine equivalent to Assist.
- A dashboard as accessible as Lovelace for non-technical users.

openHAB's Java runtime also has higher memory overhead, leaving less headroom on a Pi 5 for local AI processing.

### 3.3 Decision

**Home Assistant** was selected as the platform. The deciding factor was the native Wyoming Protocol support, which makes the integration of Whisper and Piper into a unified voice pipeline trivial. On openHAB, the same pipeline would need to be wired together manually, which would consume significant project time without any functional benefit.

The project brief itself also explicitly references "Home Assistant with embedded Node-RED as a starting point", which confirmed the choice.

---

## 4. Voice Pipeline Research

### 4.1 Whisper

Whisper is an open-source automatic speech recognition system originally developed by OpenAI. It is trained on a large multilingual dataset and offers near state-of-the-art accuracy across many languages. Several variants exist:

- **OpenAI Whisper** — the original Python implementation.
- **whisper.cpp** — a C++ port optimised for CPU inference.
- **faster-whisper** — a CTranslate2-based implementation that is significantly faster than the original.

For this project, **faster-whisper** was selected because it is the variant packaged in the official Wyoming Whisper add-on for Home Assistant. It runs efficiently on a Raspberry Pi 5 with the `base-en` model, providing accurate English transcription with acceptable latency.

### 4.2 Piper

Piper is a fast, local neural text-to-speech engine designed specifically for use on small devices like the Raspberry Pi. It supports a wide range of voices in many languages and can produce natural-sounding speech with sub-second latency.

The voice model `en_GB-alan-medium` was selected because:

- It is a British English voice, matching the target audience.
- The "medium" quality offers a good balance between naturalness and inference speed.
- It is included in the official Wyoming Piper add-on.

### 4.3 Wyoming Protocol

The Wyoming Protocol is a simple TCP-based protocol developed by the Home Assistant project to standardise the interface between Home Assistant's Assist pipeline and external voice services. By exposing Whisper and Piper as Wyoming Protocol services, Home Assistant can route audio in and out of them as part of a unified voice pipeline.

This abstraction is what made Home Assistant the obvious platform choice — Whisper, Piper, and the conversation agent all plug into the same standardised pipeline.

---

## 5. GitHub Repository Review

The team reviewed several open-source repositories for voice processing and Home Assistant integration:

| Repository | Purpose | Verdict |
|------------|---------|---------|
| `openai/whisper` | Original Whisper implementation | Reference implementation; not used directly |
| `guillaumekln/faster-whisper` | Optimised Whisper variant | Adopted via the Wyoming Whisper add-on |
| `rhasspy/piper` | Piper TTS engine | Adopted via the Wyoming Piper add-on |
| `home-assistant/wyoming` | Wyoming Protocol implementation | Used implicitly through HA add-ons |
| `acon96/home-llm` | Local LLM integration for HA | Reviewed for Sprint 3 — local Ollama path |
| `jekalmin/extended_openai_conversation` | OpenAI-compatible conversation agent | Considered but not adopted |

The review confirmed that the open-source ecosystem around Home Assistant is mature enough to support the project without writing low-level voice processing code from scratch. The team would integrate existing components rather than build new ones.

---

## 6. Conclusion

The technology stack for the project was finalised at the end of Sprint 1:

- **OS:** Home Assistant OS on Raspberry Pi 5
- **Platform:** Home Assistant (with embedded Node-RED, Mosquitto, and Wyoming services)
- **STT:** Wyoming Whisper (faster-whisper, base-en model)
- **TTS:** Wyoming Piper (en_GB-alan-medium voice)
- **Hardware Bus:** Velbus via VMB1USB serial interface

These choices were validated against the high-level requirements from Document 1 and form the technical baseline that Sprint 2 would build on.
