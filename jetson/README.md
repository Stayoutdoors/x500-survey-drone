# Jetson Setup

**Author:** Garrett Hebert · FAA Part 107 remote pilot · [LinkedIn](https://www.linkedin.com/in/garrett-hebert-285506126/) · [GitHub](https://github.com/Stayoutdoors)

Configuration for the NVIDIA Jetson Orin Nano companion computer: the Ethernet link to the Pixhawk 6X and the MAVROS service. Exported from the aircraft in November 2026.

## Versions

| Component | Version |
|---|---|
| Board | Jetson Orin Nano 8 GB Developer Kit |
| L4T | R36.5.2 |
| OS | Ubuntu 22.04.5 LTS |
| ROS 2 | Humble (ros-humble-ros-base / desktop 0.10.0) |
| MAVROS | 2.16.0 (ros-humble-mavros, ros-humble-mavros-extras) |

## 1. Ethernet link to the Pixhawk

An isolated two-device link on the aircraft: no router, no outside network.

| Item | Pixhawk 6X | Jetson |
|---|---|---|
| Address | 192.168.144.14 | 192.168.144.10/24 |
| Configured by | ArduPilot `NET_*` parameters | NetworkManager profile `pixhawk` |
| Role | UDP client, sends MAVLink2 to port 14550 | MAVROS listens on UDP 14550 |

**Jetson side** (exported profile):

```
connection.id:              pixhawk
connection.interface-name:  enP8p1s0
connection.autoconnect:     yes
ipv4.method:                manual
ipv4.addresses:             192.168.144.10/24
ipv4.gateway:               --
ipv6.method:                ignore
```

To recreate it:

```bash
sudo nmcli connection add type ethernet ifname enP8p1s0 con-name pixhawk \
  ipv4.method manual ipv4.addresses 192.168.144.10/24 \
  ipv6.method ignore connection.autoconnect yes
```

No gateway is set, so the Pixhawk link never becomes the Jetson's default route and Wi-Fi keeps internet access.

**Pixhawk side:** `NET_ENABLE 1`, `NET_DHCP 0`, `NET_IPADDR 192.168.144.14`, `NET_P1_TYPE 1` (UDP client), `NET_P1_IP 192.168.144.10`, `NET_P1_PORT 14550`, `NET_P1_PROTOCOL 2` (MAVLink2). Full export in [`config/`](../config/).

Check the link from the Jetson:

```bash
ping -c 3 192.168.144.14
```

## 2. MAVROS as a systemd service

[`mavros.service`](mavros.service) starts MAVROS at boot with the ArduPilot launch file and restarts it if it crashes, so no SSH session is needed. `fcu_url:=udp://:14550@` listens on port 14550 and replies to whichever address the Pixhawk sends from.

Install:

```bash
sudo cp mavros.service /etc/systemd/system/
sudo sed -i "s/YOUR_USER/$USER/" /etc/systemd/system/mavros.service
sudo systemctl daemon-reload
sudo systemctl enable --now mavros
```

Status: `enabled`, `active` on the aircraft.

Check and manage:

```bash
systemctl status mavros --no-pager
journalctl -u mavros -f
source /opt/ros/humble/setup.bash
ros2 topic echo /mavros/state --once   # expect connected: true
```

## 3. Data streams

ArduPilot sends only heartbeat and status text until MAVROS requests streams, so sensor, battery and GPS topics stay empty. Request them after startup:

```bash
ros2 service call /mavros/set_stream_rate mavros_msgs/srv/StreamRate \
  "{stream_id: 0, message_rate: 10, on_off: true}"
```

**Open item:** request streams automatically when the service starts (or set ArduPilot's stream-rate parameters for the Ethernet port).

## Notes

- If `ros2 topic list` shows no `/mavros` topics while the service is running, run `ros2 daemon stop` and retry.
- Don't start a second MAVROS by hand: two instances can't share UDP 14550.
- Shut the Jetson down (`sudo shutdown -h now`) before unplugging the flight battery.
