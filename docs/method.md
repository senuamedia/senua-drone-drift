# Method

## The question

Can an engine that has only ever seen normal flight catch a sensor or motor fault on a drone, quickly, from the raw
sensors alone, without raising alarms in normal flight?

## The flights

- **Simulator:** stock ArduPilot SITL, the ArduCopter quad frame with its default copter parameters, built from the
  ArduPilot source at commit `de5add0`. Run at five times real time, eight simulators at once on one cloud server.
- **Mission:** arm, take off to 20 m, fly square laps of 40 m sides for 240 s of simulator time, then return and land.
  Each flight has a 3 m/s wind from the direction given in [`../data/roster.csv`](../data/roster.csv).
- **Recording:** the flight is flown and recorded through Senua AI's own MAVLink codec, at 10 Hz per message. Recording
  starts once the aircraft is ready to fly (GPS fix, position, a 20 s settle), because before that the GPS and the
  estimator send placeholder zeros.

| Role | Flights | Wind directions | Used for |
|---|---|---|---|
| Normal reference | 16 (R0 to R15) | 16 directions around the compass | setting each sensor's level |
| Normal test | 8 (N0 to N7) | 8 further directions | counting flags in normal flight |
| Fault test | 28 (F1a to F7d) | 45, 135, 225, 315 degrees | detection |

## The faults

Each fault is switched on 120 s into the mission, mid-lap, by one of the simulator's own parameters. The time of the
switch is recorded in each fault flight's file.

| Kind | Parameter | Size | Sensors the fault moves (fixed before the flights) |
|---|---|---|---|
| GPS altitude jump | `SIM_GPS1_ALT_OFS` | 10 m | GPS altitude |
| Barometer glitch | `SIM_BARO_GLITCH` | 10 m | barometric pressure |
| Barometer drift | `SIM_BARO_DRIFT` | 0.5 m/s | GPS altitude, barometric pressure |
| Gyro bias | `SIM_GYR1_BIAS_X` | 0.1 rad/s | gyro x |
| Accelerometer bias | `SIM_ACC1_BIAS_Z` | 1 m/s² | accelerometer z |
| Compass offset | `SIM_MAG1_OFS_X` | 200 mGauss | magnetometer x |
| Motor 1 at 75% thrust | `SIM_ENGINE_MUL` 0.75 set at the start, `SIM_ENGINE_FAIL` 1 at 120 s | 25% of its thrust lost | gyro x, y and z |

## What the engine reads

Only the raw sensors: 20 fields from three MAVLink messages, GPS_RAW_INT, RAW_IMU and SCALED_PRESSURE
([`../data/streams.csv`](../data/streams.csv)). The flight controller's own estimates (attitude, position, speed) and
the motor commands are recorded but not read.

## How the engine reads a flight

1. **Change, never level.** The engine never looks at a sensor's reading itself, only at how it changes from one
   moment to the next. Each change is taken exactly, digit by digit.
2. **Normal, learned from the data.** From 168 earlier normal flights of the same simulator, the engine learns, for
   each sensor, which changes follow which. It is never shown a fault.
3. **A score for every window.** Every window of 32 consecutive moments on a sensor gets a surprise score: how far
   its changes are from what normal flight leads the engine to expect.
4. **A level from normal flight.** Each sensor's score is compared with the same sensor's windows on the 16 normal
   reference flights, at a 1% false-alarm rate. Nothing is set by hand.
5. **A belief, revised as evidence arrives.** The decision for each sensor is held as a belief in the engine's memory
   and updated window by window.

Nothing was tuned to these flights. The same reading, unchanged, was applied to all 52.

## The pass marks, written down before any test flight flew

- **Detection:** for every fault kind, at least 3 of its 4 flights flagged on one of the sensors that fault moves,
  within 10 s of the fault (30 s for the barometer drift).
- **Normal flight:** at most 5% of the normal test flights' windows flagged.
- **Chance check:** for every fault kind, at most 1 of its 4 flights flagged on the fault's sensors in the same span
  before the fault began. This shows a catch is the fault, not the normal flag rate landing on the right sensor by luck.

All three held. The results are in [`results.md`](results.md).

## Checks made before publishing

1. **Repeatable.** A second run of the same program on the same data gives every figure byte for byte.
2. **Clean split.** No test flight is among the 168 flights the engine learned from, and every fault flight carries
   its switch-on time.
3. **No levels.** No recording holds a sensor's level, only its changes.
4. **One catch traced to the raw recording.** In flight F1a the GPS altitude jumps 5 m at 227 ms and again at 427 ms
   after the switch-on; the engine's first flag on GPS altitude is at 227 ms, the moment the jump arrives.
