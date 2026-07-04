# Ginem — AI Agent for Smart Home IoT Control

<p align="center">
  <strong>Control, monitor, schedule, and automate real IoT devices using natural language through WhatsApp and web chat.</strong>
</p>

<p align="center">
  Built with Node.js, TypeScript, Express.js, RabbitMQ, Redis, MySQL, MQTT, ESP32, LangChain, RAG, and LLM function calling.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-20+-339933?style=flat-square&logo=node.js&logoColor=white" />
  <img src="https://img.shields.io/badge/TypeScript-5+-3178C6?style=flat-square&logo=typescript&logoColor=white" />
  <img src="https://img.shields.io/badge/Express.js-Backend-000000?style=flat-square&logo=express&logoColor=white" />
  <img src="https://img.shields.io/badge/RabbitMQ-Message%20Queue-FF6600?style=flat-square&logo=rabbitmq&logoColor=white" />
  <img src="https://img.shields.io/badge/Redis-Cache%20%2F%20Jobs-DC382D?style=flat-square&logo=redis&logoColor=white" />
  <img src="https://img.shields.io/badge/MySQL-Database-4479A1?style=flat-square&logo=mysql&logoColor=white" />
  <img src="https://img.shields.io/badge/MQTT-HiveMQ-660066?style=flat-square&logo=hivemq&logoColor=white" />
  <img src="https://img.shields.io/badge/ESP32-IoT-black?style=flat-square" />
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

## Documentation & Setup

This README is the **project overview** — architecture, features, and tech stack.

If you want to **run or deploy** Ginem, follow the setup guide in each repository:

| Repository | What you'll find | Setup guide |
|------------|------------------|-------------|
| **`.github`** | Organization profile — overview, repos, **Quick Start full stack** | [`.github/profile/README.md`](.github/profile/README.md) |
| **`ginem-api`** | Backend API — local run, Docker, env vars, MQTT, Swagger, testing | [`CORE/README.md`](CORE/README.md) |
| **`ginem-admin`** | Admin dashboard — Vite, `VITE_API_URL`, build, troubleshooting | [`ginem-admin/README.md`](ginem-admin/README.md) |
| **`ginem-hardware`** | ESP32 firmware — MQTT topics, flash, wiring | [`ginem-hardware/README.md`](ginem-hardware/README.md) |

**Recommended order to run the full stack:**

1. Start **ginem-api** first (database, Redis, RabbitMQ, API).
2. Start **ginem-admin** and point it to the API URL.
3. Flash **ginem-hardware** and register the device in the admin dashboard.

---

## Overview

**Ginem** is an AI Agent-powered smart home IoT platform that allows users to control, monitor, schedule, and automate physical devices using natural language.

Instead of forcing users to interact with fixed dashboard buttons or strict command formats, Ginem lets users send commands such as:

```text
Turn on the living room light
What is the current room temperature?
Turn off all devices
Turn on the fan every day at 6 PM
Turn on the fan if the temperature is above 30°C
Show me the last 10 humidity readings
```

The system uses a LangChain-based LLM agent to interpret user intent, retrieve relevant device context through RAG, choose a safe backend tool, and execute the requested action through MQTT on real ESP32-based IoT hardware.

Ginem is built as a full backend platform, not just a simple chatbot prototype. It includes authenticated REST APIs, **RabbitMQ message queue** for asynchronous AI Agent processing, relational data modeling, background job scheduling, MQTT messaging, structured logging, dynamic rule evaluation, vector search, and WhatsApp/web chat integration.

---

## Why Ginem Exists

Most conventional IoT systems rely on dashboards, static buttons, manual configuration, or rigid command formats. This works for simple use cases, but becomes less flexible as the number of devices, sensors, schedules, and automation rules increases.

Ginem introduces a more natural interaction model:

```text
Natural language → Backend API → RabbitMQ → AI Agent → Tool selection → Backend validation → MQTT command → Physical device action
```

The goal is to make IoT systems easier to use, more adaptive, and more accessible for non-technical users while still preserving backend-level validation and control.

---
## Architecture

<img width="1448" height="1086" alt="Arsitektur Perangkat Lunak" src="https://github.com/user-attachments/assets/4dd640ad-13a9-4f04-9f00-b311fe0a38b0" />


### Request Flow

1. User sends a message through WhatsApp or web chat.
2. Backend authenticates and receives the request.
3. Backend **publishes the chat job to RabbitMQ**.
4. **AI Agent worker** consumes the message from the queue.
5. RAG retrieves relevant context from Pinecone.
6. The LangChain agent receives the user message, retrieved context, and available tools.
7. The LLM decides whether to answer directly or invoke a tool.
8. The selected backend tool validates input arguments.
9. The backend performs one of several actions:
   - queries MySQL,
   - publishes an MQTT command,
   - creates a scheduled job,
   - creates or updates a dynamic rule,
   - retrieves telemetry logs.
