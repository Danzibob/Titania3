# Titania III
This project is the third iteration of the tiny titania data-logger for model rocketry.

The project goal is to create a datalogger small and light enough for an 18mm A8-3 rocket, but capable enough to use as a backup logger for mid and high power flights.
An additional aim of this project is to run on a rechargable coin cell battery instead of the usual LiPo.
This restriction heavily limits our component choices because of the low current provided by these coin cells (usually <15mA sustained).

## Docs
- [Components List](./hardware/components.md)

## Features
- Logs barometric pressure, temperature, acceleration and rotation data
- Up to 600Hz logging in bursts, 200Hz normal target
- Logs save to flash memory (4MB)
- Buzzer / beeper for basic output & assist in recovery
- PCB-edge USB-A for firmware programming, data offload & charging
- Fits in an 18mm rocket body tube
- Mass of < 10g (including battery)
- LIR1254 rechargable coin cell power source

## Logging format
In order to maximise the logging time available in the flash, we make a trade-off for minimizing storage over precision. This project aims to allow for the following flight timings to fit in the 4MB log flash:

| Flight Phase | Duration (up to) | Storage Priority | Storage Allocation | Max Data Rate |
|---|---|---|---:|---:|
| Pad time | 1 hour | Small constant storage | <1% (~100B) | N/A |
| Ascent | 2 minutes | Most important | ~75% (3 MB) | 25 kB/s |
| Descent | 20 minutes | Less important | ~25% (1 MB) | ~800 B/s |
| Recovery | 2 hours | No storage required | N/A | N/A |


### BMP580
#### Barometric pressure
- Sensor measurement range: 30-125kPa
- Absolute accuracy: +/- 50Pa 
  - Corresponds to ~4m of altitude near sea level
- Relative accuracy: 6Pa per 10kPa
  - ~0.5m / 1km of altitude
  - _Relative accuracy is what's important for determining the altitude of a flight_
- Resolution at oversampling levels (RMS noise @ 100kPa):
  - Low      (1x)   = 0.78Pa @ up to 498Hz
  - Standard (x4)   = 0.41Pa @ up to 255Hz
  - High     (x16)  = 0.21Pa @ up to 87Hz
  - Highest  (x128) = 0.08Pa @ up to 12Hz

On pad, take an f32 Highest resolution sample over a few seconds.
- Massive overkill but nice to have accurate ground pressure

In flight, record u16 pressure with LSB = 2Pa.
- This gives us spatial resolution of ~16cm
- And a range of 0-131kPa, which fully covers the sensor range
- Also easy to convert by bitshifting


#### Temperature
- While ascending, the rocket is moving too fast for the air temperature inside the chassis to equalize
- Therefore, only take temperature readings on the pad and during descent
- Based on the Troposphere lapse rate of ~ 6.5K/km, and the 10km ceiling for this logger, we need to deal with at least a range of 60 degrees
- Combined with a ground temperature range of -10 to 50C this requires a total range of at least 120 degrees
- f16 gives us plenty of resolution within this range

On pad/boot, record a temperature value over several seconds (f16)
- We can combine this with ground pressure for better altitude estimates

Not much point taking temperature readings on ascent
- Could maybe store an i8?

On descent, the barometer will be exposed to the open air at slower speed. 
- Take f16 measurements at regular intervals
- These can be used to accurately profile the air column

#### Sampling & Data Rates
- **Pad** - take a set of samples over a preset startup window (10s?)
  - 6 bytes (f32 pressure and f16 temp)
  - Potentially worth repeating every 30 seconds until launch?
  - Ring buffer to save last N measurements before launch?
- **Ascent** - Log Pressure (and _maybe_ temp) at high frequency
  - 500Hz * u16(+i8) = 1kB/s (1.5kB/s) = 4% (6%) of quota
  - 250Hz * u16(+i8) = 500B/s (750B/s) = 2% (3%) of quota
  - 100Hz * u16(+i8) = 200B/s (300B/s) = <1% (~1%) of quota
- **Descent** - Significantly lower frequency; log pressure & temp at full resolution
  - 10Hz * (u16+f16) = 40B/s = 5% of quota
- **Ground**
  - No logging required