# Sprint 2 — Document 2: Velbus Hardware Integration

## 1. Purpose

This document covers the integration of the Velbus hardware bus into the Home Assistant platform. It describes the initial integration attempt, the discovery that Velbus modules require firmware-level programming before they expose entities, the use of VelbusLink to configure channels and actions, and the final reintegration into Home Assistant. This was the most technically complex phase of Sprint 2 and produced a number of documented troubleshooting outcomes.

---

## 2. Hardware Inventory

The Velbus kit used in this project consists of eight modules connected over a single bus, communicating through the VMB1USB serial interface:

| Module | Address (Hex) | Type | Function |
|--------|---------------|------|----------|
| VMBPIRM | 01 | PIR Motion Sensor | Motion detection with separate motion zones, dark/light sensing, and absence detection |
| VMB4DC | 02 | 4-Channel Dimmer | Controls dimmable lighting circuits |
| VMBGPOD-2 | 03 | Glass Push-Button Panel (OLED) | Wall-mounted touch panel with 8 buttons and a temperature sensor |
| VMBEL2 | 04 | 2-Channel Push-Button Module | Additional button inputs |
| VMBELO | 05 | Edge-Lit Push-Button Panel | Push-button inputs with LED feedback |
| VMB1RYNO | 06 | 1-Channel Relay | On/off switching for non-dimmable loads |
| VMB4RYLD | 07 | 4-Channel Relay | Multiple relay outputs for spotlights and other devices |
| VMB7IN | 08 | 7-Channel Input Module | Multi-purpose input module |

---

## 3. Initial Integration Attempt

The Velbus integration was added in Home Assistant via:

**Settings → Devices & Services → Add Integration → Velbus**

The serial port `/dev/ttyACM0` was selected (identified as `VMB1USB Velbus USB interface - Velleman Projects`). Home Assistant successfully discovered all 8 modules on the bus and reported a total of 199 entities.

However, when inspecting the individual devices (especially the VMB4DC dimmer and VMB1RYNO relay), the team found that the **Controls** section showed `This device has no entities`. The hardware was visible, but no controllable entities had been created. Without entities, neither the dashboard nor the voice assistant could control the lights.

---

## 4. Diagnosis: Modules Need Firmware Programming

Investigation revealed that Home Assistant's Velbus integration only creates entities for **channels that have been enabled and configured** in the module firmware. A freshly-installed module is in a default state where its channels are disabled. To enable them, the modules must be programmed using **VelbusLink** — Velleman's official Windows-only configuration tool.

This is a deliberate design choice in the Velbus ecosystem: the modules are designed to operate independently of any host computer (button-to-output actions are stored on the module firmware itself), so the host integration only sees what the module is configured to expose.

---

## 5. VelbusLink Configuration

The team disconnected the VMB1USB from the Pi and connected it to a Windows laptop running **VelbusLink 11.9.7**. The setup process inside VelbusLink was:

1. Created a new project file (`MyProject.vlp`).
2. Clicked **Connect** and selected the COM port for the VMB1USB (initially COM7).
3. Performed a **Scan** of the bus, which discovered all 8 modules.
4. Confirmed the bus status changed from "Working offline" to "Connected" and "Bus Active/Receive Ready".

### 5.1 Channel Configuration on the VMB4DC

The 4-channel dimmer was configured to control four lights:

