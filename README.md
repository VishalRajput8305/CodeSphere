<div align="center">

# ☕ CodeCafé

**A browser-based, real-time collaborative code editor powered by Operational Transformation.**

Write code together, live — like Google Docs, but for your IDE.

[![Build](https://img.shields.io/github/actions/workflow/status/mrktsm/codecafe/ci.yml?branch=main&label=build&logo=github)](.)
[![License: MIT](https://img.shields.io/badge/license-MIT-e8a33d.svg)](LICENSE)
[![Java](https://img.shields.io/badge/Java-17-b07219?logo=openjdk&logoColor=white)](.)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.5-6DB33F?logo=springboot&logoColor=white)](.)
[![React](https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=black)](.)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.6-3178C6?logo=typescript&logoColor=white)](.)
[![Redis](https://img.shields.io/badge/Redis-7-DC382D?logo=redis&logoColor=white)](.)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)](.)
 · [Request Feature](../../issues)


</div>

---

## 📖 Overview

**CodeCafé** is a full-stack, multiplayer code editor that runs entirely in the browser. Multiple users can join the same session and edit the same file simultaneously — every keystroke is transformed, ordered, and broadcast in real time using a custom **Operational Transformation (OT)** engine, so every participant always converges to the exact same document state, regardless of network latency or the order operations arrive in.

It also ships with a live HTML/CSS/JS preview pane, an in-browser terminal, multi-language code execution, and session chat — giving it the feel of a lightweight, collaborative VS Code running in a tab.

---

## 🏗️ System Architecture

<p align="center">
  <img src="./assets/architecture-3d.svg" alt="CodeCafé 3D system architecture diagram" width="100%" />
</p>

<p align="center"><sub>Multiple clients connect over a persistent WebSocket (STOMP/SockJS) to a Spring Boot server, which serializes and transforms concurrent edits before broadcasting the resolved operation back to every participant. Document state and revision history live in Redis; code execution is proxied to an external sandboxed runner.</sub></p>

---

## ✨ Features

| | |
|---|---|
| 🔴 **Real-Time Collaboration** | Concurrent multi-cursor editing with deterministic OT-based conflict resolution |
| 🖥️ **Monaco Editor** | The same editor engine that powers VS Code — syntax highlighting, autocomplete, diagnostics |
| 🌐 **Live Preview** | Instantly renders HTML, CSS & JavaScript in a sandboxed `iframe` as you type |
| ▶️ **Multi-Language Execution** | Run code in-browser via a sandboxed remote execution engine, with output streamed to an embedded terminal |
| 💬 **Session Chat & Presence** | See who's in the room, live cursor colors, and chat without leaving the editor |
| 📦 **Zero Install** | 100% browser-based — share a link, start coding together |

---

## 🧰 Tech Stack

### Client

| Category | Technology |
|---|---|
| Framework | React 18 + JavaScript |
| Build tool | Vite |
| Editor | Monaco Editor (`@monaco-editor/react`) |
| State management | Zustand |
| Styling | Tailwind CSS + Radix UI |
| Realtime transport | STOMP.js over SockJS (WebSocket) |
| Terminal | Xterm.js |
| HTTP client | Axios |
| Animation | Framer Motion |
| Testing | Jest + React Testing Library |

### Server

| Category | Technology |
|---|---|
| Framework | Spring Boot 3.5 (Java 17) |
| Realtime | Spring WebSocket + STOMP message broker |
| Collaboration engine | Custom Operational Transformation implementation |
| State store | Redis (Spring Data Redis / Jedis), atomic updates via Lua scripts |
| Serialization | Jackson |
| Monitoring | Spring Boot Actuator |

### Infrastructure

| Category | Technology |
|---|---|
| Containerization | Docker & Docker Compose |
| CI/CD | GitHub Actions (test → build → deploy) |
| Hosting | Client: Vercel · Server: AWS EC2 · Cache: AWS ElastiCache (Redis) |

---

## 🔄 How the Collaboration Engine Works

1. A user types → the Monaco adapter emits a local `insert`/`delete` operation.
2. The client sends the operation to the server tagged with the **revision** it was based on.
3. The server checks whether any concurrent operations were committed after that revision. If so, it **transforms** the incoming operation against each of them in order, preserving everyone's intent.
4. The transformed operation is applied to the canonical document and **atomically** persisted to Redis (content + history) via a Lua script.
5. The resolved operation is broadcast to every subscriber of that document; the original sender receives a lightweight ACK.
6. Each remote client transforms its own pending/local operations against the incoming one before applying it — so cursors, selections, and text all stay in sync.

This is the same class of algorithm that powers Google Docs and Etherpad.

---

## 🚀 Getting Started

### Prerequisites

- [Docker](https://www.docker.com/) & Docker Compose
- *(for manual setup)* Node.js 18+, Java 17+, Maven, Redis

### Quick Start (Docker)

```bash
git clone https://github.com/mrktsm/codecafe.git
cd codecafe
docker-compose up
```

The app will be available at **http://localhost:80**.

### Manual Setup

<details>
<summary>Click to expand</summary>

**1. Start Redis**

```bash
docker run -p 6379:6379 redis:7-alpine
```

**2. Run the server**

```bash
cd server
./mvnw spring-boot:run
```

**3. Run the client**

```bash
cd client
npm install
npm run dev
```

</details>

For more detailed setup and contribution guidelines, see [CONTRIBUTING.md](CONTRIBUTING.md).

---

## 📁 Project Structure

```
codecafe/
├── client/                  # React + TypeScript frontend
│   └── src/
│       ├── components/      # UI components (Editor, Sidebar, Terminal, WebView...)
│       ├── hooks/           # useCollaborationSession, useSessionManager...
│       ├── ot/               # Client-side Operational Transformation engine
│       └── store/            # Zustand file/editor state
│
├── server/                  # Spring Boot backend
│   └── src/main/java/com/codecafe/backend/
│       ├── controller/       # OtController, ChatController, SessionController...
│       ├── service/          # OtService, SessionRegistryService
│       ├── config/           # WebSocket & Redis configuration
│       └── util/             # OtUtils — server-side transform logic
│
├── docker-compose.yml       # client + server + redis
└── .github/workflows/       # CI (test) and CD (deploy) pipelines
```

---

## 🔌 Key API & WebSocket Endpoints

| Type | Endpoint | Description |
|---|---|---|
| `STOMP` | `/app/operation` | Submit a text operation for OT processing |
| `STOMP` | `/app/get-document-state` | Request current document content + revision |
| `STOMP` | `/app/chat` | Send a session chat message |
| `REST POST` | `/api/sessions/create` | Create a new collaboration session |
| `REST POST` | `/api/execute` | Execute code (proxied to sandboxed runner) |
| `SUB` | `/topic/sessions/{id}/operations/document/{docId}` | Live operation broadcast stream |

---

## 🗺️ Roadmap

- [ ] User authentication & persistent projects
- [ ] Integrated voice/text chat
- [ ] Session rewind & history playback
- [ ] Expanded language support & tooling

---

## 🤝 Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.

## 📄 License

Distributed under the [MIT License](LICENSE).

---

<div align="center">
<sub>Built with ☕ by developers who like watching their edits appear before they finish typing.</sub>
</div>
T)
