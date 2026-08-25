# 🏭 Automated Material Handling & Sorting System

A PLC-based industrial automation project developed using **Siemens TIA Portal V16** and **Factory I/O 2.4.3**.

The system identifies products by color, routes them to dedicated processing lines, performs automated machining cycles, manages converging products using **FIFO priority logic**, and performs final color-based sorting.

---

## 🛠️ Technologies & Architecture

- **Siemens TIA Portal V16**
- **PLC Ladder Logic (LAD/KOP)**
- **HMI / WinCC**
- **Factory I/O 2.4.3**
- Function Blocks (FBs) and Instance DBs
- Timers, Interlocks, Sequencing
- FIFO Priority Management
- Alarm Management

---

## 🔄 Complete Process Flow

```text
                         AUTOMATED MATERIAL FLOW
                                │
                                ▼
                    ┌─────────────────────────┐
                    │     INFEED CONVEYOR     │
                    │      Raw Material       │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │   VISION COLOR SENSOR   │
                    │  Green / Blue / Gray    │
                    └────────────┬────────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
            GREEN              BLUE               GRAY
              │                  │                  │
              ▼                  ▼                  ▼
     ┌────────────────┐ ┌────────────────┐ ┌────────────────┐
     │ RIGHT CONVEYOR │ │  LEFT CONVEYOR │ │ MIDDLE CONVEYOR│
     └───────┬────────┘ └───────┬────────┘ │    BYPASS      │
             │                  │           └───────┬────────┘
             ▼                  ▼                   │
     ┌────────────────┐ ┌────────────────┐          │
     │ RIGHT ROBOT    │ │  LEFT ROBOT    │          │
     │ Pick & Place   │ │ Pick & Place   │          │
     └───────┬────────┘ └───────┬────────┘          │
             │                  │                   │
             ▼                  ▼                   │
     ┌────────────────┐ ┌────────────────┐          │
     │ RIGHT MACHINING│ │ LEFT MACHINING │          │
     │ Fixture +      │ │ Fixture +      │          │
     │ Clamp + Timer  │ │ Clamp + Timer  │          │
     └───────┬────────┘ └───────┬────────┘          │
             │                  │                   │
             └──────────┬───────┴───────────────────┘
                        │
                        ▼
              ┌──────────────────────┐
              │ PRIORITY / MERGE     │
              │      ROBOT           │
              │ FIFO Queue           │
              └──────────┬───────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │ FINAL CONVEYOR       │
              └──────────┬───────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │ FINAL COLOR SENSORS  │
              └──────────┬───────────┘
                         │
             ┌───────────┼───────────┐
             │           │           │
             ▼           ▼           ▼
        BLUE PUSHER  GREEN PUSHER  GRAY BYPASS
             │           │           │
             └───────────┴─────┬─────┘
                               ▼
                     FINAL STORAGE / OUTPUT
```

---

## ⚙️ Process Stages

### 1. 📥 Infeed & Color Identification

Raw products enter the **main infeed conveyor** and are detected by a vision/color sensor.

The PLC identifies the product and determines its routing:

- **Green → Right Conveyor**
- **Blue → Left Conveyor**
- **Gray → Middle Conveyor / Bypass**

> 📸 Add image: `images/01-infeed-color-detection.png`

---

### 2. 🟢 Green Product — Right Machining Station

Green products are routed to the right processing line.

The **Right Robot** performs the complete machining cycle:

1. Pick the product from the conveyor.
2. Place it in the machining fixture.
3. Close the clamp.
4. Start the machining cycle.
5. Use a timer to represent the machining process.
6. Open the clamp.
7. Return the processed product to the conveyor.

> 📸 Add image: `images/02-green-machining.png`

---

### 3. 🔵 Blue Product — Left Machining Station

Blue products follow the same automated processing sequence through the left line.

The **Left Robot** performs:

**Pick → Fixture → Clamp → Machining → Unclamp → Return**

> 📸 Add image: `images/03-blue-machining.png`

---

### 4. ⚪ Gray Product — Bypass Line

Gray products are routed to the middle conveyor and bypass the machining stations.

They continue through the material flow without machining.

> 📸 Add image: `images/04-gray-bypass.png`

---

### 5. 🔀 FIFO Priority & Merge Management

The processed lines converge at a common transfer section.

A dedicated **Priority Robot** manages the incoming products using a **FIFO queue**.

The PLC tracks the waiting products and determines the transfer order so that products are handled sequentially and conflicts at the merge area are avoided.

The queue is represented by:

```text
Queue0 → First waiting product
Queue1 → Second waiting product
Queue2 → Third waiting product
```

> 📸 Add image: `images/05-priority-fifo.png`

---

### 6. 📦 Final Sorting & Storage

After convergence, the products move through the final conveyor to the sorting section.

Vision sensors identify the product color and activate the corresponding pusher:

- **Blue → Blue Pusher**
- **Green → Green Pusher**
- **Gray → No Pusher / Straight Through**

The products are then directed to their corresponding final storage areas.

> 📸 Add image: `images/06-final-sorting.png`

---

## 🖥️ HMI & Operator Monitoring

The HMI was designed to provide a complete operator interface for monitoring and supervision.

Main HMI screens include:

- **Main:** Overall production-line overview
- **Control:** System operating controls and status
- **Status:** Machine and process status
- **Alarms:** Active alarms and alarm history

> 📸 Add screenshots:
>
> `images/hmi-main.png`  
> `images/hmi-control.png`  
> `images/hmi-status.png`  
> `images/hmi-alarms.png`

---

## 🚨 Alarm Management

The PLC monitors important abnormal conditions and exposes them to the HMI alarm system.

Implemented alarms include:

- **Emergency Stop**
- **Right Robot Cycle Timeout**
- **Left Robot Cycle Timeout**
- **Priority Robot Timeout**

The alarms are consolidated through a central alarm structure and displayed to the operator through the HMI.

> 📸 Add image: `images/hmi-alarms.png`

---

## 🧩 PLC Software Structure

The PLC application was organized into modular Function Blocks:

```text
OB1
 ├── FB_ControlPanel
 ├── FB_InFeed
 ├── FB_TurnTable
 ├── FB_MachiningStation
 ├── FB_PriorityManager
 ├── FB_FinalSorting
```

The project uses **Ladder Logic**, modular FB architecture, Instance DBs, timers, interlocks, and dedicated monitoring variables for the HMI.

> 📸 Add image: `images/tia-portal-architecture.png`

---

## 📁 Repository Structure

```text
├── README.md
├── TIA_Portal/
├── Factory_IO/
└── images/
```

---

## 🎥 Project Demo

A short demonstration video shows the complete operation of the system:

**Infeed → Color Detection → Routing → Machining → FIFO Priority → Final Sorting → HMI Monitoring**

> 🎥 Add your YouTube / LinkedIn / GitHub video link here.

---

## 👨‍💻 Author

**Yousef Abdalla**  
Mechatronics Engineer | Industrial Automation | PLC & HMI
