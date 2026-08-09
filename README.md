# 🧱 블록 레이싱

> **2인 온라인 블록 레이싱 게임**
>
> Unity Client와 C# Dedicated Server를 기반으로 구현한 실시간 멀티플레이 게임 프로젝트입니다.

🚧 **현재 개발 진행 중**

---

## 🎮 Overview

**블록 레이싱**은 두 플레이어가 각각 자신의 Lane에서 블록을 쌓으며 경쟁하는 2인 온라인 멀티플레이 게임입니다.

일반적인 블록 퍼즐 게임의 요소에 **상대방에게 블록을 보내는 공격 시스템**과 **차량의 이동 및 레이싱 요소**를 결합했습니다.

게임의 핵심 로직은 Client가 아닌 **Server에서 authoritative하게 처리**하도록 설계했으며, Client는 Server로부터 전달받은 게임 상태를 기반으로 화면을 렌더링합니다.

### 주요 목표

* 실시간 2인 멀티플레이
* Server Authoritative Game Simulation
* Matchmaking → Room → Game의 명확한 게임 흐름
* Client / Server 간 상태 동기화
* 블록 공격 및 Line Clear 시스템
* 네트워크 환경에서도 일관된 게임 상태 유지

---

## 🏗️ Project Architecture

프로젝트는 **Client / Server / Common(Shared Module)** 구조로 구성되어 있습니다.

Common은 별도의 Repository로 관리되며, 현재는 **Git Submodule로 Client와 Server에 포함되어 공유**됩니다.

```text
                    ┌──────────────────────┐
                    │       Common         │
                    │    Git Submodule     │
                    │                      │
                    │ Packets / Snapshots  │
                    │ Enums / Shared Types │
                    └───────┬───────┬──────┘
                            │       │
                        포함│       │포함
                            │       │
              ┌─────────────▼─┐   ┌─▼────────────────┐
              │ Unity Client  │   │ Dedicated Server │
              │               │   │                  │
              │ Input         │   │ Session          │
              │ Rendering     │   │ MatchMaker       │
              │ UI / Scene    │   │ RoomManager      │
              └───────┬───────┘   │ GameSimulation   │
                      │           └────────┬─────────┘
                      │                    │
                      └──────── TCP ───────┘
```

### Server Authoritative

게임의 핵심 상태와 판정은 Server가 담당합니다.

```text
Client
  │
  │ Input
  ▼
Server
  │
  ├── Game Simulation
  ├── Collision
  ├── Attack
  ├── Line Clear
  ├── Lane Scroll
  └── Game Result
  │
  │ Snapshot
  ▼
Client
  │
  └── Rendering
```

Client에서는 입력을 전달하고 Server의 상태를 화면에 반영하는 구조입니다.

---

# 📦 Repositories

각 영역을 독립적인 Repository로 관리하고 있습니다.

| Repository     | Description                                    |
| -------------- | ---------------------------------------------- |
| 🎮 **Client**  | Unity 기반 게임 클라이언트                              |
| 🖥️ **Server**  | C# / .NET 기반 Dedicated Server                  |
| 📦 **Common**  | Client / Server가 공유하는 Packet, Snapshot, Enum 등 |


```text
Block Racing
│
├── block-racing
│   └── Project Showcase / Documentation
│
├── block-racing-client
│   └── Unity Client
│
├── block-racing-server
│   └── Dedicated Server
│
└── block-racing-common (Submodule)
    └── Shared Packets / Snapshots / Enums
```

---

### 🎮 Client

Unity를 기반으로 구현한 게임 클라이언트입니다.

* Unity 2022.3
* Network Client
* Input 처리
* Snapshot 기반 Rendering 구조
* Matchmaking UI
* Game / Result Scene
* Game State Rendering