- Channel 1 → renamed to **living room ligh** (truncated due to VelbusLink's character limit)
- Channel 2 → renamed to **kitchen light**
- Channel 3 → renamed to **toilet light**
- Channel 4 → renamed to **bedroom light**

Each channel was enabled and named within the module firmware itself.

### 5.2 Action Configuration

To allow the physical glass panel buttons on the VMBGPOD-2 to control the dimmer channels, four actions were created in VelbusLink:

| Initiator (Button) | Action | Subject (Output) |
|-------------------|--------|------------------|
| VMBGPOD-2 Push Button 1 | Dim at long press, toggle at short press (0202) | VMB4DC living room ligh (CH1) |
| VMBGPOD-2 Push Button 2 | Dim at long press, toggle at short press (0202) | VMB4DC kitchen light (CH2) |
| VMBGPOD-2 Push Button 3 | Dim at long press, toggle at short press (0202) | VMB4DC toilet light (CH3) |
| VMBGPOD-2 Push Button 4 | Dim at long press, toggle at short press (0202) | VMB4DC bedroom light (CH4) |

The same pattern was applied to the VMB4RYLD relay module, mapping push buttons 5–8 to the four spotlight relay channels using the simpler "Toggle" (0103) action.

### 5.3 Synchronisation

After configuration, the team ran **Synchronize → Write to modules** to upload the configuration to the module firmware. The project was saved to disk for future reference.

A common issue encountered during synchronisation was the "Buffer full" warning, which appears when VelbusLink sends too many requests too quickly. The fix was to wait approximately 30 seconds and retry — the bus throttles itself and the operation completes.

---

## 6. Reintegration into Home Assistant

With the modules now programmed, the VMB1USB was reconnected to the Pi. The next step was to refresh the Home Assistant integration so it would re-scan the modules and pick up the newly enabled channels.

The team chose to **delete and re-add** the Velbus integration rather than rely on a reload, because deletion forces a full re-discovery. After re-adding the integration with the same `/dev/ttyACM0` port, the dimmer and relay modules now exposed their configured channels as `light` and `switch` entities respectively.

Examples of the entity IDs created:

- `light.livingroom_light_livingroom_light` (Living Room Light dimmer)
- `light.livingroom_light2_livingroom_light` (Kitchen Light dimmer)
- `light.livingroom_light3_livingroom_light` (Toilet Light dimmer)
- `light.livingroom_light4_livingroom_light` (Bedroom Light dimmer)
- `switch.livingroom_spot_livingroom_spot` (Living Room Spot relay)
- `switch.kitchen_spot_kitchen_spot` (Kitchen Spot relay)
- `switch.bedroom_spot_bedroom_spot` (Bedroom Spot relay)
- `switch.toilet_spot_toilet_spot` (Toilet Spot relay)
- `binary_sensor.vmbpirm_motion_output_1` (PIR motion sensor)

---

## 7. Entity Configuration in Home Assistant

Each entity was then refined in Home Assistant:

- **Renamed** to a clean, human-readable name (e.g., "Living Room Light", "Kitchen Light"). Home Assistant has no character limit, so the truncated VelbusLink names were corrected here.
- **Assigned to an Area** matching the room where the device is physically located (Living Room, Kitchen, Bedroom, Toilet). Areas were created via Settings → Areas, labels & zones.
- **Exposed to Assist** via Settings → Voice Assistants → Expose, with appropriate aliases added for voice matching.

The motion sensor (`binary_sensor.vmbpirm_motion_output_1`) was initially disabled by default. It was enabled via the entity settings and used as the trigger for motion-based automation in Sprint 3.

---

## 8. Lessons Learned

This phase produced several reusable insights:

1. **Velbus modules must be programmed before Home Assistant can use them.** Hardware discovery alone is not sufficient.
2. **Delete-and-re-add is the most reliable way to refresh a Velbus integration after firmware changes.** Reload alone may not pick up new channels.
3. **VelbusLink's character limits do not propagate to Home Assistant.** Use VelbusLink for the technical configuration and Home Assistant for the user-facing names.
4. **Assigning entities to Areas is critical for voice control.** Voice commands like "turn on the kitchen light" rely on the area assignment to disambiguate which entity to control.
5. **The motion sensor entity is disabled by default** and must be enabled before it can be used in automations.

---

## 9. Verification

At the end of this phase, every Velbus channel of interest was controllable from the Home Assistant dashboard. Pressing a button on the VMBGPOD-2 panel toggled or dimmed the linked light, both at the hardware level and in the HA UI. The system was now ready for the voice pipeline integration (Document 3).