10. ESP32 receives MQTT commands and executes the action.
11. ESP32 publishes state or telemetry data back to the backend.
12. The user receives a natural language response.

```text
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│  WhatsApp   │     │   Web Chat   │     │  REST API   │
│  (Baileys)  │     │   Frontend   │     │  Express.js │
└──────┬──────┘     └──────┬───────┘     └──────┬──────┘
       │                   │                    │
       └───────────────────┴────────────────────┘
                           │
                    ┌──────▼──────┐
                    │  RabbitMQ   │
                    │ Message Queue│
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  AI Agent   │◄──── Pinecone (RAG)
                    │  LangChain  │
                    │  + MCP Tools│
                    └──────┬──────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
   ┌──────▼──────┐  ┌──────▼──────┐  ┌─────▼─────┐
   │    MySQL    │  │    Redis    │  │  HiveMQ   │
   │  Sequelize  │  │   BullMQ    │  │   Cloud   │
   └─────────────┘  └─────────────┘  └─────┬─────┘
                                           │ MQTT
                                    ┌──────▼──────┐
                                    │    ESP32    │
                                    │ DHT11/Relay │
                                    └─────────────┘
```

---

## AI Agent Workflow

<p align="center">
  <img src="./assets/agent-workflow.svg" alt="Step-by-step AI Agent workflow: user sends message, backend retrieves context, AI Agent selects tool, backend validates tool call, MQTT command or database operation is executed, then natural language response is returned." width="750"/>
</p>

The AI Agent supports several intent categories:

| Intent | Description |
|--------|-------------|
| `control_device` | Turn an actuator on or off |
| `monitor_sensor` | Retrieve latest sensor data |
| `get_device_status` | Check current device state |
| `create_schedule` | Create one-time or recurring automation |
| `create_dynamic_rule` | Create condition-based automation |
| `query_knowledge` | Answer system or device-related questions |
| `clarification_required` | Ask for clarification when the command is ambiguous |
| `invalid_command` | Reject unsupported or unsafe commands |

---

## Dynamic Rule Engine

Ginem includes a Dynamic Rule Engine that allows users to create automation rules using natural language.

**Example:**

```text
Turn on the fan if the room temperature is above 30°C
```

The AI Agent converts the command into an **Event-Condition-Action** structure:

```json
{
  "ruleName": "Turn on fan when temperature is high",
  "trigger": {
    "sourceDevice": "room_sensor",
    "metric": "temperature",
    "eventType": "telemetry"
  },
  "condition": {
    "operator": ">",
    "value": 30,
    "unit": "celsius"
  },
  "action": {
    "targetDevice": "fan_relay",
    "command": "TURN_ON"
  }
}
```

### Rule Lifecycle

```text
Natural language rule
  ↓
AI Agent extracts trigger, condition, and action
  ↓
Rule Management Service validates devices, metrics, operator, and action
  ↓
Rule is stored in MySQL
  ↓
Active rules are cached in Redis
  ↓
New sensor telemetry arrives through MQTT
  ↓
Rule Engine evaluates matching rules
  ↓
If condition is true, backend publishes MQTT action
  ↓
Execution result is stored in rule logs
```

### Event-Condition-Action Model

| Part | Description | Example |
|------|-------------|---------|
| Event | Incoming trigger from a sensor or device | New temperature telemetry |
| Condition | Logical comparison that must be true | Temperature > 30 |
| Action | Device command to execute | Turn on fan relay |

### Rule Engine Features

- Create rules from natural language.
- Validate source device, target device, metric, operator, and action.
- Store rule structure in MySQL.
- Cache active rules in Redis for faster evaluation.
- Evaluate rules when telemetry arrives from MQTT.
- Execute actuator commands through MQTT.
- Store rule execution history.
- Support cooldown to prevent repeated actions too frequently.
- Avoid unnecessary LLM calls during rule execution by storing structured rules.

---

## Scheduler Automation

Ginem supports natural language scheduling for device automation.

**Examples:**

```text
Turn on the light every day at 6 PM
Turn off all devices at 11 PM
Fetch sensor data tomorrow at 8 AM
```

The scheduler extracts:

- target device,
- action,
- execution time,
- recurrence type,
- timezone,
- job status.

| Type | Description |
|------|-------------|
| `once` | One-time execution at a specific datetime |
| `repeat` | Recurring execution, such as daily automation |

