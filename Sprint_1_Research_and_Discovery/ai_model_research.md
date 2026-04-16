

# Technical Research Report: Local Voice Intelligence

## 1. Research Overview

This research investigates the deployment of **OpenAI Whisper** and **Piper TTS** as a localized, autonomous alternative to cloud-based assistants. The objective is to provide a reliable voice interface for visually and hearing-impaired users within a **Velbus** building automation ecosystem.

## 2. Speech-to-Text (STT): OpenAI Whisper

Ressource : https://github.com/openai/whisper

Whisper is a multi-purpose speech recognition model. For the **Raspberry Pi 5**, we analyzed the trade-offs between accuracy and latency across different model sizes.

### 2.1 Performance Analysis

| Model Variant | Parameters | RAM Usage | Pi 5 Speed | Accuracy Context |
| --- | --- | --- | --- | --- |
| **Tiny.en** | 39M | ~150MB | 10x (Real-time) | Best for simple commands (e.g., "Lights On"). |
| **Base.en** | **74M** | **~300MB** | **7x (Near-instant)** | **Recommended:** Handles complex sentences well. |
| **Small.en** | 244M | ~1GB | 2x (Delayed) | High accuracy but risks 3–5s latency. |

### 2.2 Local Constraints

* **NPU Acceleration:** On the Pi 5, Whisper can leverage the **Hailo AI NPU** (via the AI HAT+) to achieve Gen-AI speeds, though CPU-only inference using `whisper.cpp` remains viable for the `base.en` model.
* **Autonomy:** By utilizing the **Wyoming Protocol**, audio streams are processed in RAM and never written to permanent storage, satisfying the strict privacy requirements for social housing and managed offices.

## 3. Text-to-Speech (TTS): Piper

Ressource : https://github.com/OHF-Voice/piper1-gpl

Piper is a fast, local neural TTS system that uses **ONNX** models to generate speech.

### 3.1 Voice Architecture

Piper utilizes a VITS-based architecture to provide human-like prosody. For accessibility, we have selected the **Medium** quality tier.

* **Clarity:** 22.05 kHz sample rate (Medium) provides enough fidelity for visually impaired users to distinguish system status alerts from background noise.
* **Latency:** Piper can begin speaking before the full sentence is finished (streaming), reducing perceived delay to under **0.5 seconds** on a Pi 5.

## 4. System Integration: The Wyoming Protocol

The **Wyoming Protocol** is the communication standard that links Home Assistant to these AI engines.

* **Unified Interface:** It allows the same voice "Satellite" (the Pi 5) to handle Wake Word detection, STT, and TTS through independent services.
* **Scalability:** Landlords can manage multiple "Units" by pointing different Wyoming satellites to a central broker, ensuring the system remains responsive as the building complex grows.

## 5. Accessibility & Human Factors

* **Non-Verbal Support:** For users who cannot speak, the system uses Piper to "vocalize" pre-set phrases triggered by physical buttons or dashboard "flash cards."
* **Status Feedback:** The system is configured to automatically announce state changes (e.g., *"The living room temperature is now 21 degrees"*) to provide situational awareness to visually impaired tenants.

---

### **Conclusion**

The combination of **Whisper Base** and **Piper Medium** on a **Raspberry Pi 5** provides a robust, professional-grade accessibility solution. This stack fulfills the project’s "Autonomous" requirement while providing a user experience that rivals cloud-based competitors.

This [tutorial on setting up local voice in Home Assistant](https://www.youtube.com/watch?v=1eZJPhR30ek) is directly relevant as it demonstrates the exact configuration of Whisper, Piper, and the Wyoming protocol for the Raspberry Pi.
