# NodeRED-MQTT-SmartHome
Smart Home Monitoring using Node-RED and MQTT (Publish-Subscribe System)
# DA-2: MQTT Publish–Subscribe System — Smart Home Monitor

## System Design

**Theme:** Smart Home Monitoring  
**Broker:** HiveMQ Public Broker (`broker.hivemq.com:1883`)  
**Tool:** Node-RED v3.x

---

## Architecture

### Publishers

| Publisher | Topic | QoS | Interval | Payload |
|-----------|-------|-----|----------|---------|
| Temperature Sensor | `home/livingroom/temperature` | 1 | 5s | `{sensor, temperature, humidity, timestamp}` |
| Motion Sensor | `home/hallway/motion` | 2 | 7s | `{sensor, status, room, timestamp}` |
| Status Heartbeat | `home/status` | 1 (retained) | 30s | `"online"` |

### Subscriber

| Subscriber | Wildcard Topic | QoS |
|------------|----------------|-----|
| Wildcard Subscriber | `home/#` | 2 |

The wildcard `#` matches ALL sub-topics under `home/`, including:
- `home/livingroom/temperature`
- `home/hallway/motion`
- `home/status`

---

## QoS Demonstration

| Level | Used For | Meaning |
|-------|----------|---------|
| QoS 0 | N/A | Fire and forget — no acknowledgment |
| QoS 1 | Temperature, Status | At-least-once delivery — broker acknowledges |
| QoS 2 | Motion sensor | Exactly-once delivery — 4-step handshake |

**Retained Messages:** The `home/status` topic uses `retain: true`, meaning any new subscriber immediately receives the last known status without waiting for the next publish cycle.

---

## How Wildcards Work

MQTT supports two wildcards:
- **`+`** — single level (e.g., `home/+/temperature` matches one room)
- **`#`** — multi-level (e.g., `home/#` matches everything under home/)

Our subscriber uses `home/#` to receive messages from **all sensors** with a single subscription, routing them through a Parse & Route function node.

---

## Dashboard

Accessible at: `http://localhost:1880/ui`

| Widget | Data Source |
|--------|-------------|
| Temperature Gauge | `home/livingroom/temperature` |
| Temperature Line Chart | `home/livingroom/temperature` (time series) |
| Motion Status Text | `home/hallway/motion` |
| Message Log | All `home/#` messages |

---

## Message Log Sample
