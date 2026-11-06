# Arduino RPC Bridge Guide: Arduino Q

This guide details bidirectional Remote Procedure Call (RPC) communication between native Python running on the Linux MPU and the Arduino C++ sketch running on the STM32 MCU. This guide assume you already did the [Quickstart](../README.md).

---

## 1. How the RPC Bridge Works

The **`arduino-router`** daemon coordinates MessagePack-RPC communication between the two processor cores.

* **Arduino Sketch (Server/Provider):** Registers local C++ functions using `Bridge.provide()`.
* **Python Client (Caller):** Connects to the local router socket via `arduino.router_bridge.Bridge` and invokes exposed methods using `bridge.call()` or `bridge.notify()`.

---

## 2. Microcontroller Sketch (`sketch.ino`)

Upload this sketch to the MCU using the CLI (modifying the earlier blink example works)

```cpp
/*
 * Arduino Q - RPC Bridge Example Sketch
 * Requires library: Arduino_RouterBridge
 */

#include <Arduino_RouterBridge.h>

const int LED_PIN = LED_BUILTIN;
const int SENSOR_PIN = A0;

// RPC Callback 1: Control the LED state
bool set_led(bool state) {
  digitalWrite(LED_PIN, state ? HIGH : LOW);
  return state;
}

// RPC Callback 2: Read an analog sensor pin
int read_sensor() {
  int sensorValue = analogRead(SENSOR_PIN);
  return sensorValue;
}

void setup() {
  pinMode(LED_PIN, OUTPUT);
  digitalWrite(LED_PIN, LOW);

  // Initialize the bridge hardware bus
  Bridge.begin();

  // Register methods with the router
  Bridge.provide("set_led", set_led);
  Bridge.provide("read_sensor", read_sensor);
}

void loop() {
  // Keep loop non-blocking; the Bridge handles RPC requests asynchronously
  delay(10);
}
```

> **Tip (`Bridge.provide` vs `Bridge.provide_safe`):**
> * `Bridge.provide(name, func)`: Runs the callback directly in the high-priority RPC listener context.
> * `Bridge.provide_safe(name, func)`: Synchronizes execution with the Arduino `loop()` thread. Use `provide_safe` if your callbacks access hardware or state shared with `loop()`.

---

## 3. Native Python Script (`main.py`)

Create `main.py` inside your project directory on the board's Linux environment:

```python
#!/usr/bin/env python3
"""
Direct Python on Arduino Q via arduino-router-bridge.
Invokes C++ functions on the MCU and receives returned sensor readings.
"""

import time
import sys
from arduino.router_bridge import Bridge

def main():
    print("[*] Connecting to Arduino Router Bridge...")

    # Connect to the local router daemon socket
    with Bridge() as bridge:
        try:
            bridge.connect(timeout=5)
            print("[+] Successfully connected to Arduino Router Bridge.")
        except Exception as e:
            print(f"[-] Failed to connect to arduino-router: {e}", file=sys.stderr)
            print("[-] Check that 'arduino-router' is running: sudo systemctl status arduino-router", file=sys.stderr)
            sys.exit(1)

        print("[*] Starting control loop. Press Ctrl+C to exit.\n")
        led_state = False
        iteration = 0

        try:
            while True:
                iteration += 1
                led_state = not led_state

                # 1. Remote Call: Toggle the LED on the MCU
                try:
                    result = bridge.call("set_led", led_state, timeout=2)
                    status_str = "ON" if result else "OFF"
                    print(f"[{iteration:04d}] RPC -> set_led({led_state}) -> MCU LED is {status_str}")
                except Exception as rpc_err:
                    print(f"[-] RPC Error calling 'set_led': {rpc_err}")

                # 2. Remote Call: Read analog sensor value from the MCU
                try:
                    sensor_reading = bridge.call("read_sensor", timeout=2)
                    print(f"[{iteration:04d}] RPC -> read_sensor()   -> Value: {sensor_reading}")
                except Exception as rpc_err:
                    print(f"[-] RPC Error calling 'read_sensor': {rpc_err}")

                print("-" * 50)
                time.sleep(1.0)

        except KeyboardInterrupt:
            print("\n[*] Stopping control loop...")
        finally:
            # Clean up: ensure the LED is turned off on exit
            try:
                bridge.call("set_led", False, timeout=2)
                print("[+] Turned off LED.")
            except Exception:
                pass
            print("[+] Disconnected cleanly.")

if __name__ == "__main__":
    main()
```

Quick tip: `call()` vs `notify()`

* **`bridge.call(method, *args, timeout=N)`**: Synchronous (blocking) call that waits for the MCU to process the function and return a value. Always pass an explicit `timeout` to prevent indefinite hangs.
* **`bridge.notify(method, *args)`**: Asynchronous (fire-and-forget) notification. Does not wait for a return value. Ideal for non-critical telemetry or fast status updates.

---

## 4. Upload sketch and run python
See the [Quickstart](../README.md) for those commands.

---

[Back to Quickstart (README)](../README.md)

