# First Test Flight Protocol

Used for the first flights after the build, and again after any repair or major change. Fly in order and finish each phase before starting the next.

## Results

| Date | Phases | Outcome |
|---|---|---|
| 2026-11-07 | Phase 2 (first hover, Stabilize) | Completed over seven flights |
| 2026-11-08 | Phases 2–5 | **All passed:** hover, AltHold, Loiter, and the in-air radio failsafe (RTL, landed and disarmed). Extended flight to 19.9 m |
| 2026-11-08 | Outside the protocol | Loss-of-thrust incident after a mode switch; see the [incident report](troubleshooting.md#incident-2026-11-08-loss-of-thrust-after-mode-switch) |

The full protocol is repeated after the post-incident repairs and before any mapping missions.

## Abort and emergency

- Land and disarm at once for oscillation or wobble, motors or ESCs too hot to touch, voltage under about 14.0 V, circling in Loiter, or any message you can't explain.
- Erratic in Loiter: switch to AltHold or Stabilize and fly it down by hand.
- Flyaway, last resort: CH5 arm switch to disarmed. The motors stop and the drone falls.
- Prop strike or crash: disarm immediately.

## Phase 0: Site

- Flat, open ground, 15 m clear of people, buildings, trees and vehicles. Pilot 10 m back, upwind.
- Wind under about 10 mph.
- Pack at 16.6–16.8 V (won't arm under 15.8 V). Transmitter charged.
- Ground station with QGC open and the SiK radio connected. Props, prop tool, LiPo bag or extinguisher.
- Props off.

## Phase 1: Ground checks (props off)

1. Power on: transmitter first, then the flight battery. Wait for QGC to connect.
2. QGC messages: "Check mag field" is gone. If it still reads about 120, recalibrate the compass on site.
3. `ARMING_MAGTHRESH` reads 100, not 0.
4. GPS: 3D fix, 10 or more satellites, HDOP under 1.0.
5. Heading: nose north reads about 0° and follows a 90° turn.
6. Mode switch (CH6): Up = Stabilize, Middle = AltHold, Down = Loiter (skip Phase 4 if Down shows Land).
7. Tilt the drone by hand: the attitude indicator moves the same way.
8. **Failsafe test A:** Stabilize, armed, throttle at the bottom, transmitter off. The drone disarms or shows Radio failsafe. Transmitter back on.
9. **Failsafe test B:** Stabilize, armed, throttle about 25%, transmitter off. QGC shows Radio failsafe and the mode changes to RTL (Land without GPS). Transmitter on, flip the mode switch to take control, throttle down, disarm. If nothing happens, the receiver is set to hold: do not fly until fixed.

## Phase 2: Props on, first hover (Stabilize)

1. Disarm and unplug the battery. Fit the props: M1 front-right CCW, M2 rear-left CCW, M3 front-left CW, M4 rear-right CW. The thick, rounded leading edge points into the direction of spin.
2. Battery in, wait for GPS, back off 10 m.
3. Mode switch up (Stabilize). Arm.
4. Raise the throttle smoothly to about 2 m and hover 30–60 s with small corrections. If it flips or lifts unevenly at takeoff, cut the throttle and disarm: a prop or motor direction is wrong.
5. Land gently; hold the throttle at the bottom until it disarms, or use CH5.
6. Check: motors warm, not hot; props tight; note the voltage.

## Phase 3: AltHold

1. Take off in Stabilize to about 3 m. Center the throttle and switch to AltHold.
2. It holds height with the throttle centered. Try small roll, pitch and yaw inputs.
3. If it bounces up and down, note it, switch back to Stabilize (**throttle at mid first**) and land.
4. To land in AltHold, hold the throttle below center; hold it low to disarm.

## Phase 4: Loiter (10 or more satellites)

1. Take off to about 5 m. Center the throttle and switch to Loiter.
2. Hands off: it holds position within about 1–2 m.
3. If it circles in a growing spiral, switch to AltHold at once and land: that points to a compass problem.
4. Small stick inputs: it moves and stops cleanly.

## Phase 5: In-air failsafe

1. Hover in Loiter at about 5 m, 10–15 m from the pilot.
2. Turn the transmitter off. The drone switches to RTL, climbs, flies home and lands.
3. Turn the transmitter back on and flip the mode switch to take control. Land.

## After each flight

- Record mode, duration, start and end voltage, and notes in the flight log.
- Two or three short flights per pack; keep it above 14.0 V under load (3.5 V per cell).
- Download the dataflash log: QGC → Analyze Tools → Log Download.
