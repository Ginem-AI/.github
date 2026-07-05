# Ginem - AI Agent Smart Home IoT Platform

<p align="center">
  <strong>Control, monitor, schedule, and automate real IoT devices using natural language.</strong>
</p>

<p align="center">
WhatsApp + Web Chat · LLM Agent · RabbitMQ · RAG · MQTT · IoT Devices · Scheduler · Dynamic Rule Engine
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-20+-339933?style=flat-square&logo=node.js&logoColor=white" />
  <img src="https://img.shields.io/badge/TypeScript-5+-3178C6?style=flat-square&logo=typescript&logoColor=white" />
  <img src="https://img.shields.io/badge/Express.js-Backend-000000?style=flat-square&logo=express&logoColor=white" />
  <img src="https://img.shields.io/badge/RabbitMQ-Message%20Queue-FF6600?style=flat-square&logo=rabbitmq&logoColor=white" />
  <img src="https://img.shields.io/badge/Redis-Cache%20%2F%20Jobs-DC382D?style=flat-square&logo=redis&logoColor=white" />
  <img src="https://img.shields.io/badge/MySQL-Database-4479A1?style=flat-square&logo=mysql&logoColor=white" />
  <img src="https://img.shields.io/badge/MQTT-HiveMQ-660066?style=flat-square&logo=hivemq&logoColor=white" />
  <img src="https://img.shields.io/badge/IoT-MQTT%20Devices-black?style=flat-square" />
  <img src="https://img.shields.io/badge/LangChain-AI%20Agent-purple?style=flat-square" />
  <img src="https://img.shields.io/badge/Pinecone-RAG-000000?style=flat-square" />
  <img src="https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker&logoColor=white" />
</p>

<p align="center">
  <a href="https://misdar.dev">Portfolio</a> ·
  <a href="https://github.com/misdarmanto">GitHub</a> ·
  <a href="https://www.linkedin.com/in/misdar-manto-06a8b2231/">LinkedIn</a>
</p>

---

## What is Ginem?

**Ginem** is an AI Agent-powered Smart Home IoT platform that lets users control, monitor, schedule, and automate physical devices using natural language.

Traditional IoT apps often feel harder than they should be. Too many buttons, too many menus, too many settings, and too much context users need to understand before they can actually control their own devices.

Ginem takes a different approach: just say what you want.

Users can control devices, check sensor data, create schedules, or define automation rules using simple everyday language through WhatsApp or web chat.

```text
Turn on the living room light
What is the current temperature?
Turn off all devices at 11 PM
Turn on the fan if temperature is above 30°C
Turn on all the lights in my house at 6 PM and turn them off at 6 AM every day
```
The system processes the command through an LLM-based AI Agent, retrieves device context using RAG, executes validated backend tools, and communicates with real IoT devices through MQTT.

Ginem is designed to be device-agnostic. The current prototype uses ESP32 with DHT11, relay, and LED as the reference hardware, but the platform is not limited to ESP32. Any internet-connected microcontroller or IoT device can be integrated as long as it is registered in the dashboard and follows the MQTT topic and payload contract.

---

## Project Repositories

| Repository | Description | Setup guide |
|------------|-------------|-------------|
| **ginem-api** | Backend API, AI Agent worker, RabbitMQ queue, MQTT service, scheduler, rule engine, RAG, authentication, and Swagger documentation | [`CORE/README.md`](CORE/README.md) |
| **ginem-dashboard** | Admin dashboard and web chat interface for managing devices, telemetry logs, schedules, rules, and system settings | [`ginem-admin/README.md`](ginem-admin/README.md) |
| **ginem-hardware** | Reference IoT firmware for MQTT-based device communication. The current prototype uses ESP32, DHT11, relay, and LED, but the platform can support any internet-connected microcontroller or IoT device that follows the MQTT topic and payload contract. | [`ginem-hardware/README.md`](ginem-hardware/README.md) |
| **`.github`** | Organization profile and high-level project overview | [`.github/profile/README.md`](.github/profile/README.md) |

**Recommended order to run the full stack:**

1. Start **ginem-api** first (database, Redis, RabbitMQ, API).
2. Start **ginem-dashboard** and point it to the API URL.
3. Flash **ginem-hardware** the reference firmware or connect any MQTT-compatible IoT device, then register it in the dashboard.

