# Results

All figures are in [`../results/test_flights.csv`](../results/test_flights.csv).

## Detection: 28 of 28

| Fault | Flights caught | Seconds to the first catch, per flight | Sensor that flagged first |
|---|---|---|---|
| GPS altitude jump, 10 m | 4 of 4 | 0.227, 0.527, 0.227, 0.427 | GPS altitude |
| Barometer glitch, 10 m | 4 of 4 | 0.700, 0.601, 0.601, 0.801 | barometric pressure |
| Barometer drift, 0.5 m/s | 4 of 4 | 0.600, 0.701, 0.701, 0.601 | barometric pressure |
| Gyro bias, 0.1 rad/s | 4 of 4 | 2.100, 1.900, 1.700, 2.100 | gyro x |
| Accelerometer bias, 1 m/s² | 4 of 4 | 0.701, 0.500, 0.500, 0.200 | accelerometer z |
| Compass offset, 200 mGauss | 4 of 4 | 0.400, 0.300, 0.500, 0.501 | magnetometer x |
| Motor 1 at 75% thrust | 4 of 4 | 0.500, 0.500, 0.501, 0.501 | gyro x |

Flights in each row are in roster order (winds 45, 135, 225 and 315 degrees). A weakening motor shows first in the
gyro, the sensor that measures the aircraft's rotation.

## Normal flight: 0.89%

On the 8 normal test flights, 2,370 of 266,960 sensor windows were flagged, 0.89%, against a pass mark of 5%.

| Flight | Windows | Flagged | Share |
|---|---|---|---|
| N0 | 33,400 | 272 | 0.81% |
| N1 | 33,502 | 276 | 0.82% |
| N2 | 33,468 | 264 | 0.79% |
| N3 | 33,491 | 255 | 0.76% |
| N4 | 33,399 | 168 | 0.50% |
| N5 | 33,429 | 431 | 1.29% |
| N6 | 33,497 | 398 | 1.19% |
| N7 | 32,774 | 306 | 0.93% |

## Chance check

For each fault kind, at most 1 of its 4 flights had the fault's own sensors flagged in the same span before the fault
began: 0 of 4 for the GPS altitude jump, both barometer faults and the motor; 1 of 4 for the gyro, accelerometer and
compass faults. All three of those came from flights with the same wind direction.

## Reading the CSV

- `windows_scored`: the sensor windows scored in the flight, across all 20 sensors read.
- `windows_flagged`: of those, the windows flagged. A fault flight's count includes the windows after the fault,
  which is why the motor flights are high.
- `caught`: `yes` when a sensor the fault moves was flagged within the time limit after the fault.
- `seconds_to_catch`: from the fault's switch-on to the first flagged window on such a sensor, in simulator time.
- `flagged_sensor`: that sensor, by its name in [`../data/streams.csv`](../data/streams.csv).