Scheduler jobs are persisted in MySQL and executed through BullMQ workers backed by Redis.

---

## RAG Context Retrieval

Ginem uses Retrieval-Augmented Generation to make the AI Agent more context-aware.

The vector database may store:

- device descriptions,
- device capabilities,
- sensor descriptions,
- function/tool documentation,
- scheduler instructions,
- dynamic rule examples,
- system constraints,
- user command examples,
- technical documentation.

Before each chat turn, relevant context is retrieved and injected into the agent prompt. This allows the agent to reason with system-specific information instead of relying only on the model's general training data.

---

## Tool-Based Execution

The AI Agent can only execute predefined tools. This keeps the system predictable and safer for IoT control.

**Example tool call:**

```json
{
  "name": "set_actuator_state_by_device_name",
  "arguments": {
    "deviceName": "living_room_light",
    "state": "on"
  }
}
```

The backend validates:

- whether the device exists,
- whether the device supports the requested action,
- whether the arguments are complete,
- whether the action is allowed,
- whether the MQTT topic can be resolved.

Only after validation does the backend publish a command to the MQTT broker.

### Available Agent Tools

| Tool | Description |
|------|-------------|
| `list_devices` | Returns a paginated list of registered IoT devices |
| `get_device_by_id` | Retrieves a device by numeric ID |
| `get_last_log_by_device_name` | Gets the latest telemetry log for a device |
| `get_last_10_logs_by_device_name` | Gets recent telemetry history |
| `create_device_log_by_device_name` | Creates a manual device log |
| `set_actuator_state_by_device_name` | Sends ON/OFF MQTT command to an actuator |
| `schedule_actuator_state_at` | Schedules actuator ON/OFF action |
| `schedule_sensor_data_at` | Schedules future sensor data retrieval |
| `get_scheduled_job_result` | Checks scheduled job status |
| `list_scheduled_jobs` | Lists recent scheduled jobs |
| `create_dynamic_rule` | Creates an Event-Condition-Action rule from natural language |
| `list_dynamic_rules` | Lists active and inactive automation rules |
| `update_dynamic_rule` | Updates an existing rule |
| `delete_dynamic_rule` | Deletes or disables a rule |
| `get_rule_execution_logs` | Returns dynamic rule execution history |

---
## Key Capabilities

- Natural language control for actuators such as relays, lamps, and fans.
- Real-time monitoring for sensor data such as temperature and humidity.
- Function calling / tool use to safely convert user intent into backend actions.
- RAG with Pinecone to provide device and system context to the AI Agent.
- **RabbitMQ message queue** to process AI Agent chat requests asynchronously and decouple API from LLM execution.
- MQTT communication between backend and ESP32 devices through HiveMQ Cloud.
- WhatsApp interface using Baileys for mobile-first interaction.
- Web chat API for dashboard or internal chat integration.
- Scheduler automation for one-time and recurring actions.
- Dynamic Rule Engine for condition-based automation across devices.
- MySQL persistence for users, devices, telemetry, schedules, rules, and logs.
- Redis + BullMQ for background jobs and scheduler execution.
- Swagger documentation for API exploration and testing.
- Docker Compose for local infrastructure setup.

---

## Engineering Highlights

This project demonstrates practical backend, AI, and IoT engineering skills in one system.

| Area | Implementation |
|------|----------------|
| Backend Architecture | Modular Express.js + TypeScript service architecture |
| AI Agent | LangChain agent with structured tool/function calling |
| Message Queue | RabbitMQ for async AI Agent request processing |
| RAG | Pinecone vector search for contextual retrieval |
| IoT Messaging | MQTT over TLS using HiveMQ Cloud |
| Device Layer | ESP32 with DHT11 sensor and relay/LED actuator |
| Scheduling | Redis-backed BullMQ jobs for one-time and recurring automation |
| Dynamic Rules | Event-Condition-Action rule engine for sensor-triggered automation |
| Database | MySQL with structured relational entities |
| Authentication | JWT-based protected API access |
| API Docs | Swagger UI for interactive testing |
| Messaging Interface | WhatsApp integration using Baileys |
| DevOps | Docker Compose for backend, MySQL, and Redis |
| Observability | Application, device, scheduler, and AI interaction logs |
| Safety | Validated backend tools instead of arbitrary model-generated commands |

---

## Demo Commands

