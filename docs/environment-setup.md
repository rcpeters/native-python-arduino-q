# Environment & Package Setup: Arduino Q

This guide walks you through connecting to your **Arduino Q** terminal via SSH and configuring isolated Python environments using modern best practices.

---

## 1. Connecting via SSH

Ensure your Arduino Q board is powered and connected to the same local network (Wi-Fi or Ethernet) as your development machine.

Open your local terminal and connect:

```bash
# Connect using the board's mDNS hostname
ssh arduino@arduino-q.local

# Or connect using the board's static or DHCP IP address
ssh arduino@<YOUR_BOARD_IP>
```

> **Note:** The default username on Debian-based Arduino Q images is `arduino`. Enter your board's configured password when prompted.

---

## 2. Managing Python on Debian (PEP 668)

Modern Debian Linux marks system Python installations as **externally managed** ([PEP 668](https://peps.python.org/pep-0668/)). Running global `pip install` commands will fail with:

```text
error: externally-managed-environment
× This environment is externally managed
╰─> To install Python packages system-wide, try apt install...
```

**Rule:** Always isolate your projects inside virtual environments. This keeps system administration scripts intact and prevents package version conflicts.

---

## 3. Recommended Workflow: Astral uv

**`uv`** is an ultra-fast Python package and project manager developed by Astral. It is the toolchain used internally by Arduino App Lab.

### Step 1: Verify or Install uv
Check if `uv` is installed on your board. If not, install it using the official installer:

```bash
if ! command -v uv &> /dev/null; then
    curl -LsSf https://astral.sh/uv/install.sh | sh
    source ~/.cargo/env
fi
```

### Step 2: Initialize Your Project Directory
```bash
mkdir -p ~/arduino-python-bridge
cd ~/arduino-python-bridge
```

### Step 3: Create a Virtual Environment
```bash
uv venv
```
This generates a fast, isolated `.venv` directory.

### Step 4: Activate the Virtual Environment
```bash
source .venv/bin/activate
```

### Step 5: Install Dependencies
Install the official Arduino Router Bridge client package:
```bash
uv pip install arduino-router-bridge
```

---

## 4. Alternative Workflow: Standard python3 venv

If you prefer using only the Python standard library without third-party utilities:

```bash
# Create and enter your workspace
mkdir -p ~/arduino-python-bridge
cd ~/arduino-python-bridge

# Create standard virtual environment
python3 -m venv .venv

# Activate environment
source .venv/bin/activate

# Upgrade pip and install package
pip install --upgrade pip
pip install arduino-router-bridge
```

---

[Back to Quickstart (README)](../README.md)

