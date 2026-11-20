# Senua AI: sensor and motor faults on a simulated drone, caught on the device

Senua AI's engine learned what normal flight looks like from a quadcopter's own raw sensors, then watched 52 flights it
had never seen. In 28 of them the simulator injected a fault mid-mission: seven kinds, four flights each, from a GPS
altitude jump to a motor losing a quarter of its thrust.

**The engine caught all 28, each within 0.2 to 2.1 seconds of the fault, and flagged under 1% of normal flight.** The
pass marks were written down before any of these flights flew.

## At a glance

| | |
|---|---|
| Simulator | Stock ArduPilot SITL, ArduCopter quad, default parameters |
| Faults | 7 kinds injected by the simulator's own parameters, 4 flights each |
| Caught | 28 of 28, every kind 4 of 4, within 0.2 to 2.1 s |
| Normal flight flagged | 0.89% of windows (pass mark 5%) |
| Chance check | at most 1 of 4 flights per kind flagged on the fault's sensor in the same span before the fault |
| What the engine reads | 20 raw sensor fields: GPS, accelerometer, gyro, compass, barometer |
| What it learned from | 168 earlier normal flights; it never saw a fault before the test |

## Results

| Fault | Caught | Time to catch (s) | Sensor that flagged it |
|---|---|---|---|
| GPS altitude jump, 10 m | 4 of 4 | 0.23 to 0.53 | GPS altitude |
| Barometer glitch, 10 m | 4 of 4 | 0.60 to 0.80 | barometric pressure |
| Barometer drift, 0.5 m/s | 4 of 4 | 0.60 to 0.70 | barometric pressure |
| Gyro bias, 0.1 rad/s | 4 of 4 | 1.70 to 2.10 | gyro x |
| Accelerometer bias, 1 m/s² | 4 of 4 | 0.20 to 0.70 | accelerometer z |
| Compass offset, 200 mGauss | 4 of 4 | 0.30 to 0.50 | magnetometer x |
| Motor 1 at 75% thrust | 4 of 4 | 0.50 | gyro x |

Every test flight's figures are in [`results/test_flights.csv`](results/test_flights.csv), and how they were produced is
in [`docs/method.md`](docs/method.md).

## What this does not show yet

These are the next steps, stated plainly:

- It is simulation: one airframe, one size of each fault. Real flights, other airframes and other fault sizes are next.
- Every normal test flight still raised at least one short-lived flag on some sensor, mostly at take-off and on the
  return. The figures above are per window and per sensor. A single alarm for a whole flight needs the next layer,
  reading how the sensors relate to each other, which is in progress.
- The 168 normal flights the engine learned from are not in this repository yet. The 52 test flights are.

## How Senua AI helps

Senua AI is an engine that learns cause and effect straight from a machine's own data. We do not use models. No LLMs. No
SLMs. Of any kind. There is no GPU, no data centre and no AI token cost, ever, and it runs on low end CPUs: the whole
engine is about 8 MB and runs offline on a Raspberry Pi. For a drone, robot or satellite that means fault detection on
board, learned from its own normal operation, that keeps working when the link to base is jammed or gone. The same
engine can read any vehicle's telemetry, and this simulator study is the first of several.

## What is here

| Path | What it holds |
|---|---|
| [`docs/method.md`](docs/method.md) | The flights, the faults, how the engine reads a flight, the pass marks and the checks |
| [`docs/results.md`](docs/results.md) | The results flight by flight, and what each figure means |
| [`docs/data-format.md`](docs/data-format.md) | How to read the recordings |
| [`data/flights/`](data/flights) | The 52 test flights' recordings, one file per flight |
| [`data/roster.csv`](data/roster.csv) | Each flight's role, fault, size and wind |
| [`data/streams.csv`](data/streams.csv) | Every recorded sensor field, its unit, and whether the engine read it |
| [`results/test_flights.csv`](results/test_flights.csv) | Each test flight's windows, flags and first catch |

## Licence

The data, results and documentation are licensed under [CC BY 4.0](LICENSE).

## Support

Senua AI Pty Ltd, Australia. Questions: support@senuaai.com.
