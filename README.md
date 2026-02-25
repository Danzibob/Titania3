# Titania III
This project is the third iteration of the tiny titania data-logger for model rocketry.

The project goal is to create a datalogger small and light enough for an 18mm A8-3 rocket, but capable enough to use as a backup logger for mid and high power flights.
An additional aim of this project is to run on a rechargable coin cell battery instead of the usual LiPo.
This restriction heavily limits our component choices because of the low current provided by these coin cells (usually <15mA sustained).

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
In order to maximise the logging time available in the flash, we make a trade-off for minimizing storage over precision.

### BMP580
#### Barometric pressure
- Sensor measurement range: 30-125kPa
- Absolute accuracy: +/- 50Pa 
  - Corresponds to ~4m of altitude near sea level
- Relative accuracy: 6Pa per 10kPa
  - ~0.5m / 1km of altitude
  - _Relative accuracy is what's important for determining the altitude of a flight_
- Resolution at oversampling levels (RMS noise @ 100kPa):
  - Low      (1x)   = 0.78Pa @ 498Hz
  - Standard (x4)   = 0.41Pa @ 255Hz
  - High     (x16)  = 0.21Pa @ 87Hz
  - Highest  (x128) = 0.08Pa @ 12Hz

On pad, take an f32 Highest resolution sample over a few seconds.
- Massive overkill but nice to have accurate ground pressure
In flight, record u16 pressure with LSB = 2Pa.
- This gives us spatial resolution of ~16cm
- And a range of 0-131kPa, which fully covers the sensor range
- Also easy to convert by bitshifting

#### Temperature
- While the sensor does measure this to silly precision, it's not gonna be as good of a reflection of the actual ambient air temperature
- As such we want to use an altitude algorithm that doesn't rely on precise temperature

<!-- At this point I got distracted and started emailing people asking how they calculate altitude -->

**Potential Altitude algorithms**

| Algorithm         | Needs High-Res T?  | Good for 10 km? | Rocket-Appropriate? |
| ----------------- | ------------------ | --------------- | ------------------- |
| Isothermal        | No                 | Moderate        | Yes                 |
| ISA Lapse         | No (launch T only) | Yes             | ⭐ Best balance      |
| Full Hypsometric  | Yes                | Yes             | Usually overkill    |
| Relative Pressure | No                 | Yes             | ⭐ Very robust       |
