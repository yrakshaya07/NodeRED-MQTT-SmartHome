# 🏠 Smart Home MQTT Monitoring System — Node-RED

**Student Name:** Akshaya  
**Assignment:** Publish-Subscribe System using Node-RED and MQTT  
**Broker:** HiveMQ Public Broker (`broker.hivemq.com:1883`)  
**Due Date:** 8th Feb 2026

---

## 📋 Overview

This project implements a **Smart Home Monitoring System** using Node-RED and MQTT pub-sub architecture. It simulates two IoT sensors (temperature and motion) publishing live data to an MQTT broker, with a wildcard subscriber routing messages to a real-time dashboard.

---

## 🏗️ System Architecture

```
Publishers                    Broker                  Subscriber
----------                 -----------              -------------
[Temp Sensor]  --QoS1-->  [HiveMQ]   --wildcard-->  [Node-RED]
[Motion Sensor]--QoS1-->  [HiveMQ]   home_akshaya/#  [Dashboard]
[Status]       --retain-> [HiveMQ]
```

---

## 📡 MQTT Topics

| Topic | Publisher | QoS | Retained | Description |
|-------|-----------|-----|----------|-------------|
| `home_akshaya/livingroom/temperature` | Temperature Publisher | 1 | No | Live temperature + humidity |
| `home_akshaya/livingroom/motion` | Motion Publisher | 1 | No | Motion detected/clear |
| `home_akshaya/status` | Status Publisher | 1 | Yes | Device online status |

**Wildcard Subscriber Topic:** `home_akshaya/#`

---

## 🔧 Node-RED Flow Description

### Publishers (Flow 1)

**1. Temperature Publisher**
- Triggers every **5 seconds** via inject node
- Publishes JSON payload with temperature (20–30°C), humidity, sensor ID, and timestamp
- Topic: `home_akshaya/livingroom/temperature`
- QoS: 1 (at least once delivery)

**2. Motion Publisher**
- Triggers every **7 seconds** via inject node
- Randomly publishes `detected` or `clear` motion status
- Topic: `home_akshaya/livingroom/motion`
- QoS: 1

**3. Status Publisher**
- Triggers every **30 seconds**
- Publishes `online` status with timestamp
- Topic: `home_akshaya/status`
- QoS: 1, **Retain: true** (broker stores last message)

### Subscriber + Dashboard (Flow 1)

**Wildcard Subscriber (`home_akshaya/#`)**
- Subscribes to ALL topics under `home_akshaya/`
- Single subscriber handles all sensor data

**Parse & Route Function**
- Parses incoming JSON
- Routes temperature to gauge + chart (output 1)
- Routes motion to text display (output 2)
- Routes all messages to message log (output 3)

---

## 📊 Dashboard Components

| Widget | Data Source | Description |
|--------|-------------|-------------|
| Gauge | Temperature topic | Shows live temperature 0–50°C |
| Line Chart | Temperature topic | Historical temperature over time |
| Text | Motion topic | Shows Motion DETECTED / No Motion |
| Text Log | All topics | Shows raw MQTT messages |

---

## 💻 Function Node Code

### Temperature Publisher
```javascript
var temp = (20 + Math.random() * 10).toFixed(1);
var humidity = (40 + Math.random() * 30).toFixed(1);

msg.payload = JSON.stringify({
    sensor: "temp_01",
    temperature: parseFloat(temp),
    humidity: parseFloat(humidity),
    unit: "celsius",
    timestamp: new Date().toISOString()
});
msg.topic = "home_akshaya/livingroom/temperature";
msg.qos = 1;
msg.retain = false;
return msg;
```

### Motion Publisher
```javascript
var motionStates = ["detected", "clear"];
var status = motionStates[Math.floor(Math.random() * 2)];

msg.payload = JSON.stringify({
    sensor: "motion_01",
    status: status,
    location: "livingroom",
    timestamp: new Date().toISOString()
});
msg.topic = "home_akshaya/livingroom/motion";
msg.qos = 1;
msg.retain = false;
return msg;
```

