# The recordings

Each file in [`../data/flights/`](../data/flights) is one flight, gzipped, tab-separated, one line per event. A
recording never holds a sensor's reading itself, only its changes. That is what the engine reads, and it keeps the data
free of absolute positions.

## Line types

| First field | Line | Meaning |
|---|---|---|
| `A` | `A <message> <t>` | A message stream's first reading at time t (its appearance; no value) |
| `M` | `M <message> <t> <change 1> <change 2> ...` | A moment: each field's change since the same message's previous reading |
| `S` | `S <message> <t> <speed 1> <speed 2> ...` | The speed of those changes: how much each field's change itself changed, from the stream's second moment on |
| `D` | `D <message> <t>` | The stream's last reading (its disappearance; no value) |
| `I` | `I <t> <parameter> <value x 1000>` | The fault switched on: the simulator parameter and its value times 1000 |
| `P` | `P <t> <parameter> <value x 1000>` | A parameter set at the start of the flight (the motor flights only) |
| `#` | `# <stage>=<value>` | A flight stage reached: GPS fix, recording started, guided mode, armed, take-off, laps flown (the number of lap corners), return |

- `t` is simulator time in microseconds.
- `<message>` is the MAVLink message id. The fields follow in the order listed in
  [`../data/streams.csv`](../data/streams.csv), which also gives each field's unit.
- Integer fields are in MAVLink's own units. Floating-point fields (ATTITUDE, VFR_HUD's speeds and altitude, the
  pressures) are stored exactly as fixed point, the value times 65536.
- A message whose own clock has not moved on is the same reading and is not recorded again (the GPS sends each fix
  twice). VFR_HUD carries no clock and takes the time of the latest GLOBAL_POSITION_INT.

## The messages recorded

| Id | Message | Fields | Read by the engine |
|---|---|---|---|
| 24 | GPS_RAW_INT | 8 | yes |
| 27 | RAW_IMU | 9 | yes |
| 29 | SCALED_PRESSURE | 3 | yes |
| 30 | ATTITUDE | 6 | no |
| 33 | GLOBAL_POSITION_INT | 8 | no |
| 36 | SERVO_OUTPUT_RAW | 4 | no |
| 74 | VFR_HUD | 6 | no |

## Example

From flight F1a:

```
A	24	110363000
M	24	114163000	0	0	30	0	0	0	3	0
I	243336000	SIM_GPS1_ALT_OFS	10000
```

The first line says the GPS stream started at 110.363 s of simulator time. The second says that at 114.163 s GPS
latitude and longitude did not change, altitude rose by 30 mm, and the course over ground moved by 0.03 degree. The
third says the GPS altitude offset of 10 m (10000 / 1000) was switched on at 243.336 s.
