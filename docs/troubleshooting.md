# Troubleshooting and Incident Log

**Author:** Garrett Hebert · FAA Part 107 remote pilot · [LinkedIn](https://www.linkedin.com/in/garrett-hebert-285506126/) · [GitHub](https://github.com/Stayoutdoors)


Every fault below happened on this build. Each entry records the symptom, how it was diagnosed, and the fix.

## Build issues

### Flight controller and radio control

| Symptom | Cause | Fix |
|---|---|---|
| QGC: "RC not found"; radio page shows no stick movement | ELRS receiver TX/RX wires run straight through instead of crossing | `SERIAL2_OPTIONS = 8` (swap TX/RX in software). If the cable is ever rewired, set it back to 0 |
| CH5 never moves on the radio page | ELRS reserves CH5 as its arm channel | Flight modes moved to CH6 (`FLTMODE_CH = 6`); arm switch on CH5 (`RC5_OPTION = 153`) |
| A switch works backwards | Switch direction on the transmitter | Invert the channel on the transmitter (Outputs), not with RC parameters |
| GPS not detected, LED dark | DroneCAN not enabled | `CAN_P1_DRIVER = 1`, reboot; then `CAN_D1_PROTOCOL = 1` and `GPS1_TYPE = 9`, reboot |
| "Compass not calibrated" right after adding the GPS | New GPS compass landed in priority slot 2 | Swap `COMPASS_PRIO1_ID` and `COMPASS_PRIO2_ID`, reboot, recalibrate |
| "Check mag field" pre-arm indoors | Building wiring distorts the field | Bench-only: `ARMING_MAGTHRESH = 0` with props off; restore to 100 before flight |
| Battery % resets to 100 after reboot | ArduPilot estimates capacity from voltage at boot | Always fly on a fully charged pack |

**Motors don't spin in the motor test, but QGC reports "starting/finished motor test."**
Diagnosis: MAVLink Inspector → `SERVO_OUTPUT_RAW`. `servo1_raw` rose during the test, so the Pixhawk was commanding the motors and the signal wasn't reaching the ESCs. Cause: ESC plugs upside down on MAIN 1–4. Fix: white (signal) wire on top. BLHeli_S ESCs play their startup tune with or without a signal, so the tune alone doesn't prove the signal is connected.

### Power

**Jetson won't boot on a 5 V UBEC.** The Jetson Orin Nano's barrel input needs 9–20 V. Final power path: 4S pack → spare XT30 on the PDB → 5.5 × 2.5 mm center-positive barrel pigtail with a parallel capacitor near the plug, no regulator. A full 4S pack (16.8 V) and an empty one (14.0 V) both stay inside the Jetson's input range; about 15.3 V measured at the plug.

> **Warning:** This only works on 4S. A 6S pack reaches 25.2 V and needs a regulator.

### Jetson and MAVROS

| Symptom | Cause | Fix |
|---|---|---|
| `/mavros/state` shows `connected: false` | Ethernet down or Pixhawk `NET_*` parameters wrong | `ping` the Pixhawk; check the wired connection profile and `NET_*` parameters |
| Topics exist but sensor, battery and GPS data are blank | ArduPilot only sends heartbeat and status text until streams are requested | Call `/mavros/set_stream_rate` (all streams, 10 Hz). Automatic stream setup at MAVROS start is an open item |
| MAVROS stops when the SSH session closes | Process tied to the session | Runs as an enabled systemd service |
| SSH over Wi-Fi: "No route to host" after idle | Wi-Fi power saving | Wi-Fi power save turned off |
| NVMe health shows many unsafe shutdowns | Power pulled without shutting down | `sudo shutdown -h now` before unplugging the flight battery |

### Ground station

| Symptom | Cause | Fix |
|---|---|---|
| "Vehicle did not return the full parameter list" | Slow SiK link after reboot | Parameters → Tools → Refresh |
| QGC can't see the SiK radio | User not in `dialout` | `sudo usermod -aG dialout $USER`, log out and in |
| Mission Planner freezes at the splash screen | Mono on Ubuntu 24.04 | Unresolved; QGroundControl used instead |

---

## Incident 2026-11-08: Loss of thrust after mode switch

**Summary.** During flight 6 of the day, the aircraft lost all thrust in a hover at about 3 m after the mode switch was moved to Stabilize with the throttle stick at the bottom. It fell and flipped on landing. No one was hurt.

**Timeline** (from dataflash log 12):

| Time | Event |
|---|---|
| Takeoff | Took off from QGC in **Guided**, throttle stick left at the bottom (Guided ignores the stick) |
| — | Hovering at about 3 m |
| 110.77 s | CH6 switched to **Stabilize** with throttle at 1003 µs |
| 110.77 s | Stabilize passes the stick straight to the motors: commanded output drops to zero and the aircraft falls |
| +0.6 s | Back in Loiter, too late to recover from 3 m |
| Impact | Jetson came off its mounts, battery ejected, aircraft flipped |

**Root cause.** Pilot procedure. Guided, AltHold and Loiter hold altitude regardless of a low throttle stick; Stabilize does not. Switching into Stabilize with the stick at the bottom commands zero thrust.

**Corrective actions.**
- Procedure added to the [field checklist](field-checklist.md) and [test protocol](test-protocol.md): bring the throttle stick to mid **before** switching to Stabilize from any altitude-holding or autonomous mode.
- Post-incident inspection, tracked below.

**Post-incident checks.**

| Item | Status |
|---|---|
| Jetson: CPU, RAM, CUDA, NVMe health, Ethernet to Pixhawk, MAVROS | Passed |
| Pixhawk: "Internal errors 0x4000 spi_fail" pre-arm error (an onboard SPI sensor stopped responding) | Cleared after a full power cycle; all sensors healthy in `SYS_STATUS`. Treated as a one-time error latched by the impact; watching on the next startups |
| Accelerometer recalibration | Pending |
| Props-off motor test | Pending |
| Props, arms, frame, landing gear, Jetson standoffs and power pigtail | Pending physical inspection |
| Ejected battery: check for puffing or dents | Passed (2026-11-24): no damage |
| Outdoor compass and GPS fix check | Pending |

The full [test protocol](test-protocol.md) is repeated before the next mission.
