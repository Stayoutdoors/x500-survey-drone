# Field Checklist

**Author:** Garrett Hebert · FAA Part 107 remote pilot · [LinkedIn](https://www.linkedin.com/in/garrett-hebert-285506126/) · [GitHub](https://github.com/Stayoutdoors)


The whole field day in order: pack, check the site, power up, preflight, fly, power down. Adapted from the X500 operator's manual.

## 1. Packing list

**Aircraft**
- [ ] X500 with props installed and tight; spare set of 1045 props
- [ ] 4 × 5200 mAh 4S LiPo packs, charged to full the night before
- [ ] LiPo-safe bag or ammo can
- [ ] Remote ID broadcast module, powered and mounted

**Ground station**
- [ ] Ground station laptop (QGroundControl), charged, with charger
- [ ] SiK 915 MHz ground radio and USB cable
- [ ] USB-C cable for the Jetson's backup link
- [ ] External SSD for imagery (when the camera payload is flying)

**Radio**
- [ ] RadioMaster Boxer (ELRS), charged, with the X500 model selected

**Support**
- [ ] Charger and power source
- [ ] Landing pad
- [ ] Small toolkit: hex drivers, zip ties, spare JST-GH cables
- [ ] Fire extinguisher rated for lithium batteries
- [ ] Part 107 certificate, FAA registration, airspace authorization if needed

## 2. Site check (before you unpack)

1. **Airspace.** Check the site in B4UFLY or a LAANC provider. Get LAANC authorization in controlled airspace.
2. **Weather.** Wind under about 20 mph at altitude, no precipitation, visibility at least 3 statute miles (Part 107 minimum).
3. **Takeoff spot.** Flat, clear of people, at least 10 m from metal structures and power lines (they disturb the compass), clear view of the sky.
4. **Terrain.** Note any slope. Altitude in QGC is height above the takeoff point, not above the ground under the drone.

## 3. Power-up order

ELRS needs the transmitter on **before** the receiver powers up, and the Jetson must never lose power mid-write.

1. **Transmitter on.** Select the X500 model; check its battery.
2. **Ground station on.** Plug in the SiK radio, open QGroundControl.
3. **Flight battery in.** Strap it down, plug in the XT60. Keep clear of the props.
4. **Wait about 60 s.** The Pixhawk boots, QGC connects, the receiver LED goes solid blue, the Jetson boots.
5. **Check MAVROS** (starts automatically as a systemd service):

```bash
source /opt/ros/humble/setup.bash
ros2 topic echo /mavros/state --once   # expect connected: true
```

## 4. Preflight checks

| Check | Where | Pass |
|---|---|---|
| Vehicle connected | QGC top bar | Flight mode shown, not "Disconnected" |
| Pre-arm messages | QGC messages | None |
| Battery voltage | QGC or transmitter telemetry | 16.6–16.8 V for a fresh pack |
| GPS | QGC top bar | 3D fix, 10 or more satellites |
| RC link | Transmitter telemetry (RQly) | 100% on the pad |
| Flight mode switch | CH6 | Up = Stabilize, shown in QGC |
| Radio failsafe | Parameters | `FS_THR_ENABLE` set |
| Props | Visual | Tight, no chips or cracks, correct rotation |
| Frame | Visual | Arms locked, battery strap tight, GPS mast upright |
| Jetson (if used) | `/mavros/state` | `connected: true` |

> **Warning:** `ARMING_MAGTHRESH` must read **100** (default) before props go on. It is set to 0 only for indoor, props-off bench tests, where building wiring trips the "Check mag field" pre-arm check.

## 5. Flying a survey mission

**Plan it** (QGC Plan view): Pattern → Survey, draw the polygon, set altitude, overlap and camera, confirm the mission ends with Return to Launch or Land, then Upload.

**Fly it:**
1. Mode switch to **Stabilize**. Arm with the CH5 arm switch.
2. Take off by hand to about 5 m; confirm it holds steady.
3. Switch to **Loiter**; confirm it holds position.
4. In the Fly view, slide to **Start Mission** (Auto).
5. Watch the drone, not the screen. Keep visual line of sight the whole time.
6. It returns and lands at the end. Disarm once the motors stop.

**Taking control back:** flip the mode switch. Stabilize, AltHold or Loiter overrides Auto at once.

> **Warning:** Before switching **to Stabilize** from Guided, AltHold, Loiter or Auto, bring the throttle stick to mid. Stabilize hands throttle straight to the stick; with the stick at the bottom the motors go to zero. See the [incident log](troubleshooting.md#incident-2026-11-08-loss-of-thrust-after-mode-switch).

## 6. Emergency procedures

| Situation | What the drone does | What you do |
|---|---|---|
| RC link lost | Radio failsafe: RTL (Land without GPS) | Move toward the drone; it recovers when the link returns |
| Telemetry link lost | Keeps flying its mode | Fly visually |
| Low battery | RTL at 14.0 V or 1,300 mAh left; Land at 13.6 V or 780 mAh left | Land immediately |
| Drifting or erratic in Loiter or Auto | GPS or compass problem | Switch to **AltHold**, fly it home by hand |
| Flyaway | — | Switch to **Stabilize**, fly it down. Last resort: arm switch to disarmed (motors stop) |
| Prop strike or crash | — | **Disarm immediately** |
| Battery puffed or hot | — | LiPo bag, away from flammables. Do not charge |

## 7. Power-down order

1. Disarm; wait for the props to stop.
2. **Shut down the Jetson first** (`sudo shutdown -h now`); wait about 15 s for its LED to go off.
3. Unplug the flight battery.
4. Close QGC, unplug the SiK radio.
5. Transmitter off last.
6. Log flight time, mAh used and resting voltage for each pack.

## 8. After the field day

- [ ] Packs to storage charge (3.80 V per cell) if idle longer than two days
- [ ] Download flight logs (QGC → Analyze Tools → Log Download)
- [ ] Copy imagery off the payload
- [ ] Inspect props, arms and connectors
- [ ] Record any problem in the [troubleshooting log](troubleshooting.md)
