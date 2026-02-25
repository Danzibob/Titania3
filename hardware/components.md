## Components

### MCU - [STM32L072CBT6](https://www.st.com/resource/en/datasheet/stm32l072v8.pdf)
- Ultra low power
- ARM Cortex M0+ 32kHz - 32MHz
- 128kB internal flash
- 20kB SRAM
- USB Support
  - Pre-programmed bootloader with USB FS
  - No external crystal required

### Barometer - [BMP580](https://www.lcsc.com/datasheet/C22391138.pdf)  <!-- Can't find this on Bosch's website for some reason -->
- Ultra low power
- 30..125kPa
- -40..85C
- up to 622Hz data rate (FORCED or CONTINOUS mode)
  - up to 240Hz in NORMAL mode
- Configurable oversampling & filtering

### IMU - [BMI260](https://www.smart-chips.cn/wp-content/uploads/2025/10/BST-BMI260-DS000-01_typ_final.pdf) <!-- Can't find this on Bosch's website for some reason -->
- Ultra low power (typ 0.7mA)
- 6 axis (acceleration & gyro)
- +/- 16g @ up to 1.6kHz
- +/- 2000dps @ up to 6.4kHz
- Configurable oversampling & filtering

### External Flash - [MX25R3235FM1IL0](https://www.macronix.com/Lists/Datasheet/Attachments/8755/MX25R3235F,%20Wide%20Range,%2032Mb,%20v1.8.pdf)
- Ultra low power (max 6mA bursts on write, typ 3.5mA)
- 4MB storage (32Mbit)
- Accessed via SPI

### Battery Charging - [MCP73831T-2ATI/OT](https://ww1.microchip.com/downloads/en/DeviceDoc/MCP73831-Family-Data-Sheet-DS20001984H.pdf)
- 4.2V CCCV
- 15-500mA configurable (we use the minimum of 15mA)
- Automatic power-down

### Voltage Regulator (LDO) - [XC6206P332MR-G](https://jlcpcb.com/api/file/downloadByFileSystemAccessId/8579707861620228096)
- JLC Basic component
- 3.3V 200mA output
- Will function down to 3.6V battery voltage
  - This gives us access to ~70-90% of the battery capacity (varies based on temperature)

### Battery - [LIR1254/LIR1255](https://eemb.oss-accelerate.aliyuncs.com//uploads/20230322/bf76cf810935c483152a5993efdd3994.pdf)
- Any 12mm rechargable coin cell is compatible
- Retained by an M2 bolt and minimal 3D printed chassis
- 45-75mAh capacity depending on brand & age
- Max output current approx. 1C
- Reccomended output current ~0.3C

### Charging LED (Red) - [KT-0603R](https://jlcpcb.com/api/file/downloadByFileSystemAccessId/8550723991833485312)
- JLC Basic component
- 2.4V 20mA
- Amusingly uses more power than the rest of the datalogger combined lol
  - Is only used as a USB charging indicator