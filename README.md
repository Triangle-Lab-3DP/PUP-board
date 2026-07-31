# PUP Project README

## Introduction

"PUP" is a control board specifically designed for MMU (Multi-Material Unit) systems like [EMU](https://github.com/DW-Tas/EMU) , where each channel requires an independent controller. Of course, you can use it for any other project you wish.

### Specifications

  * **Main Chip**: RP2040 133mhz.
  * **Connectivity**: Dual power connectors and dual CAN interfaces for convenient daisy-chaining.
  * **Drivers**: 1 x Onboard TMC2226 (TMC2209 in different packages) stepper driver.
  * **Dedicated Interfaces**: 1 x 8-pin XH2.54 connector is provided for direct connection to the EMU Hatch Board.
  * **Lighting**: 2 x Neo pixel RGB interfaces, one of which is integrated into the 8-pin XH2.54 connector.
  * **IO Expansion**:
      * 8 x I/O connectors, two of which are integrated into the 8-pin XH2.54 connector.
      * 3 x I2C interfaces.
      * 1 x SPI (AUX) interfaces.
  * **Outputs & Sensors**:
      * 2 x 24V Fans Output (1A max).
      * 1 x Heater output, 5A max.
      * 1 x NTC temperature sensor interface for temperature monitoring.
      * 1 x ADC pin for PSF Sensor.
  * **Power Output**: The 5V/3A output can drive larger servos.
<img width="800" height="875" alt="image" src="https://github.com/user-attachments/assets/8fbddd2b-a178-4310-8daf-c14f5e96fb1f" />
<img width="800" height="867" alt="image" src="https://github.com/user-attachments/assets/fb2e704b-d1c0-4942-a7a5-d1557ca04803" />
<img width="979" height="826" alt="image" src="https://github.com/user-attachments/assets/e64ccac7-cddc-4415-8729-ab9a2047b67f" />

### Simplified Wiring & Precautions

The PUP board features integrated CAN bus and power I/O ports designed for daisy-chain expansion, significantly streamlining your wiring layout.

> **⚠️ Terminating Resistor Note:**  
> The CAN bus requires a 120Ω terminating resistor connected in parallel at both ends of the link. You need to install a jumper with a 120-ohm resistor on the last PUP board; none of the intermediate ones require it.
<img width="667" height="214" alt="image" src="https://github.com/user-attachments/assets/72023e0c-d432-49ca-94f7-c8e83ac99639" />

> **⚠️ Power Limit Warning:**  
> When daisy-chaining power across multiple PUP boards, the total combined output current must never exceed **5A**. Avoid using high-power heating outputs in this configuration. If you need to drive heaters on multiple boards simultaneously, you must wire their power inputs in parallel directly to the main power supply.

## Pin Map
<img width="1123" height="967" alt="image" src="https://github.com/user-attachments/assets/6ddf01f6-78d3-48e9-8cfa-c6d3cccd91b8" />


## Getting Started

PUP boards ship pre-flashed with the Katapult bootloader and Klipper firmware. Once the boards are correctly wired and powered on, they are ready for use, and the RUN LED will remain solidly lit.

### 1\. Host Configuration

Please ensure that the CAN bus interface on your Klipper host is configured and the speed is set to 1,000,000.

### 2\. Querying UUIDs

Executing the `canbus_query` command will display the UUIDs of all connected PUP boards:

``` bash
~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0

```
<img width="1111" height="204" alt="image" src="https://github.com/user-attachments/assets/d3de1fcb-5fbd-421a-a33c-8559c1ec0a44" />


### 3\. Lane Mapping

To map a specific UUID to its physical lane (e.g., Lane 0), follow these steps:

1.  Double-click the **RST** button on the target board. The **RUN LED** will begin blinking.
 <img width="953" height="598" alt="image" src="https://github.com/user-attachments/assets/f407553e-67eb-40cf-89c9-5d6e1db9e984" />

2.  Re-run the `canbus_query` command. The application mode for this specific board's UUID will now display as **CanBoot** (or **Katapult**).
 <img width="1033" height="221" alt="image" src="https://github.com/user-attachments/assets/be3c065a-3fba-49ae-a755-6af2f0dadfaa" />

3.  Record this UUID, then repeat the process for the remaining boards to map every lane.



## License

This project is licensed under the MIT License - see the LICENSE file for details.



