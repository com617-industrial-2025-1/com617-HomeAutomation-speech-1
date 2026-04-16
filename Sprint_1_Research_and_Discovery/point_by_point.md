

# Point-by-Point Statement of Requirements

## 1. Functional Requirements (What the system MUST do)

* **FR-1: Local Voice Command Processing:** The system **shall** process spoken commands (e.g., "Turn on the light") using the Whisper `base.en` model locally on the Raspberry Pi 5.
* **FR-2: Real-Time Audio Feedback:** On any state change of a Velbus module, the system **shall** trigger a Text-to-Speech (TTS) announcement via Piper describing the change (e.g., "Heating is now ON").
* **FR-3: Environmental Status Inquiry:** The system **shall** respond verbally to user queries regarding the current temperature or device status based on Velbus sensor data.
* **FR-4: Non-Verbal Dashboard (Flash Cards):** The Home Assistant interface **shall** include a dedicated "Flash Card" dashboard with high-contrast buttons for users with vocal fatigue or hearing impairments.
* **FR-5: Centralized Landlord Logging:** The system **shall** generate and publish an MQTT message for every device actuation or state change to allow for remote landlord monitoring.

## 2. Technical & Constraint Requirements (How it must work)

* **TR-1: Autonomous Operation:** The system **shall** operate entirely without external cloud processing (e.g., No Siri, Alexa, or Google Cloud) to ensure privacy and reliability.
* **TR-2: Velbus Protocol Integration:** The system **shall** communicate with Velbus hardware using the 4-wire CAN bus protocol via a USB-to-TCP bridge.
* **TR-3: Response Latency:** The total time from a voice command's end to the Velbus physical action **shall** not exceed 2.0 seconds.
* **TR-4: Local Networking:** The system **shall** utilize the Wyoming Protocol to link Whisper and Piper to the Home Assistant core locally.

## 3. Accessibility & Usability Requirements

* **UR-1: High-Contrast Interface:** The digital dashboard **shall** follow WCAG 2.1 accessibility guidelines for color contrast to assist visually impaired users like Arthur.
* **UR-2: Accented Intelligibility:** The TTS engine **shall** utilize a British English (en-GB) voice model to ensure familiarity for users in the UK.