| Use Case | Command | Result |
|----------|---------|--------|
| Device control | Turn on the living room light | Actuator receives MQTT ON command |
| Device control | Matikan lampu kamar | Device is turned off through MQTT |
| Multi-device query | List all devices | Agent calls `list_devices` and returns device list |
| Sensor monitoring | What is the current temperature? | Latest sensor log is fetched and summarized |
| Historical telemetry | Show last 10 logs for DHT11 sensor | Recent readings are returned in natural language |
| Scheduler, recurring | Turn off the light every day at 11 PM | Daily job is created |
| Scheduler, one-time | Turn on the fan tomorrow at 7 AM | One-time job is enqueued in BullMQ |
| Dynamic rule | Turn on the fan if temperature is above 30°C | Rule is created and evaluated automatically |
| Knowledge query | How does the relay device work? | RAG context + agent-generated answer |

---

## How It Works

At a high level, Ginem turns human language into controlled backend actions.

```text
User message
  ↓
WhatsApp or Web Chat
  ↓
Backend API
  ↓
RabbitMQ (message queue)
  ↓
AI Agent worker
  ↓
RAG context retrieval
  ↓
LLM Agent reasoning
  ↓
Validated tool/function call
  ↓
Database query, MQTT command, scheduled job, or dynamic rule
  ↓
Natural language response
```

The model does not directly control hardware. Every action must go through predefined backend tools with validated arguments.

---

## RabbitMQ Message Queue

Ginem uses **RabbitMQ** as the primary message queue between the REST API and the AI Agent worker layer.

When a user sends a chat message through WhatsApp or web chat, the backend does not block the HTTP request while waiting for LLM inference. Instead:

1. The API **authenticates** the request and **publishes** a job to a RabbitMQ queue.
2. An **AI Agent worker** consumes the message asynchronously.
3. The worker runs RAG retrieval, LLM reasoning, and tool execution.
4. The result is stored and returned to the user as a natural language response.

```text
Client request
  ↓
Express API
  ↓
Publish to RabbitMQ queue
  ↓
AI Agent worker consumes message
  ↓
LangChain + tools + RAG
  ↓
Response persisted / delivered
```

**Why RabbitMQ?**

| Benefit | Description |
|---------|-------------|
| Async processing | Long-running LLM calls do not block the API thread |
| Decoupling | API layer and AI Agent worker can scale independently |
| Reliability | Messages can be retried if worker processing fails |
| Throughput | Multiple workers can consume from the same queue |
| Responsiveness | WhatsApp and web chat remain fast at the ingress layer |

RabbitMQ handles **AI chat request orchestration**. **Redis + BullMQ** handles **scheduler jobs** and **rule-related background tasks** separately.

---



## Tech Stack

### AI

| Technology | Purpose |
|------------|---------|
| OpenAI GPT-4o | Primary LLM for agent reasoning and function calling |
| LangChain | Agent orchestration and tool binding |
| Pinecone | Vector database for RAG |
| OpenAI TTS | Optional voice response generation |

### Backend

| Technology | Purpose |
|------------|---------|
| Node.js 20 | Runtime environment |
| Express.js | REST API framework |
| TypeScript | Type-safe backend development |
| Sequelize | ORM and database migrations |
| MySQL 8 | Main relational database |
| RabbitMQ | Message queue for async AI Agent chat processing |
| Redis 7 | Queue backend, cache, and active rule storage |
| BullMQ | Background job and scheduler execution |
| Zod | Request validation |
| Swagger UI | API documentation |
| Winston | Structured logging |

### IoT and Messaging

| Technology | Purpose |
|------------|---------|
| MQTT over TLS | Backend-to-device communication |
| HiveMQ Cloud | Managed MQTT broker |
| MQTT.js | MQTT client library |
| ESP32 | IoT microcontroller |
| DHT11 | Temperature and humidity sensor |
| Relay / LED | Actuator control |
| Baileys | WhatsApp Web integration |

### Infrastructure

| Technology | Purpose |
|------------|---------|
| Docker Compose | Local infrastructure setup |
| MySQL container | Local relational database |
| Redis container | Local queue/cache service |
| RabbitMQ | Local message broker for AI Agent jobs |
| Environment variables | Secret and configuration management |

---

## Background

**Ginem** was developed as part of a final thesis in **Telecommunication Engineering** at **Institut Teknologi Sumatera (Itera)**.

The research focuses on building an LLM-based AI Agent for controlling and monitoring IoT devices using natural language. The system combines AI Agent reasoning, RabbitMQ-based async processing, function calling, RAG, MQTT, ESP32, scheduling, and dynamic rule automation.

---

<p align="center">
  <strong>Ginem — Smart Home IoT, powered by AI.</strong>
</p>

<p align="center">
  Made with ❤️ by <a href="https://github.com/misdarmanto">Misdar Manto</a>
</p>
