# Sprint 3 — Document 3: MQTT Messaging & Communication Layer

## 1. Purpose

This document describes the MQTT messaging layer that allows the MDAR system to publish structured device-state reports to an external landlord or monitoring service. It covers the configuration of the Mosquitto broker, the design of the report JSON schema, and the publish/subscribe pipeline implemented in Node-RED.

---

## 2. What is MQTT?

MQTT (Message Queuing Telemetry Transport) is a lightweight publish/subscribe messaging protocol designed for low-bandwidth, high-latency, or unreliable networks. It is the standard messaging protocol for IoT devices because of its small footprint and efficient handling of many concurrent connections.

In an MQTT system:

- A **broker** is a central server that routes messages.
- **Publishers** send messages to specific **topics** (e.g., `mdar/landlord/report`).
- **Subscribers** receive any message published to topics they have subscribed to.

The publisher and subscriber never communicate directly. They only know about the broker. This decoupling is what makes MQTT ideal for distributed IoT systems.

---

## 3. Why MQTT for Landlord Reporting?

Requirement R8 specifies that the system must publish device state reports to a remote landlord. There are several ways this could be implemented:

| Option | Pros | Cons |
|--------|------|------|
| HTTP POST to a landlord-managed REST API | Familiar, easy to debug | Requires the landlord to host an HTTP server; tightly coupled |
| Email reports | Simple | Not real-time; not structured |
| **MQTT publish** | Decoupled, structured, real-time, scalable to many properties | Requires both ends to support MQTT |

MQTT was chosen because:

1. The landlord can subscribe to topics from many properties simultaneously without the home network needing to know who is listening.
2. The broker can be hosted by either party (the home, the landlord, or a third-party cloud broker).
3. It is the de facto standard for IoT and is supported by every major smart home platform.
4. The Mosquitto broker is already running as a Home Assistant add-on (installed in Sprint 2).

---

## 4. Mosquitto Broker Configuration

The Mosquitto MQTT broker is a Home Assistant add-on that was installed in Sprint 2 and is already running. It listens on the standard MQTT port `1883` on the local network.

For this project, the default configuration was sufficient:

- Listens on `localhost:1883` for local connections.
- No authentication required for local network use.
- All clients (Home Assistant, Node-RED, future external subscribers) can connect freely on the trusted local network.

For a production deployment, the broker would be configured with username/password authentication and TLS encryption, especially if the landlord is subscribing from outside the local network. This is documented as a future improvement.

---

## 5. Topic Design

A clear, hierarchical topic structure makes it easy to subscribe to specific subsets of data. The team adopted the following pattern:

```
mdar/<property>/<message_type>
```

For this project's single-property prototype, the topics are:

| Topic | Purpose | Direction |
|-------|---------|-----------|
| `mdar/landlord/report` | Periodic device state report | Published by Node-RED, subscribed by landlord systems |

In a multi-property deployment, the structure would expand to:

```
mdar/property_001/report
mdar/property_002/report
mdar/property_001/alerts
mdar/property_001/commands
```

This pattern allows the landlord to subscribe to `mdar/+/report` (where `+` is an MQTT wildcard) and receive reports from all properties.

---

## 6. JSON Report Schema

The report payload is a JSON document. The structure was designed to be self-describing, easy to parse, and extensible:

```json
{
  "timestamp": "2026-03-19T14:30:00.000Z",
  "property": "MDAR Smart Home",
  "devices": {
    "living_room": {
      "light": "on",
      "spot": "off"
    },
    "kitchen": {
      "light": "off"
    },
    "bedroom": {
      "light": "off"
    },
    "toilet": {
      "light": "off"
    },
    "motion_sensor": {
      "status": "on"
    }
  }
}
```

Design notes:

- **`timestamp`** is in ISO 8601 format (UTC) so that downstream systems can sort, filter, and align messages from multiple properties.
- **`property`** identifies the source. In a multi-property deployment this becomes a unique identifier.
- **`devices`** is grouped by area (living room, kitchen, etc.) rather than by entity ID, because the landlord cares about the room state, not the technical entity name.
- States are represented as `"on"` / `"off"` strings rather than booleans, matching Home Assistant's native state format.

---

## 7. Implementation in Node-RED (Flow 5)

The MQTT reporting flow, described briefly in Document 2 of this sprint, is implemented as follows:

1. **`inject` node** — fires every 300 seconds (5 minutes), with an initial fire 10 seconds after Node-RED starts.
2. **`api-current-state` nodes** — one per polled entity. Each node retrieves the current state of one entity and stores it on the message object as a property (e.g., `msg.living_room_light = "on"`).
3. **`function` node** — runs JavaScript code that constructs the JSON report from the collected message properties:

   ```javascript
   var report = {
       timestamp: new Date().toISOString(),
       property: "MDAR Smart Home",
       devices: {
           living_room: {
               light: msg.living_room_light || "unknown",
               spot: "check_separately"
           },
           kitchen:  { light: msg.kitchen_light  || "unknown" },
           bedroom:  { light: msg.bedroom_light  || "unknown" },
           toilet:   { light: msg.toilet_light   || "unknown" },
           motion_sensor: { status: msg.motion || "unknown" }
       }
   };
   msg.payload = JSON.stringify(report, null, 2);
   return msg;
   ```

4. **`mqtt out` node** — publishes the resulting payload to the topic `mdar/landlord/report` with:
   - **QoS 1** (at least once delivery).
   - **Retain: true** (the broker keeps the most recent message, so any new subscriber immediately receives the latest report).
5. **`debug` node** — logs the published payload to the Node-RED debug panel for inspection.

---

## 8. Subscribing to Reports

For a landlord to consume these reports, they would run an MQTT client subscribed to `mdar/landlord/report` (or the wildcarded multi-property pattern in production).

For testing during Sprint 3, the team used the **MQTT Explorer** tool on a laptop to subscribe to the broker and confirm that reports were arriving every 5 minutes with the expected schema. Reports appeared on schedule and contained the current state of all monitored devices.

---

## 9. Future Improvements

The MQTT layer is functional but several enhancements would be needed for a real deployment:

1. **Authentication and TLS** — for any external broker access.
2. **Bidirectional commands** — a `mdar/landlord/commands` topic that allows the landlord to send commands to the property (e.g., "turn off all lights" if a tenant has moved out).
3. **Alert messages** — a separate `mdar/<property>/alerts` topic for high-priority events (smoke detected, water leak, etc.).
4. **Compressed payloads** — for properties with constrained bandwidth, the JSON could be compressed or replaced with a binary format like CBOR.
5. **Cloud broker** — using a hosted MQTT service (HiveMQ Cloud, AWS IoT Core) so that landlords do not have to run their own broker infrastructure.

---

## 10. Conclusion

The MQTT messaging layer adds a critical second interface to the MDAR system. The voice pipeline and Node-RED flows serve the resident; the MQTT publisher serves the building manager. Both layers operate on the same underlying entity model in Home Assistant but expose it for two completely different audiences. This architectural separation is what makes the system scalable from one flat to many.
