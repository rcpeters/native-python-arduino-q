# Architecture & System Design: Arduino Q

This document details the hardware architecture, software stack, and design rationale for running native Python scripts on the **Arduino Q** series (such as Arduino UNO Q and VENTUNO Q).

---

## 1. Dual-Processor ("Dual-Brain") Architecture

The Arduino Q family pairs high-performance application processing with real-time microcontroller hardware:

```
+-------------------------------------------------------------+
|                        Arduino Q                            |
|                                                             |
|  +-------------------------------------------------------+  |
|  |                   Linux MPU (Debian)                  |  |
|  |                                                       |  |
|  |  +---------------------+     +---------------------+  |  |
|  |  |  Native Python App  | <-> |   arduino-router    |  |  |
|  |  |  (uv / venv)        |     |   daemon service    |  |  |
|  |  +---------------------+     +----------+----------+  |  |
|  +-----------------------------------------|-------------+  |
|                                            | Internal Bus   |
|  +-----------------------------------------|-------------+  |
|  |  +---------------------+                |             |  |
|  |  |   Arduino Sketch    | <--------------+             |  |
|  |  |  (Arduino_Router-   |                              |  |
|  |  |       Bridge)       |                              |  |
|  |  +----------+----------+                              |  |
|  |             | GPIO / Hardware Peripherals             |  |
|  |             v                                         |  |
|  |     STM32 Real-Time MCU                               |  |
|  +-------------------------------------------------------+  |
+-------------------------------------------------------------+
```

### Components
1. **Linux MPU (Microprocessor Unit):**
   - High-performance SoC running a full Debian Linux operating system.
   - Executes Python, networking stacks, file storage, complex data pipelines, and terminal utilities.
2. **Real-Time MCU (Microcontroller Unit):**
   - STM32 microcontroller running traditional Arduino C++ sketches or Zephyr RTOS.
   - Dedicated to hard real-time tasks, microsecond-accurate timing, PWM, analog-to-digital conversion (ADC), and hardware interrupts.

---

## 2. Why Run Python Natively (Outside Containers)?

By default, Arduino App Lab encapsulates Python applications inside Docker containers. While containerization provides a sandboxed environment for packaged apps, native terminal execution offers critical advantages:

* **Direct Host Hardware Access:** Container sandboxing isolates host serial devices (`/dev/tty*`), USB endpoints, and raw device nodes unless explicitly mounted. Running natively gives your Python scripts direct access to all board interfaces.
* **Lower Memory and CPU Footprint:** Bypassing Docker container virtualization layers conserves RAM and CPU cycles on embedded hardware.
* **Standard Linux Development Workflow:** Develop, run, test, and profile code using familiar terminal tools (`tmux`, `systemctl`, `gdb`, Python debuggers, interactive REPLs).
* **Arbitrary Native Packages:** Compile and install any C/C++ Python wheels (NumPy, OpenCV, PyTorch, PySerial) without configuring custom Dockerfiles or rebuilding images.

---

## 3. Communication Layer: The Arduino Router

Inter-processor communication between the Linux MPU and the MCU is managed by the **`arduino-router`** daemon:

* **Protocol:** MessagePack-RPC (Remote Procedure Call).
* **Topology:** Star topology where multiple clients (Python scripts, C++ apps, web sockets) connect to the central router over Unix domain sockets or internal serial buses.
* **Method Registration:** The MCU sketch exposes named functions via `Bridge.provide("method_name", handler)`. When a Python client invokes `bridge.call("method_name")`, `arduino-router` marshals the arguments, forwards the call across the internal bus, and returns the response.

---

[Back to Quickstart (README)](../README.md)

