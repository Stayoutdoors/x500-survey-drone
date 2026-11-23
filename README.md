# X500 Survey Drone

A Holybro X500 quadcopter built from the frame up as a survey platform for ecological and geospatial mapping. It supports fieldwork for my M.S. in Geographic Information Science at San Francisco State University.

**Status:** first test flights complete (November 2026). Tuning, sensor integration, and the first mapping mission are next.

<p align="center">
  <img width="720" alt="X500 hovering during a test flight over a forested site" src="https://github.com/user-attachments/assets/ddc73811-88c3-4c3d-aed4-1ddff1c37166" />
</p>

## In the field

https://github.com/user-attachments/assets/baba1b58-a19a-430d-8238-f0993afd1e11

<table>
  <tr>
    <td width="50%"><img alt="X500 on its landing pad, ready for a test flight" src="https://github.com/user-attachments/assets/ca4f486e-4e1d-4dfc-a737-138d266945cc" /></td>
    <td width="50%"><img alt="Field ground station: QGroundControl on a laptop with the ExpressLRS transmitter" src="https://github.com/user-attachments/assets/21e42d9c-fe05-41ef-8850-5c8b4e2a8dac" /></td>
  </tr>
  <tr>
    <td align="center"><em>Ready for a test flight</em></td>
    <td align="center"><em>Field ground station: QGroundControl and ExpressLRS transmitter</em></td>
  </tr>
</table>

## Why I'm building it

Data you can trust starts with a platform you understand. I built this aircraft myself to understand the whole system:

- **Hardware:** frame, motors, ESCs, power distribution, and how every component works together
- **Software:** flight controller firmware, parameters, calibration, and the tuning that turns a pile of parts into a stable aircraft
- **Flying:** how inputs, conditions, and setup choices show up in the air

When something works, I want to know why. When something doesn't, I want to be able to diagnose it myself instead of guessing.

## System stack

| Layer | Component |
|---|---|
| Airframe | Holybro X500 |
| Flight controller | Pixhawk 6X |
| Flight firmware | ArduPilot (ArduCopter 4.7) |
| Companion computer | NVIDIA Jetson Orin Nano, JetPack 6.2 |
| Robotics middleware | ROS 2 Humble + MAVROS |
| FC ↔ companion link | MAVLink over Ethernet |
| Positioning | Holybro H-RTK F9P GNSS over DroneCAN |
| Ground control | QGroundControl |
| RC link | ExpressLRS |

## Architecture

```mermaid
flowchart LR
    GCS["QGroundControl<br/>(ground station)"] <-- telemetry --> FC
    RC["ExpressLRS<br/>RC transmitter"] --> FC
    GNSS["H-RTK F9P GNSS"] -- DroneCAN --> FC["Pixhawk 6X<br/>ArduCopter 4.7"]
    FC <-- "MAVLink / Ethernet" --> JET["Jetson Orin Nano<br/>ROS 2 Humble + MAVROS"]
    JET --> SENS["Survey sensors<br/>(planned)"]
```

The Jetson runs alongside the flight controller and talks to it over MAVLink on Ethernet. That link is the foundation for onboard processing and autonomy: the flight controller keeps the aircraft stable, and the Jetson handles mission logic and sensor data.

## Roadmap

- [x] Assemble airframe and power system
- [x] Configure ArduCopter on the Pixhawk 6X
- [x] Bring up Jetson Orin Nano with ROS 2 Humble and MAVROS
- [x] MAVLink link between flight controller and companion computer
- [x] First test flights
- [ ] Extended flights and PID tuning
- [ ] Survey sensor integration
- [ ] First mapping mission and processed orthomosaic
- [ ] Compare RTK-tagged imagery against ground control points

## Related

Part of my GIS and remote sensing work. See my [profile](https://github.com/Stayoutdoors) for other projects.

---

Garrett Hebert · FAA Part 107 remote pilot · [LinkedIn](https://www.linkedin.com/in/garrett-hebert-285506126/)