---

## System Architecture

<img width="1447" height="1087" alt="ARSITEKTUR EN" src="https://github.com/user-attachments/assets/35965167-946a-4a7e-a940-01e9b9df07ec" />



The LLM does not directly control hardware. Every device action must pass through validated backend tools before an MQTT command is published to a registered IoT device.

---

## Engineering Highlights

| Area | Implementation |
|------|----------------|
| Backend Architecture | Modular Node.js, Express.js, and TypeScript backend |
| AI Agent | LangChain-based agent with structured tool/function calling |
| Message Queue | RabbitMQ for asynchronous AI Agent request processing |
| RAG | Pinecone vector database for device and system context retrieval |
| IoT Communication | MQTT over TLS using HiveMQ Cloud |
| Hardware Integration | Supports registered MQTT-capable IoT devices; ESP32 is used as the reference implementation |
| Scheduler | Redis + BullMQ for one-time and recurring device automation |
| Dynamic Rules | Event-Condition-Action rule engine for sensor-triggered automation |
| Database | MySQL for users, devices, telemetry, schedules, rules, and logs |
| Interfaces | WhatsApp via Baileys and web chat through REST API |
| Observability | Logs for AI interactions, MQTT commands, telemetry, schedules, and rule execution |
| Safety | LLM cannot execute arbitrary hardware commands; all actions are validated by backend services |

---

## Core Features

### Natural Language Device Control

Users can control IoT devices through WhatsApp or web chat.

```text
Turn on the living room light
Turn off all devices
```

The AI Agent converts the command into a backend tool call, validates the target device and action based on the registered device metadata, then publishes an MQTT command to the target IoT device.

### Sensor Monitoring

Registered IoT devices can publish telemetry data through MQTT, such as temperature, humidity, power state, or other sensor readings depending on the device capability.

```text
What is the current temperature?
Show me the last 10 humidity readings
```

Telemetry is stored in MySQL and can be queried through the AI Agent, web chat, or dashboard.

### Scheduler Automation

Users can create one-time or recurring automation schedules using natural language.

```text
Turn off the light every day at 11 PM
Turn on the fan tomorrow at 7 AM
```

Schedules are stored in MySQL and executed using Redis-backed BullMQ workers.

### Dynamic Rule Engine

Users can define condition-based automation rules using natural language.

```text
Turn on the fan if temperature is above 30°C
```

The system converts the command into an Event-Condition-Action rule:

```json
{
  "event": "telemetry received from a registered sensor device",
  "condition": "temperature > 30",
  "action": "publish MQTT command to the registered fan actuator"
}
```

Rules are stored in MySQL, cached in Redis, and evaluated automatically whenever new telemetry arrives through MQTT.

### WhatsApp and Web Chat Interfaces

Ginem supports two interaction channels:

- **WhatsApp**, for mobile-first natural language control.
- **Web Chat**, for dashboard-integrated interaction and internal system use.

---

## Tech Stack

| Layer | Technologies |
|-------|-------------|
| Backend | Node.js, Express.js, TypeScript |
| Database | MySQL, Sequelize |
| Queue & Jobs | RabbitMQ, Redis, BullMQ |
| AI Agent | LangChain, LLM Function Calling |
| RAG | Pinecone Vector Database |
| IoT | MQTT-compatible IoT devices, ESP32 reference firmware, sensors, actuators, relay, LED |
| Messaging | MQTT, HiveMQ Cloud |
| Interfaces | WhatsApp via Baileys, Web Chat API |
| Documentation | Swagger |
| Infrastructure | Docker Compose |

---

## Background

**Ginem** was developed as part of my final thesis in **Telecommunication Engineering** at **Institut Teknologi Sumatera**.

The research focuses on building an LLM-based AI Agent for controlling and monitoring IoT devices using natural language. The system combines AI Agent reasoning, RabbitMQ-based asynchronous processing, function calling, RAG, MQTT-based device communication, registered device management, scheduling, and dynamic rule automation.

Beyond the thesis, this project represents my interest in building backend systems that connect AI, automation, distributed architecture, and real-world hardware.

---

<p align="center">
  <strong>Ginem - Smart Home IoT, powered by AI.</strong>
</p>

<p align="center">
  Made with ❤️ by <a href="https://github.com/misdarmanto">Me</a>
</p>
