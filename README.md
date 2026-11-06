# Direct Python on Arduino Q

A streamlined quickstart guide to running native Python scripts directly in the terminal on the **Arduino Q** series (Arduino UNO Q and VENTUNO Q), bypassing Docker containerization and communicating directly with the microcontroller via the MessagePack-RPC **Router Bridge**.

Based on community insights from [Arduino Forum Topic #1436102](https://forum.arduino.cc/t/uno-q-possible-to-connect-python-not-containerized-with-arduino/1436102/8).

---

## ⚡ Quickstart

### 1. Connect via SSH
Open your workstation's terminal and log in to the board's Linux environment:

```bash
ssh arduino@arduino-q.local
# Or: ssh arduino@<BOARD_IP>
```

---

### 2. Set Up a Python Environment
Use **`uv`** (Astral) to create an isolated environment without encountering Debian PEP 668 system package errors:

```bash
# Install uv if not present
if ! command -v uv &> /dev/null; then
    curl -LsSf https://astral.sh/uv/install.sh | sh
    source $HOME/.local/bin/env  
fi
```

> **Note:** You may need to restart your shell or re-source your environment for `uv` to become available.

---

### 3. Create Project Directory & Virtual Environment
Initialize a bare project directory using Python 3.13:

```bash
uv init --python 3.13 --bare --no-package ~/bridgetest
cd ~/bridgetest
```

---

### 4. Install the App Bricks Library
Install the `arduino_app_bricks` package into your virtual environment:

```bash
uv add https://github.com/arduino/app-bricks-py/releases/download/bricks%2F0.13.2/arduino_app_bricks-0.13.2-py3-none-any.whl
```

---

### 5. Install numpy and watchdog 
Install the `arduino_app_bricks` package into your virtual environment:

```bash
uv add numpy watchdog 
```

---

### 6. Download the Blink Example
Download both the Python script and the MCU sketch files:

```bash
wget --directory-prefix=python https://raw.githubusercontent.com/arduino/app-bricks-examples/refs/tags/0.13.2/inspirational/common/blink/python/main.py
wget --directory-prefix=sketch https://raw.githubusercontent.com/arduino/app-bricks-examples/refs/tags/0.13.2/inspirational/common/blink/sketch/sketch.ino https://raw.githubusercontent.com/arduino/app-bricks-examples/refs/tags/0.13.2/inspirational/common/blink/sketch/sketch.yaml
```

---

### 7. Upload the Sketch to MCU

For **Uno Q**:
```bash
arduino-cli compile --fqbn arduino:zephyr:unoq --upload sketch/
```

For **Ventuno Q**:
```bash
arduino-cli compile --fqbn arduino:zephyr:ventunoq --upload sketch/
```

---

### 8. Run the Script
Execute the Python script natively using `uv`:

```bash
sudo $(which uv) run python/main.py
```
> **Note: Running a Python script with sudo introduces significant security risk**

---

## 📚 In-Depth Documentation

For detailed architectural diagrams, advanced code examples, and production deployment instructions, explore the dedicated documentation guides:

| Guide | Description |
| :--- | :--- |
| **[Architecture & Design](docs/architecture.md)** | Hardware dual-brain architecture (MPU + MCU), native vs containerized App Lab trade-offs, and the MessagePack-RPC router. |
| **[Environment & Package Setup](docs/environment-setup.md)** | SSH login, managing Debian PEP 668 restrictions, Astral `uv` setup, and standard `python3 -m venv` fallback. |
| **[Bidirectional RPC Bridge Guide](docs/rpc-bridge.md)** | Full bidirectional C++ sketch and Python scripts, analog sensor reading, error handling, and `provide` vs `provide_safe`. |
| **[Process Lifecycle & Troubleshooting](docs/background-and-services.md)** | Keeping scripts running in the background with `tmux` and `nohup`, managing `arduino-router` system services, and common troubleshooting steps. |

---

## 🔗 References & Credits

* [Arduino Forum Topic #1436102](https://forum.arduino.cc/t/uno-q-possible-to-connect-python-not-containerized-with-arduino/1436102/8): Community discussion and official advice by `ptillisch` on running native Python.
* [arduino/arduino-router-bridge-py](https://github.com/arduino/arduino-router-bridge-py): Official Python client library for the Arduino Router Bridge.
* [arduino/app-bricks-examples](https://github.com/arduino/app-bricks-examples): Official Arduino App Lab starter examples repository.
