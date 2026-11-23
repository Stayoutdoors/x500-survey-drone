# X500 Survey UAS: Operator's Manual

Reference manual for the Holybro X500 V2 survey UAS: what each component is, how it is configured on this aircraft, and why. Configuration reflects the aircraft as of November 2026.

**Related documents:** [Field checklist](field-checklist.md) · [Test flight protocol](test-protocol.md) · [Troubleshooting and incident log](troubleshooting.md) · [Parameter file](../config/X500_2026-11-06_Params.params)

## Contents

1. [System overview](#1-system-overview)
2. [Batteries and charger](#2-batteries-and-charger)
3. [Airframe and power distribution](#3-airframe-and-power-distribution)
4. [Flight controller: Pixhawk 6X and ArduCopter](#4-flight-controller-pixhawk-6x-and-arducopter)
5. [Sensors and GPS](#5-sensors-and-gps)
6. [Radio control: ExpressLRS](#6-radio-control-expresslrs)
7. [Telemetry radio: SiK 915 MHz](#7-telemetry-radio-sik-915-mhz)
8. [Ground station](#8-ground-station)
9. [Jetson Orin Nano companion computer](#9-jetson-orin-nano-companion-computer)
10. [Software stack](#10-software-stack)
11. [Onboard link reference](#11-onboard-link-reference)

---

## 1. System overview

A Pixhawk-controlled quadcopter with a Jetson companion computer, flown from an ExpressLRS transmitter and monitored from a Linux ground station running QGroundControl. Two independent radio links reach the aircraft: **ELRS at 2.4 GHz** for stick control and **SiK at 915 MHz** for MAVLink telemetry. Losing either one alone does not leave the pilot without control.

### Component roster

| Component | Model | Role |
|---|---|---|
| Airframe | Holybro X500 V2 ARF (500 mm, carbon fiber) | Frame, motors, ESCs, PDB |
| Motors | Holybro 2216 × 4 (kit) | Thrust |
| ESCs | BLHeli_S 20A × 4 (kit) | Motor drive |
| Props | 1045 (10 × 4.5 in) | Thrust |
| Flight controller | Holybro Pixhawk 6X | Stabilization, navigation, failsafes |
| Firmware | ArduCopter 4.7.0 | Flight software |
| GPS | Holybro H-RTK F9P Rover, DroneCAN | Position, RTK-capable |
| Companion computer | NVIDIA Jetson Orin Nano 8 GB dev kit, 1 TB NVMe | Onboard autonomy and perception |
| RC transmitter | RadioMaster Boxer, internal ELRS | Pilot control |
| RC receiver | RadioMaster RP4TD-M (ELRS 2.4 GHz, true diversity) | Pilot control, on TELEM2 |
| Telemetry radio | Holybro SiK V3, 915 MHz (pair) | MAVLink to QGC, on TELEM1 |
| Ground station | Linux laptop, Ubuntu 24.04, QGroundControl 5.1 | Setup, monitoring, mission planning |
| Batteries | Tattu G-Tech 5200 mAh 4S 35C, XT60 × 4 | Flight power |
| Charger | Gens Ace IMARS D300 (dual channel) | Charging and storage |

### How it connects

The Pixhawk is the hub: pilot sticks arrive on TELEM2 over ELRS, QGC telemetry on TELEM1 over SiK, the GPS on CAN1, and the Jetson on the Ethernet port. The ground station reaches the Jetson directly over Wi-Fi or a USB-C cable, not through the Pixhawk.

---

## 2. Batteries and charger

Judge a pack by its **voltage**, not the percentage in QGC: ArduPilot's percentage resets to 100% every time the Pixhawk reboots.

### Flight battery: Tattu G-Tech 5200 mAh 4S

| Spec | Value |
|---|---|
| Chemistry | LiPo, 4S1P |
| Nominal voltage | 14.8 V |
| Capacity | 5200 mAh (about 77 Wh) |
| Discharge | 35C continuous, 70C burst |
| Weight | about 436 g |
| Connectors | XT60 main, JST-XH balance |
| Fleet | 4 packs |

### Voltage reference (4S, resting)

| Pack voltage | Per cell | State | Action |
|---|---|---|---|
| 16.8 V | 4.20 V | Full | Ready to fly |
| 16.6 V | 4.15 V | Nearly full | Fine to fly |
| 15.2 V | 3.80 V | Storage charge | Charge before flying |
| 14.8 V | 3.70 V | About half | Land soon |
| 14.4 V | 3.60 V | Low | Land now |
| 14.0 V | 3.50 V | Critical | Never fly this low |

Under load the pack reads 0.3–0.5 V lower and recovers after landing.

### Flight time

Holybro rates the bare X500 V2 at about 18 minutes of hover on 5000 mAh with no payload. This aircraft also carries the Jetson, RTK GPS and radios, so flights are planned at 10 minutes until hover current is measured, then set to keep about 25% reserve.

### Charger: Gens Ace IMARS D300

| Setting | Value | Why |
|---|---|---|
| Chemistry | LiPo (or G-Tech auto) | Standard LiPo, not LiHV |
| Cells | 4S | 16.8 V full |
| Mode | Balance charge | Keeps all four cells equal |
| Current | 5.2 A (1C) | Standard rate, longest pack life |
| Two packs at once | 2 × about 87 W | Well inside the 300 W AC limit |

### Storage and safety

- Storage-charge any pack idle more than two days (3.80 V per cell).
- Never charge unattended; charge on a non-flammable surface or in a LiPo bag.
- Retire a pack that is puffed, dented, hot after a normal flight, or has a cell more than 0.05 V off after balancing.
- Airline travel: packs under 100 Wh (these are about 77 Wh) go in carry-on only, terminals covered.

> **Warning:** A damaged LiPo can ignite without warning. Move a swelling or smoking pack outdoors onto concrete or sand. Do not use water on a lithium fire.

---

## 3. Airframe and power distribution

### Frame specifications

| Spec | Value |
|---|---|
| Wheelbase | 500 mm |
| Body plates | 144 × 144 mm carbon fiber, 2 mm |
| Arms | 16 mm carbon fiber tubes |
| Landing gear height | 215 mm |
| Frame weight (kit, no battery) | 610 g |
| Rated payload | 1500 g without battery, at 70% throttle |
| Recommended battery | 4S 3000–5000 mAh, 20C+, XT60 |

The 5200 mAh packs are slightly above Holybro's recommended range; they fit the tray, but check the strap and balance before each flight.

### Motor order (ArduCopter Quad X)

| Motor | Position | Spin | Pixhawk output | Motor test letter |
|---|---|---|---|---|
| 1 | Front right | CCW | MAIN OUT 1 | A |
| 2 | Rear left | CCW | MAIN OUT 2 | C |
| 3 | Front left | CW | MAIN OUT 3 | D |
| 4 | Rear right | CW | MAIN OUT 4 | B |

ESC signal leads plug into MAIN 1–4 with the **white (signal) wire on top**. Motor order and spin direction verified with a props-off motor test.

### Power distribution

The pack feeds a digital I2C power module (INA2xx), which powers the Pixhawk and reports voltage and current, then the PDB, which splits raw pack voltage to the four ESCs, the Jetson and a spare port.

### Jetson power path

| Item | Detail |
|---|---|
| Source | Spare XT30 on the PDB, raw 4S pack voltage |
| Connector | 5.5 × 2.5 mm barrel, center positive |
| Filtering | Capacitor in parallel near the plug |
| Regulator | None |
| Measured voltage | about 15.3 V at the plug |
| Jetson DC input range | 9–20 V |

A full pack (16.8 V) and an empty one (14.0 V) both stay inside the Jetson's input range. A 5 V UBEC was tried first and could not power it.

> **Warning:** Never connect a pack above 4S without adding a regulator. A 6S pack reaches 25.2 V.

### Jetson mounting

- Four M2.5 nylon standoffs, matched lengths at all four corners, with vibration isolation.
- Port edge faces inboard, so a tip-over does not land on the connectors.

### Payload mounting options

| Mount | Use |
|---|---|
| Platform board | Companion computer (Jetson or Raspberry Pi hole patterns) |
| Depth camera mount | Intel RealSense D4xx series |
| Bottom-facing platform | Lidar, optical flow, infrared |
| GPS mount | Mast on the top plate |
| Rails under the body | Gimbal or camera |

---

## 4. Flight controller: Pixhawk 6X and ArduCopter

### Port map

| Port | Connected to | ArduPilot name |
|---|---|---|
| TELEM1 | SiK V3 air radio | SERIAL1 |
| TELEM2 | RP4TD-M ELRS receiver | SERIAL2 |
| CAN1 | H-RTK F9P (DroneCAN) | CAN_P1 / CAN_D1 |
| ETH | Jetson Orin Nano Ethernet | NET_ |
| POWER1 | Power module (INA2xx) | BATT_ |
| MAIN OUT 1–4 | ESC signal leads | SERVO1–4 |
| USB-C | Bench setup and firmware flashing | — |

### Firmware

ArduCopter 4.7.0 is installed; the 4.7.2 bug-fix release is pending. Export a parameter file before any firmware update (QGC → Parameters → Tools → Save to file).

### Parameters changed from default

| Parameter | Value | Purpose |
|---|---|---|
| `FRAME_CLASS` / `FRAME_TYPE` | 1 / 1 | Quad, X layout |
| `BRD_SAFETY_DEFLT` | 0 | No physical safety switch |
| `NET_ENABLE` | 1 | Ethernet on |
| `NET_DHCP` | 0 | Static IP; no router on the cable |
| `NET_P1_TYPE` | 1 (UDP client) | Sends MAVLink to the Jetson |
| `NET_P1_PORT` | 14550 | Port MAVROS listens on |
| `NET_P1_PROTOCOL` | 2 (MAVLink2) | Protocol on that port |
| `CAN_P1_DRIVER` | 1 | Enables CAN1 |
| `CAN_D1_PROTOCOL` | 1 (DroneCAN) | Protocol on CAN1 |
| `GPS1_TYPE` | 9 (DroneCAN) | GPS on CAN |
| `COMPASS_PRIO1_ID` / `COMPASS_PRIO2_ID` | GPS compass / internal | External RM3100 primary, internal BMM150 backup |
| `SERIAL2_PROTOCOL` | 23 (RCIN) | ELRS receiver on TELEM2 |
| `SERIAL2_OPTIONS` | 8 (SwapTXRX) | Corrects crossed TX/RX wiring in software |
| `RSSI_TYPE` | 3 | Link quality from the receiver |
| `RC_OPTIONS` | includes 8192 | ELRS 420 kbaud |
| `FLTMODE_CH` | 6 | Flight modes on CH6 (ELRS sends CH5 as on/off) |
| `FLTMODE1` / `4` / `6` | Stabilize / AltHold / Loiter | Three switch positions |
| `RC5_OPTION` | 153 | Arm/disarm switch on CH5 |
| `RC7_OPTION` | 4 | RTL switch on CH7 |
| `BATT_CAPACITY` | 5200 | Pack size |
| `BATT_LOW_VOLT` / `BATT_FS_LOW_ACT` | 14.0 V / 2 (RTL) | Low-battery failsafe |
| `BATT_CRT_VOLT` / `BATT_FS_CRT_ACT` | 13.6 V / 1 (Land) | Critical-battery failsafe |
| `BATT_LOW_MAH` / `BATT_CRT_MAH` | 1300 / 780 | mAh triggers (25% / 15% remaining) |
| `BATT_ARM_VOLT` | 15.8 | Won't arm on a pack that isn't near full |

Whichever trigger trips first, voltage or mAh, sets off the failsafe. Voltage must stay below the threshold for 10 s (`BATT_LOW_TIMER`), so brief sag under load doesn't trip it. Until the current sensor is checked against a meter, the voltage triggers are trusted over the mAh ones.

### Calibration status

| Calibration | Status |
|---|---|
| Accelerometer and Level Horizon | Done; redo pending after the November 8 incident |
| Compass (GPS RM3100, priority 1; internal BMM150, priority 2) | Done |
| Radio | Done |

### Flight modes (CH6, three-position switch)

| Switch | Slot | Mode | GPS needed |
|---|---|---|---|
| Up | 1 | Stabilize | No |
| Middle | 4 | AltHold | No |
| Down | 6 | Loiter | Yes |

Auto (missions) and Guided (Jetson or QGC control) are started from QGC or the Jetson, not the switch. RTL is on CH7.

> **Warning:** Stabilize passes the throttle stick straight to the motors. Bring the throttle to mid before switching into Stabilize from Guided, AltHold, Loiter or Auto. See the [incident report](troubleshooting.md#incident-2026-11-08-loss-of-thrust-after-mode-switch).

### Arming

- Switch arming on CH5: high arms (if checks pass and throttle is down); low **disarms immediately, even in flight**.
- ArduCopter will not arm in Land, RTL or Auto. Arm in Stabilize or AltHold.
- All pre-arm checks must pass; `ARMING_CHECK` is never disabled.

### Failsafes

| Failsafe | Setting | Verified |
|---|---|---|
| Radio loss | RTL (Land without GPS) | Ground test and in-air test (test protocol Phase 5) |
| Battery low | RTL at 14.0 V or 1,300 mAh left | Configured |
| Battery critical | Land at 13.6 V or 780 mAh left | Configured |
| EKF | Land | Configured |
| Crash and vibration checks | On | Configured |
| GCS (telemetry loss) | Off; the RC failsafe covers loss of control | — |

---

## 5. Sensors and GPS

### Flight controller sensors

| Sensor | Part | Bus | Use |
|---|---|---|---|
| IMU 1–3 | ICM-45686 × 3 | SPI1–SPI3 | Attitude and acceleration, triple-redundant |
| Barometer | BMP388 | I2C | Altitude and climb rate |
| Compass | BMM150 (internal) | I2C | Heading backup |
| Battery monitor | INA2xx | I2C via POWER1 | Voltage and current |

### GPS: Holybro H-RTK F9P Rover (DroneCAN)

| Item | Detail |
|---|---|
| Receiver | u-blox ZED-F9P, multi-band (GPS L1/L2, GLONASS, Galileo, BeiDou) |
| Accuracy | Centimeter-level with RTK corrections; meter-level without |
| Compass | RM3100 (external, low-noise) |
| Interface | DroneCAN over CAN1 |
| Rate | 5 Hz |
| Mounting | On the mast, arrow pointing forward |

**Why DroneCAN:** the GPS uses the CAN bus instead of a serial port, which keeps TELEM2 free for the ELRS receiver and leaves the Jetson on Ethernet.

**Compass priority.** The internal compass was calibrated first, so the GPS compass arrived in priority slot 2 and was moved to slot 1. In QGC Sensors → Compass, Compass 1 should read external, priority 1, rotation None.

**Recalibrate the compass after:** moving the GPS mast; adding or rewiring metal or current-carrying parts near the GPS; a crash or hard landing; persistent "Compass inconsistent" errors or circling in Loiter. Before each flight, compare QGC's heading to a phone compass. Declination is set automatically from GPS position.

### RTK corrections (planned)

| Correction source | How |
|---|---|
| Portable base station | Base on a tripod at the site; QGC relays corrections over SiK |
| NTRIP network | QGC pulls corrections over the internet |

### Planned payload sensors

| Sensor | Purpose | Connects to |
|---|---|---|
| Intel RealSense depth camera | Depth and visual odometry for SLAM | Jetson USB |
| SIYI ZR10 gimbal camera | Inspection imagery, MAVLink control | Pixhawk or Jetson |
| Downward rangefinder or lidar | True height above ground | Pixhawk |

Altitude in QGC is height above the takeoff point (barometer). Over sloping ground, true height needs a downward rangefinder or terrain following.

---

## 6. Radio control: ExpressLRS

RadioMaster Boxer (EdgeTX) with its internal ExpressLRS 2.4 GHz module; RadioMaster RP4TD-M receiver on TELEM2, read by ArduPilot as CRSF.

### Receiver: RP4TD-M

| Spec | Value |
|---|---|
| Protocol | ExpressLRS 2.4 GHz, CRSF output |
| Diversity | True diversity, 2 × SX1281, 2 × T antennas |
| Max packet rate | 500 Hz (F1000) |

| LED | Meaning |
|---|---|
| Solid blue | Bound and connected |
| Slow blink | No transmitter found |
| Fast blink | Bind mode |

**Wiring to TELEM2:** 5V → pin 1, RX → pin 2 (Pixhawk TX), TX → pin 3 (Pixhawk RX), GND → pin 6. On this aircraft the two signal wires are physically crossed; ArduPilot swaps them in software (`SERIAL2_OPTIONS = 8`). If the cable is ever rewired correctly, set it back to 0 at the same time.

### Transmitter model setup

The aircraft has its own model slot with a unique receiver number and **Model Match** on, so the receiver only connects when this aircraft's model is selected. Mode 2 sticks, AETR channel order.

### Channel map

| Channel | Control | ArduPilot function |
|---|---|---|
| CH1 | Aileron stick | Roll |
| CH2 | Elevator stick | Pitch |
| CH3 | Throttle stick | Throttle |
| CH4 | Rudder stick | Yaw |
| CH5 | 2-position switch | Arm/disarm (ELRS sends CH5 as on/off) |
| CH6 | 3-position switch | Flight mode |
| CH7 | 2-position switch | RTL |

### Binding

1. Shut down the Jetson and unplug its barrel jack (binding power-cycles the aircraft three times).
2. Select this aircraft's model on the transmitter.
3. Plug in the battery for about 1 s, unplug; repeat three times and leave it plugged in. The receiver LED fast-blinks.
4. Transmitter: SYS → Tools → ExpressLRS → Bind. The receiver LED goes solid blue.
5. Reconnect the Jetson.

Transmitter module and receiver must run the same major ELRS version.

### Telemetry screens

| Value | Sensor | Why |
|---|---|---|
| Pack voltage | RxBt | Real fuel gauge |
| Flight mode | FM | Confirms the switch; shows failsafe mode changes |
| Link quality | RQly | Below about 70% means near the edge of range |
| Altitude | Alt | Height above takeoff |
| Distance from home | Dist | Visual line of sight check |
| Satellites | Sats | 10 or more before GPS modes |
| Capacity used | Capa | Land at about 3,600–4,000 mAh (70–75% of 5,200) |
| Transmitter power | TPWR | Pinned at maximum means near range limit |

---

## 7. Telemetry radio: SiK 915 MHz

A Holybro SiK V3 pair carries MAVLink between the Pixhawk (TELEM1) and QGroundControl, separate from the ELRS control link. It will also carry RTK corrections.

| Spec | Value |
|---|---|
| Band | 915 MHz (902–928 MHz) |
| Link | Full-duplex, frequency hopping |
| Serial rate | 57,600 baud (factory default) |
| Antennas | RP-SMA, 2 dBi stock |

| LED | Meaning |
|---|---|
| Green, blinking | Searching |
| Green, solid | Linked |
| Red, flashing | Data moving |

- Keep both antennas upright and the ground radio away from the laptop and the Jetson's Wi-Fi.
- At 57,600 baud the full parameter list can arrive incomplete after a reboot: Parameters → Tools → Refresh.
- Give the pair its own Net ID if another SiK pair flies nearby.
- Losing telemetry does not trigger a failsafe; losing the RC link does.

---

## 8. Ground station

| Item | Detail |
|---|---|
| OS | Ubuntu 24.04 LTS (QGroundControl 5.1 AppImages need 24.04 or later) |
| Ground control | QGroundControl 5.1 (primary) |
| SiK ground radio | USB, `/dev/ttyUSB0` |
| Pixhawk USB (bench) | `/dev/ttyACM0` |

### System settings for drone work

| Setting | How | Why |
|---|---|---|
| Serial port access | `sudo usermod -aG dialout $USER`, then log out and in | Lets QGC open the radio and Pixhawk |
| ModemManager off | `sudo systemctl mask --now ModemManager.service` | It grabs USB serial ports |
| Automatic suspend off | Settings → Power | A sleeping laptop drops telemetry mid-flight |

### QGroundControl

| Section | Use |
|---|---|
| Fly view | Map, telemetry, arm and mode controls, start mission |
| Plan view | Build and upload missions |
| Vehicle Configuration | Calibrations, radio, flight modes, failsafes, parameters, firmware |
| Analyze Tools | Log download, MAVLink Inspector, MAVLink Console |

**Motor test (props off):** Vehicle Configuration → Motors, throttle 5–10%. ArduPilot labels motors A–D clockwise from front right (see the motor order table in section 3).

### Survey mission settings

| Setting | Typical value |
|---|---|
| Altitude | Sets ground sample distance; under 400 ft AGL |
| Front overlap | 75–80% |
| Side overlap | 65–75% |
| Speed | Slow enough for sharp images at the shutter speed |
| Mission end | Return to Launch or Land item |

Over uneven terrain, use terrain following to keep a constant height above ground and a consistent GSD. Split areas that need more than one pack with reserve.

---

## 9. Jetson Orin Nano companion computer

Runs the onboard software (ROS 2 and MAVROS today; vision and autonomy later). It talks to the Pixhawk over Ethernet.

### Hardware and software

| Item | Value |
|---|---|
| Board | Jetson Orin Nano 8 GB Developer Kit |
| Storage | 1 TB NVMe SSD |
| OS | Ubuntu 22.04, L4T R36.5.2 |
| JetPack components | CUDA 12.6, cuDNN 9.3, TensorRT 10.3 |
| Robotics | ROS 2 Humble, MAVROS |
| Boot target | Headless (`multi-user.target`) |
| Idle RAM | about 433 MB after trimming (from 698 MB) |

### Configuration

- **Headless and trimmed:** desktop apps removed; ModemManager and Bluetooth disabled; networking packages pinned so autoremove keeps them.
- **SSH:** key-based authentication from the ground station.
- **Wi-Fi power save off**, so SSH doesn't drop after idle.
- **Network access:** Wi-Fi when available, a Jetson-hosted field hotspot otherwise, and USB-C device mode as a wired backup.

### Running MAVROS

MAVROS runs as an enabled systemd service, `mavros.service`. It starts at boot and restarts on crash, so no SSH session or tmux is needed.

```bash
systemctl status mavros --no-pager
journalctl -u mavros -f
sudo systemctl restart mavros
ros2 topic echo /mavros/state --once   # connected: true
```

If `ros2 topic list` shows no `/mavros` topics while the service runs, the discovery cache is stale: `ros2 daemon stop` and retry. Don't start a second MAVROS by hand; two copies can't share UDP 14550.

### Power and shutdown

Always `sudo shutdown -h now` before unplugging the flight battery. Pulling power on a running Jetson can corrupt the SSD.

---

## 10. Software stack

| Layer | Runs on | Software | Talks to |
|---|---|---|---|
| Pilot input | Transmitter | EdgeTX, internal ELRS | RP4TD-M receiver (2.4 GHz CRSF) |
| Flight control | Pixhawk 6X | ArduCopter 4.7.0 | Sensors, ESCs, receiver, SiK, Jetson |
| Ground station | Laptop (Ubuntu 24.04) | QGroundControl 5.1 | Pixhawk over SiK (MAVLink, 915 MHz) |
| Onboard compute | Jetson (L4T R36.5.2) | ROS 2 Humble, MAVROS | Pixhawk over Ethernet (MAVLink over UDP) |
| Applications | Jetson | ROS 2 nodes (future) | MAVROS topics and services |

### MAVLink identities

| System | System ID |
|---|---|
| Pixhawk (autopilot) | 1 |
| QGroundControl | 255 |
| MAVROS on Jetson | 1, component 240 |

### ROS 2 topics and services

| Topic | Contents |
|---|---|
| `/mavros/state` | Connected, armed, mode |
| `/mavros/battery` | Voltage, current, percent |
| `/mavros/global_position/global` | GPS latitude, longitude, altitude |
| `/mavros/local_position/pose` | Position relative to home |
| `/mavros/imu/data` | Attitude and rates |
| `/mavros/set_mode` (service) | Change flight mode |
| `/mavros/cmd/arming` (service) | Arm or disarm |

ArduPilot doesn't stream sensor data to MAVROS until rates are requested:

```bash
ros2 service call /mavros/set_stream_rate mavros_msgs/srv/StreamRate \
  "{stream_id: 0, message_rate: 10, on_off: true}"
```

Requesting streams automatically at MAVROS start is an open item.

### Planned additions

| Addition | Purpose |
|---|---|
| Camera trigger and geotagging | Mapping photos |
| SIYI ZR10 gimbal camera | Inspection and video |
| Depth camera and SLAM | Obstacle awareness, GPS-denied flight |
| Object detection (TensorRT) | Onboard AI |

---

## 11. Onboard link reference

### Radio and wired links

| Link | Hardware | Frequency | Pixhawk port | Protocol |
|---|---|---|---|---|
| RC control | Boxer ELRS → RP4TD-M | 2.4 GHz | TELEM2 (SERIAL2) | CRSF |
| Telemetry | SiK V3 pair | 915 MHz | TELEM1 (SERIAL1) | MAVLink2, 57600 baud |
| GPS and compass | H-RTK F9P | Wired | CAN1 | DroneCAN |

### Ethernet: Pixhawk ↔ Jetson

An isolated two-device link on the aircraft; no router or outside network.

| Item | Pixhawk | Jetson |
|---|---|---|
| Address | 192.168.144.14 | 192.168.144.10/24 (static) |
| Configured by | `NET_ENABLE 1`, `NET_DHCP 0` | NetworkManager wired profile |
| Role | UDP client sending to the Jetson | Listens on UDP 14550 |
| MAVROS | — | `fcu_url:=udp://:14550@` |

Test from the Jetson: `ping 192.168.144.14`.