→ **[Block Racing Client Repository](https://github.com/rlawodud89/block-racing-client)**

### 🖥️ Server

C# / .NET 기반의 Dedicated Server입니다.

게임의 실제 상태와 판정을 Server에서 관리하는 구조로 구현했습니다.

* TCP Server
* Session Management
* Matchmaking
* Room Management
* Game Simulation
* Collision System
* Attack System
* Line Clear System
* Lane Scroll System
* Game Result

→ **[Block Racing Server Repository](https://github.com/rlawodud89/block-racing-server)**

---

## 📦 Common (Git Submodule)

Client와 Server에서 공통으로 사용하는 데이터는 별도의 Repository로 분리되어 있으며 Git Submodule로 포함되어 있습니다.

* Packet
* Snapshot
* Enum
* Shared Data Types

이 구조를 통해 다음을 관리합니다:

- Client / Server 간 데이터 구조 일관성 유지
- Shared Module 버전 명시적 관리
- 공통 데이터 구조의 독립적인 관리

```text
Client ── Submodule ── Shared Module Repo
Server ── Submodule ── Shared Module Repo
```

→ **[Block Racing Common Repository](https://github.com/rlawodud89/block-racing-common)**

---

# ⚙️ Tech Stack

### Client

* Unity 2022.3.62f3
* C#
* TCP Socket
* Unity Scene Management

### Server

* C#
* .NET 9
* TCP Socket
* Dedicated Server Architecture

### Common (Shared Module)

* C#
* Packet Serialization
* Snapshot System
* Shared Enums / Data Types

### Development

* Git / GitHub
* Git Submodule
* Visual Studio 2022
* Unity
* Windows

---

# 🎯 Core Features

## 1. Login

Client가 Server에 접속한 후 로그인 요청을 보내고 Player Session을 생성합니다.

```text
Client
  │
  │ C_LoginPacket
  ▼
Server
  │
  └── PlayerSession 생성
```

---

## 2. Matchmaking

두 플레이어가 Matchmaking을 요청하면 Server의 `MatchMaker`가 대기 중인 플레이어를 관리하고 매칭을 수행합니다.

```text
Player A ──┐
           │
           ▼
       MatchMaker
           │
           ▼
Player B ──┘
           │
           ▼
        Room 생성
```

Matchmaking 상태는 Player 단위로 관리하며,

```text
None
 ↓
Queued
 ↓
InRoom
```

의 흐름으로 관리합니다.

---

## 3. Room Management

Matchmaking이 완료되면 두 플레이어를 하나의 `Room`에 배치합니다.

Room은 자신의 Game Simulation을 가지고 있으며 Server의 Update Loop에 의해 지속적으로 게임을 진행합니다.

```text
GameManager
    │
    ├── MatchMaker
    │
    └── RoomManager
            │
            ├── Room 1
            ├── Room 2
            └── Room 3
```

---

## 4. Game Simulation

실제 게임 진행은 Server의 `GameSimulation`에서 처리합니다.

현재 주요 Update 흐름은 다음과 같습니다.

```text
ProcessInput
     ↓
UpdatePlayers
     ↓
AttackSystem
     ↓
UpdateBlockSystem
     ↓
LineClear
     ↓
LaneScroll
     ↓
Collision
     ↓
UpdateTick
```

이를 통해 Client의 프레임이나 입력 처리와 독립적으로 게임의 핵심 상태를 Server에서 관리합니다.

---

## 5. Block / Attack System

플레이어가 블록을 배치하거나 Line Clear를 수행하면 상대방에게 공격 블록을 전달할 수 있습니다.

공격 블록은 `FlyingBlock`으로 표현되며 상대방 Lane을 향해 이동합니다.

```text
Player A
   │
   │ Line Clear
   ▼
Attack Piece
   │
   ▼
Flying Block
   │
   │
   ▼
Player B Lane
```

FlyingBlock은 착지하기 전까지 `Lane.Grid`에 직접 반영되지 않으며 별도의 리스트에서 관리합니다.

---

## 6. Collision System

Server에서 블록과 차량의 충돌을 판정합니다.

특히 FlyingBlock의 이동 속도와 Server Tick 사이에서 발생할 수 있는 충돌 누락 문제를 해결하기 위해 **다음 위치에서 충돌이 발생하는지 먼저 검사한 후 이동**하도록 수정했습니다.

```text
현재 위치
    │
    │ 다음 위치 충돌 검사
    ▼
Collision ?
 ┌──┴──┐
Yes    No
 │      │
착지   이동
```

이를 통해 FlyingBlock이 빠르게 이동하면서 충돌 지점을 건너뛰는 문제를 방지했습니다.

---

## 7. Line Clear

Lane의 Grid를 기반으로 완성된 줄을 검사하고 제거합니다.

FlyingBlock은 착지하기 전까지 Grid에 존재하지 않기 때문에, FlyingBlock의 착지 과정과 Line Clear 처리 순서를 고려하여 게임 상태를 갱신하도록 구현했습니다.

또한 현재 게임 규칙에서는 Line Clear 이후 남은 블록을 아래로 떨어뜨려 빈 공간을 채우는 방식이 아니라, **제거된 줄의 위치를 그대로 비워두는 방식**을 사용합니다.

---

## 8. Lane Scroll

시간의 흐름에 따라 Lane이 Scroll되며 플레이어의 레이싱 진행 상황에 영향을 줍니다.

차량의 Speed와 게임 진행 상태를 고려하여 Scroll 속도를 조정하는 방향으로 구현하고 있습니다.

---

## 9. Stun System

차량이 Grid의 블록과 충돌하면 일정 시간 동안 Stun 상태가 적용됩니다.

Stun 상태에서는 차량의 이동 속도가 감소하며 Client에서는 해당 상태를 시각적으로 표현합니다.

```text
Normal
  ↓
Collision
  ↓
Stunned
  ↓
Recovery
```

---

# 🔄 State Synchronization

Server는 게임 상태를 Snapshot 형태로 구성하고 Client에 전달합니다.

```text
GameStateSnapshot
│
├── Tick
│
└── Players
     │
     ├── PlayerSnapshot
     │
     └── LaneSnapshot
           │
           ├── Blocks
           └── FlyingBlocks
```

Client는 전달받은 Snapshot을 기반으로 자신의 화면 상태를 갱신합니다.

이를 통해 게임의 실제 상태와 화면 표현을 분리하고 Server Authoritative 구조를 유지하는 것을 목표로 합니다.

---

# 🧩 Packet Architecture

Client와 Server 간 통신 데이터는 Common Repository에서 관리합니다.

예를 들어 Matchmaking 요청은 다음과 같은 구조로 전달됩니다.

```text
Client
 │
 │ C_MatchRequestPacket
 │
 ▼
Server
 │
 ├── MatchMaker.Register()
 │
 └── Match Found
       │
       ▼
   S_MatchFoundPacket
       │
       ▼
     Client
```

Packet의 ID와 데이터 구조를 Client / Server에서 동일하게 사용하기 위해 Common 영역으로 분리했습니다.

---

# 👨‍💻 Development Focus

이 프로젝트에서 가장 중요하게 생각한 부분은 단순한 게임 기능 구현보다 **멀티플레이 게임의 서버 구조와 상태 관리**입니다.

특히 다음과 같은 문제를 직접 설계하고 해결하는 것을 목표로 개발하고 있습니다.

* Server Authoritative Architecture
* Matchmaking State Management
* Room Lifecycle Management
* Real-time TCP Communication
* Game State Synchronization
* Collision / Simulation Order
* Client / Server Data Sharing
* Network Packet Processing
* Multiplayer Error Handling

이를 통해 Unity 게임 클라이언트뿐만 아니라 **실제 멀티플레이 게임 서버가 어떻게 게임 상태를 관리하고 Client와 동기화하는지**를 경험하는 것을 프로젝트의 주요 목표로 하고 있습니다.

---
