# Sprint 1 — Document 1: Project Foundation & Requirements Gathering

## 1. Purpose

This document establishes the foundational concepts that underpin the MDAR VELBUS Voice-Assistive System. Before any technology was selected or built, the team needed a shared understanding of the underlying domains: the Internet of Things, voice processing, the Raspberry Pi platform, and the Velbus ecosystem. This document captures that initial knowledge baseline and defines the high-level requirements for the project.

---

## 2. Foundational Concepts

### 2.1 Internet of Things (IoT)

The Internet of Things describes the network of physical devices, vehicles, appliances, and sensors that connect, exchange data, and respond to commands over a network. In a smart home context, IoT enables devices like lights, thermostats, locks, and sensors to communicate with each other and with a central controller, allowing automated and remote operation.

The MDAR project is fundamentally an IoT project — every Velbus module, the Raspberry Pi controller, and the cloud-hosted AI agent are nodes in a connected system that exchanges state information and commands.

### 2.2 Speech-to-Text (STT)

Speech-to-Text is the process of converting spoken language into written text using a machine learning model. STT is a critical component of any voice assistant because it forms the input layer — the assistant cannot act on a command until that command has been transcribed into text. Modern STT systems use deep neural networks trained on large speech datasets to achieve high accuracy across accents, noise levels, and speaking styles.

For this project, **Whisper** (specifically the `faster-whisper` variant) was selected as the STT engine because it can run locally without cloud dependency, supports British English accents well, and is integrated into Home Assistant via the Wyoming Protocol.

### 2.3 Text-to-Speech (TTS)

Text-to-Speech is the inverse process — converting written text into spoken audio. TTS is the output layer of a voice assistant: once the system has determined a response, TTS generates the audio that the user hears. Modern neural TTS systems can produce natural, human-sounding speech in a variety of voices and accents.

For this project, **Piper** was selected as the TTS engine. Piper is a lightweight neural TTS that runs locally, supports multiple voices, and integrates with Home Assistant via the same Wyoming Protocol used by Whisper. The voice model `en_GB-alan-medium` was chosen to match the British English target audience.

### 2.4 Raspberry Pi

The Raspberry Pi is a series of small, affordable single-board computers developed by the Raspberry Pi Foundation. It runs a full Linux operating system on ARM hardware, has GPIO pins for hardware control, and is widely used in embedded, educational, and home automation projects.

The **Raspberry Pi 5** was selected for this project because it offers significantly improved CPU and GPU performance over previous generations, supports up to 16GB of RAM, has built-in dual-band WiFi and Bluetooth, and is well-supported by Home Assistant OS. Its performance is sufficient to host the operating system, the Wyoming voice services, and the Velbus integration simultaneously.

### 2.5 Velbus Ecosystem

Velbus is a Belgian-designed home automation system manufactured by Velleman. It uses a wired bus topology (similar to KNX) where modules communicate over a 4-wire bus carrying both power and data. Velbus modules include dimmers, relays, push-button panels, motion sensors, temperature sensors, and more.

Two related products are central to this project:

- **The Velbus kit** — the physical hardware: dimmer modules (VMB4DC), relay modules (VMB1RYNO, VMB4RYLD), the glass push-button panel (VMBGPOD-2), the PIR motion sensor (VMBPIRM), and a USB interface (VMB1USB) that connects the bus to a computer.
- **VelbusLink** — the Windows-only configuration software used to scan the bus, name channels, configure module behaviour, and define button-to-output actions. VelbusLink writes configuration data directly to the module firmware over the bus.

Velbus was chosen because it is robust, widely used in Belgian residential and commercial buildings, and has a mature Home Assistant integration that exposes every channel as a controllable entity.

---

## 3. High-Level Requirements

The system must meet the following high-level requirements, derived from the project brief and the user persona "Arthur":

1. The system must be controllable through natural-language voice commands.
2. Voice processing must work entirely locally where possible to protect user privacy.
3. The system must integrate with Velbus hardware to control real lighting and electrical loads.
4. The system must support background automation (motion-triggered lighting, scheduled scenes).
5. The system must be able to publish device state reports to a remote landlord/operator over MQTT.
6. The system must be accessible to users with mobility or visual impairments.

These requirements informed every subsequent technology choice, from the operating system through to the AI conversation agent.

---

## 4. Stakeholder Context

The project sits within the broader context of assistive technology for residential buildings. The primary stakeholder is the end user (represented by the persona "Arthur") — a person who would benefit from voice and automated control of their home environment. The secondary stakeholder is the building manager or landlord, who needs visibility into device states and energy usage across multiple properties.

The system must therefore serve two interfaces simultaneously: a conversational, accessible interface for the resident, and a structured, programmatic interface (via MQTT) for the landlord.

---

## 5. Conclusion

This document establishes the conceptual foundation upon which the rest of the project is built. With a shared understanding of IoT, STT, TTS, the Raspberry Pi platform, and the Velbus ecosystem, the team was equipped to evaluate specific technologies (covered in Document 2) and propose a concrete solution architecture (Document 3).