### Status Publisher
```javascript
msg.payload = JSON.stringify({
    status: "online",
    timestamp: new Date().toISOString()
});
msg.topic = "home_akshaya/status";
msg.qos = 1;
msg.retain = true;
return msg;
```

### Parse & Route
```javascript
var data;
try { 
    data = JSON.parse(msg.payload); 
} catch(e) { 
    data = msg.payload; 
}

var out1 = null, out2 = null, out3 = null;

if (msg.topic.includes("temperature")) {
    var temp = parseFloat(data.temperature);
    if (!isNaN(temp)) {
        out1 = { payload: temp, topic: msg.topic };
    }
} else if (msg.topic.includes("motion")) {
    out2 = { 
        payload: data.status === "detected" ? 1 : 0, 
        topic: msg.topic,
        status: data.status 
    };
}

out3 = { 
    payload: "[" + msg.topic + "] " + JSON.stringify(data), 
    topic: msg.topic 
};

return [out1, out2, out3];
```

### Motion to Text
```javascript
if (msg.payload === 1) {
    msg.payload = "Motion DETECTED";
} else {
    msg.payload = "No Motion";
}
return msg;
```

---

## 📨 Sample Message Log

```
[home_akshaya/livingroom/temperature] {"sensor":"temp_01","temperature":24.3,"humidity":58.2,"unit":"celsius","timestamp":"2026-04-05T06:28:10.123Z"}

[home_akshaya/livingroom/motion] {"sensor":"motion_01","status":"detected","location":"livingroom","timestamp":"2026-04-05T06:28:14.950Z"}

[home_akshaya/status] {"status":"online","timestamp":"2026-04-05T06:28:20.255Z"}
```

---

## 🔁 QoS vs Retained Messages

| Feature | Value | Effect |
|---------|-------|--------|
| QoS 0 | At most once | Message may be lost |
| QoS 1 | At least once | Message guaranteed, may duplicate |
| QoS 2 | Exactly once | Guaranteed, no duplicates |
| Retained | true | Broker stores last message, new subscribers get it instantly |

In this project:
- Temperature and Motion use **QoS 1** — ensures delivery even on unstable networks
- Status uses **QoS 1 + Retain: true** — new subscribers immediately know device is online

---

## 🌐 How Wildcards Work

The wildcard subscriber `home_akshaya/#` matches:
- `home_akshaya/livingroom/temperature` ✅
- `home_akshaya/livingroom/motion` ✅
- `home_akshaya/status` ✅
- `home_akshaya/hallway/temperature` ✅ (any future topic)

This means **one subscriber handles all sensors** — no need for separate subscribers per topic.

---

## 🛠️ Setup Instructions

### Prerequisites
- Node-RED installed (`npm install -g node-red`)
- node-red-dashboard installed (`Manage Palette → node-red-dashboard`)

### Import Flow
1. Open `localhost:1880`
2. Click menu (☰) → **Import**
3. Paste contents of `flow.json`
4. Click **Deploy**
5. Open dashboard at `localhost:1880/ui`

### MQTT Broker Settings
- Server: `broker.hivemq.com`
- Port: `1883`
- Client ID: `home_akshaya_client_001`
- Keep Alive: `60`

---

## 📁 Repository Structure

```
smart-home-nodered/
├── README.md           ← This file
├── flow.json           ← Node-RED flow (import this)
└── screenshots/
    ├── flow.png        ← Node-RED flow screenshot
    └── dashboard.png   ← Dashboard screenshot
```

---

## 🎓 Concepts Demonstrated

- **Publish-Subscribe Pattern** — Publishers and subscribers are decoupled
- **MQTT Protocol** — Lightweight IoT messaging protocol
- **Wildcard Subscriptions** — `#` matches all subtopics
- **QoS Levels** — Guaranteed message delivery
- **Retained Messages** — Broker stores last status message
- **Real-time Dashboard** — Live data visualization with Node-RED UI
