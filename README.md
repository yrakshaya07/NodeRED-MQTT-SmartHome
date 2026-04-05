# Smart Home Monitoring System


---

## Objective
To design and implement a publish-subscribe system using Node-RED and MQTT 
broker that monitors smart home sensors and visualizes live data on a dashboard.
 
---

## Tools Used
- Node-RED
- HiveMQ Public Broker (`broker.hivemq.com:1883`)
- MQTT Protocol
- Node-RED Dashboard

---

## System Overview

Two publishers send sensor data to HiveMQ broker. A wildcard subscriber 
receives all messages and routes them to a live dashboard.

---

## Publishers

| Publisher | Topic | Interval | QoS |
|---|---|---|---|
| Temperature | `home_akshaya/livingroom/temperature` | Every 5s | 1 |
| Motion | `home_akshaya/livingroom/motion` | Every 7s | 1 |
| Status | `home_akshaya/status` | Every 30s | 1 |

---

## Wildcard Subscriber

**Topic:** `home_akshaya/#`  
Receives all messages published under `home_akshaya/` in a single subscription.

---

## Sample Message Log

**Temperature:**
```json
[home_akshaya/livingroom/temperature]
{"sensor":"temp_01","temperature":25.6,"humidity":68.6,"unit":"celsius",
"timestamp":"2026-04-05T06:41:10.249Z"}
```

**Motion:**
```json
[home_akshaya/livingroom/motion]
{"sensor":"motion_01","status":"detected","location":"livingroom",
"timestamp":"2026-04-05T06:28:20.950Z"}
```

**Status:**
```json
[home_akshaya/status] "online"
```

---

## Dashboard
- Temperature Gauge (live)
- Temperature Over Time (line chart)
- Motion Status (DETECTED / No Motion)
- Message Log (all topics)

---

## QoS vs Retained Messages

| Topic | QoS | Retain | Reason |
|---|---|---|---|
| Temperature | 1 | false | Live data only |
| Motion | 1 | false | Live data only |
| Status | 1 | true | New subscribers get last status |

---

## How Wildcards Worked
`home_akshaya/#` matched all three topics:
- `home_akshaya/livingroom/temperature` ✅
- `home_akshaya/livingroom/motion` ✅
- `home_akshaya/status` ✅

---



---

## Pub/Sub Roles
| Role | Node | Description |
|---|---|---|
| Publisher | Temperature Publisher | Sends temp data every 5s |
| Publisher | Motion Publisher | Sends motion data every 7s |
| Publisher | Status Publisher | Sends online status every 30s |
| Subscriber | Wildcard Subscriber | Receives all messages |
| Broker | HiveMQ | Routes messages between pub and sub |
