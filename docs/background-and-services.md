# Process Lifecycle & Troubleshooting: Arduino Q

This guide covers running native Python scripts interactively, keeping them persistent across terminal disconnects using `tmux` or `nohup`, managing the underlying router service, and resolving common errors.

---

## 1. Interactive Execution

To run and monitor your script directly in your terminal:

```bash
cd ~/arduino-python-bridge
source .venv/bin/activate
python3 main.py

# Or execute directly with uv without manual activation:
# uv run main.py
```

---

## 2. Background Execution with tmux (Recommended)

When connecting via SSH, closing the terminal or losing your Wi-Fi connection terminates running processes. `tmux` keeps your scripts running inside a detached session.

### Step 1: Install tmux (if needed)
```bash
sudo apt update && sudo apt install -y tmux
```

### Step 2: Start a New Named Session
```bash
tmux new -s bridge-session
```

### Step 3: Run Your Python Script
```bash
cd ~/arduino-python-bridge
source .venv/bin/activate
python3 main.py
```

### Step 4: Detach From the Session
Press **`Ctrl + B`**, release both keys, then press **`D`**. You are returned to your standard shell, and the script continues running in the background.

### Step 5: Reattach to the Session Later
```bash
tmux attach -t bridge-session
```

### Step 6: Terminate the Session
Inside the tmux session, press `Ctrl + C` to stop the script, then type `exit`, or run:
```bash
tmux kill-session -t bridge-session
```

---

## 3. Background Execution with nohup

For simple fire-and-forget background execution where output is piped to a file:

```bash
cd ~/arduino-python-bridge
source .venv/bin/activate

# Launch in background redirecting stdout & stderr
nohup python3 main.py > bridge.log 2>&1 &

# Monitor logs in real time
tail -f bridge.log

# Check running process
pgrep -fl main.py

# Stop the background process
pkill -f main.py
```

---

## 4. Managing the Arduino Router Service

The `arduino-router` daemon runs as a `systemd` service on Debian to route RPC calls between the Linux MPU and the MCU.

- **Check service status:**
  ```bash
  sudo systemctl status arduino-router
  ```

- **Restart the router daemon:**
  ```bash
  sudo systemctl restart arduino-router
  ```

- **View live router logs:**
  ```bash
  journalctl -u arduino-router -f
  ```

- **Inspect the communication socket:**
  ```bash
  ls -la /var/run/arduino-router/
  ```

---

## 5. Troubleshooting Reference

| Problem | Root Cause | Solution |
| :--- | :--- | :--- |
| **`ConnectionRefusedError`** or connection timeout | `arduino-router` service is stopped or socket is inaccessible | Run `sudo systemctl restart arduino-router` and verify status with `systemctl status arduino-router`. |
| **`MethodNotFound` RPC Error** | The method name in Python doesn't match the sketch registration | Verify that the Arduino sketch called `Bridge.provide("name", ...)` in `setup()` and matches the string passed to `bridge.call("name")`. |
| **Script hangs during `bridge.call()`** | MCU is blocked in a long delay or loop | Keep the sketch `loop()` non-blocking and always specify an explicit `timeout` (e.g., `bridge.call("func", timeout=2)`). |
| **`externally-managed-environment` error** | Attempting global `pip install` on Debian Linux | Always activate your virtual environment (`source .venv/bin/activate`) or use `uv pip install`. |
| **Permission Denied on Serial / Ports** | Missing permissions to host device nodes | Add the `arduino` user to the dialout group: `sudo usermod -aG dialout arduino`, then log out and log back in. |

---

[Back to Quickstart (README)](../README.md)

