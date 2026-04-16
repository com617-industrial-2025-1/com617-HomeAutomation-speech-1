# Sprint 3 — Document 2: Node-RED Automation Implementation

## 1. Purpose

This document describes the installation, configuration, and population of the Node-RED add-on with the five automation flows that form the autonomous layer of the MDAR system. It also documents the SSL startup issue that initially prevented Node-RED from running, and the use of a Long-Lived Access Token to authenticate Node-RED against Home Assistant.

---

## 2. Installation and the SSL Startup Issue

The Node-RED add-on had been installed during Sprint 2 from the Home Assistant Apps store. When the team tried to start it for the first time in Sprint 3, the add-on would briefly start and then crash, looping indefinitely without exposing a Web UI.

The Log tab of the add-on revealed the cause:

```
[14:22:04] FATAL: SSL has been enabled using the 'ssl' option,
[14:22:04] FATAL: this requires an SSL certificate file which is
[14:22:04] FATAL: configured using the 'certfile' option in the
[14:22:04] FATAL: add-on configuration.
[14:22:04] FATAL: Unfortunately, the file specified in the
[14:22:04] FATAL: 'certfile' option does not exist.
```

By default, the Node-RED add-on enables SSL but expects a certificate file in the `/ssl/` directory. Since the system runs entirely on a trusted local network, SSL was not required. The fix was straightforward:

1. Opened the **Configuration** tab of the Node-RED add-on.
2. Set the `ssl` option to `false`.
3. Saved the configuration.
4. Started the add-on.

Node-RED started successfully. The Web UI button became active and opened the flow editor.

---

## 3. Authenticating with Home Assistant

When Node-RED nodes that interact with Home Assistant (such as `api-call-service` or `server-state-changed`) were placed on the canvas, they showed a `no connection` warning. Node-RED requires explicit credentials to talk to Home Assistant — even when running as an add-on on the same system.

A **Long-Lived Access Token** was generated to authenticate Node-RED:

1. In Home Assistant, clicked the user profile (bottom left of the sidebar).
2. Scrolled to the **Long-Lived Access Tokens** section.
3. Clicked **Create Token**, named it `Node-RED`, and copied the token string.
4. In Node-RED, double-clicked any Home Assistant node and clicked the pencil icon next to the **Server** field.
5. Set the Base URL to `http://homeassistant.local:8123` and pasted the token in the **Access Token** field.
6. Saved the configuration.

Once configured, the connection turned green and all Home Assistant nodes across all flows shared the same server credential.

---

## 4. Flow 1 — Motion-Triggered Lighting

**Trigger:** `binary_sensor.vmbpirm_motion_output_1` state changes.

**Behaviour:**
- When the sensor changes to `on`, turn on `light.livingroom_light_livingroom_light` (full brightness) and `switch.livingroom_spot_livingroom_spot`.
- When the sensor changes to `off`, wait 2 minutes and then turn both off. If new motion is detected during the wait, the timer resets.

**Node structure:**
- `server-state-changed` node listening to the motion sensor entity, with two outputs (one for `on`, one for `off`).
- The `on` branch leads to two `api-call-service` nodes that turn on the light and the spot.
- The `off` branch leads to a `delay` node (2 minutes) followed by two `api-call-service` nodes that turn off the light and the spot.
- Debug nodes capture the result for monitoring.

This flow satisfies requirement R6.

---

## 5. Flow 2 — All Lights On

**Trigger:** Manual inject button.

**Behaviour:** Turn on every dimmer light at 100% brightness and every relay-controlled spot.

**Node structure:**
- An `inject` node that the user clicks to trigger the flow.
- One `api-call-service` node calling `light.turn_on` on all four dimmer entities with `brightness_pct: 100`.
- One `api-call-service` node calling `switch.turn_on` on all four spot entities.
- A debug node logging the action.

This is useful for waking up the house quickly or as a "panic button" to make all rooms visible.

---

## 6. Flow 3 — All Lights Off

**Trigger:** Manual inject button.

**Behaviour:** Turn off every dimmer light and every relay-controlled spot.

**Node structure:** Identical to Flow 2 but calling `light.turn_off` and `switch.turn_off`.

This flow satisfies requirement R9 alongside Flow 2.

---

## 7. Flow 4 — Night Mode

**Trigger:** Two scheduled cron triggers — one at 22:00 and one at 07:00 daily.

**Behaviour:**
- At **22:00**: dim all four lights to 20% brightness and turn off all four spots.
- At **07:00**: restore all four lights to 100% brightness.

**Node structure:**
- Two `inject` nodes with cron schedules.
- The 22:00 node leads to a sequence: dim lights, then turn off spots.
- The 07:00 node leads to a single node: restore lights to full brightness.

This flow satisfies requirement R7.

---

## 8. Flow 5 — MQTT Landlord Reporting

**Trigger:** Cron schedule every 5 minutes.

**Behaviour:** Poll the state of all key entities, build a JSON report, and publish it to the MQTT topic `mdar/landlord/report`.

**Node structure:**
- An `inject` node firing every 300 seconds.
- A chain of `api-current-state` nodes, one per entity (4 lights, plus the motion sensor).
- A `function` node that constructs a JSON object from the collected states.
- An `mqtt out` node that publishes the JSON to `mdar/landlord/report` with QoS 1 and retain enabled.
- A debug node logging the published payload.

The JSON structure is:

```json
{
  "timestamp": "2026-03-19T14:30:00.000Z",
  "property": "MDAR Smart Home",
  "devices": {
    "living_room": { "light": "on", "spot": "check_separately" },
    "kitchen":     { "light": "off" },
    "bedroom":     { "light": "off" },
    "toilet":      { "light": "off" },
    "motion_sensor": { "status": "on" }
  }
}
```

This flow satisfies requirement R8 and forms the bridge between the local home network and any external landlord monitoring system. The full MQTT messaging architecture is described in Document 3 of this sprint.

---

## 9. Deployment and Testing

Once all five flows were built, the **Deploy** button (top right of Node-RED) was clicked to activate them. From that moment:

- Walking past the VMBPIRM caused the living room lights to come on, and they turned off two minutes after motion stopped.
- Pressing the inject button on the All Lights On flow turned on every light in the system within a second.
- The MQTT debug node showed reports being published every 5 minutes.
- The Night Mode flow was tested manually by clicking the inject buttons (rather than waiting until 22:00) and behaved as expected.

---

## 10. Voice Pipeline vs Node-RED — How They Coexist

A key insight, documented in Bridging Research Document 2, is that the voice pipeline and Node-RED operate as two independent layers over the same Home Assistant entities. A user speaking "turn off the kitchen light" via voice and the Night Mode flow turning off lights at 22:00 both end up calling the same `light.turn_off` service on the same entity. There is no conflict because Home Assistant serialises the calls and the last command wins.

This means the user can override an automated action at any time simply by speaking, and Arthur retains full conversational control over a system that otherwise runs itself.

---

## 11. Conclusion

The Node-RED add-on, once unblocked from the SSL startup issue and authenticated with a Long-Lived Access Token, became the autonomous brain of the MDAR system. Five flows now run without any user intervention, covering motion-triggered lighting, manual scene buttons, scheduled night-time dimming, and periodic MQTT reporting. Together with the voice pipeline, they deliver a system that is both responsive when spoken to and proactive when not.
