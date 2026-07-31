# PUP Project README

## Introduction

"PUP" is a control board specifically designed for MMU (Multi-Material Unit) systems like EMU, where each channel requires an independent controller. Of course, you can use it for any other project you wish.

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

### 3\. Lane Mapping

To map a specific UUID to its physical lane (e.g., Lane 0), follow these steps:

1.  Double-click the **RST** button on the target board. The **RUN LED** will begin blinking.
2.  Re-run the `canbus_query` command. The application mode for this specific board's UUID will now display as **CanBoot** (or **Katapult**).
3.  Record this UUID, then repeat the process for the remaining boards to map every lane.

-----

## Firmware Update Guide

PUP boards ship pre-flashed with the latest Katapult bootloader and Klipper firmware. Under normal circumstances, no firmware updates are required before installation and use. If you need to update firmware, please choose from the methods below.

### Method A: Updating via CAN Bus using Katapult bootloader (Recommended)

This is the recommended method for updating Klipper firmware over the CAN interface.

#### 1\. Install Katapult

Ensure Katapult is installed on your Klipper host. (Reference: \[link removed\])

#### 2\. Compile Klipper Firmware

Access your Klipper host via SSH and run the following commands:

``` bash
cd ~/klipper
make clean
make menuconfig

```

*Note: You must select **Bootloader offset (16KiB bootloader)**.*  
Once configured, press **ESC** to exit and **Y** to save. Then, compile the firmware:

``` bash
make

```

This will generate the `klipper.bin` file.

#### 3\. Flash the Firmware

Ensure the target PUP board is powered on and visible on the CAN bus.

1.  Double-click the **RST** button on the PUP board. The **RUN LED** will begin blinking, indicating the board has entered Katapult bootloader mode.
2.  Query the bus to find the bootloader UUID:

<!-- end list -->

``` bash
python3 ~/katapult/scripts/flashtool.py -i can0 -q

```

3.  Locate and record the UUID where the application state is listed as `Application: Katapult`.
4.  Flash the new firmware using your recorded UUID (replace `<your_uuid>` with your actual UUID):

<!-- end list -->

``` bash
python3 ~/katapult/scripts/flashtool.py -i can0 -u <your_uuid> -f ~/klipper/out/klipper.bin

```

*Note: If you select the "Bootloader offset (16KiB bootloader)" option when compiling the Klipper firmware, the compilation process will generate a `.bin` file instead of a `.uf2` file, meaning you cannot flash the firmware via drag-and-drop. You must use Katapult's `flashtool.py` to flash the firmware.*

-----

### Method B: Direct Flashing via USB

This method completely overwrites the flash memory and does not use the Katapult bootloader. **This is generally not recommended**, as all future updates will require a physical USB connection unless you specifically intend to use the PUP board over a USB communication interface instead of CAN bus.

#### 1\. Compile Klipper Firmware

``` bash
cd ~/klipper
make clean
make menuconfig

```

*Note: Ensure you select **No bootloader**. If you do not want to use CAN bus, set the communication interface to **USBSERIAL**.*  
Press **ESC**, then **Y** to save. Run `make` to compile. This will generate a `klipper.uf2` file.

#### 2\. Enter BOOTSEL Mode

Connect the PUP board to your host via a USB cable. Press and hold the **BOOT** button, press the **RST** button once, wait for at least 1 second, and then release the **BOOT** button. The board is now in BOOTSEL mode.

#### 3\. Verify Device Connection

Run the following command in your host's SSH terminal:

``` bash
lsusb

```

If successful, you should see a device listed with the ID `2e8a:0003` (e.g., `Bus 006 Device 002: ID 2e8a:0003 Raspberry Pi RP2Boot`).

#### 4\. Flash the UF2 Firmware

  * **Via Command Line:** Klipper provides a script to automatically mount the RP2040 and copy the file. Run:

<!-- end list -->

``` bash
cd ~/klipper
make flash FLASH_DEVICE=2e8a:0003

```

This command searches for the RPI-RP2 block device under `/dev/sd*`, copies `out/klipper.uf2`, and triggers a reboot. Once rebooted, the **RUN LED** will turn on.

  * **Alternative (PC/Mac Drag & Drop):** If you encounter issues with the command line, connect the board to your PC/Mac via USB and enter BOOTSEL mode. The board will mount as a USB storage drive. You can copy the file from your Klipper host to your local computer using an SFTP client (like WinSCP or Cyberduck) from `~/klipper/out/`, and simply drag it into this drive. The device will automatically reboot, and the RUN LED will light up.

-----

### Method C: Flashing Katapult via USB & Updating via CAN Bus

Use this method if you need to manually reinstall the Katapult bootloader from scratch.

#### 1\. Install and Compile Katapult

``` bash
git clone [link removed]
cd katapult
make menuconfig

```

Configure your settings, then compile Katapult:

``` bash
make

```

This will generate a `katapult.uf2` (or `Canboot.uf2`) file.

#### 2\. Verify BOOTSEL Mode

Enter BOOTSEL mode (Hold **BOOT**, press **RST**, wait 1s, release **BOOT**). Check the connection:

``` bash
lsusb

```

Verify the `2e8a:0003 Raspberry Pi RP2Boot` device is listed.

#### 3\. Flash the Katapult Bootloader

Use Katapult's automatic mounting script:

``` bash
cd ~/katapult
make flash FLASH_DEVICE=2e8a:0003

```

This copies the `out/katapult.uf2` file to the board and triggers a reboot. Upon reboot, the **RUN LED** will start blinking, indicating a successful Katapult bootloader installation.  
*(Alternatively, you can use the PC/Mac Drag & Drop method to copy `katapult.uf2` directly into the mounted drive.)*

#### 4\. Install Klipper Firmware

Now that Katapult bootloader is installed, return to **Method A** to flash the actual Klipper firmware over the CAN bus.

-----

### 🛠️ Troubleshooting & Helpful Links

  * **Clearing the Flash:** If you are unable to successfully flash any uf2 files, the flash memory may need to be completely erased. Enter BOOTSEL mode and flash the `flash_nuke.uf2` file using the Drag & Drop method on your PC/Mac.
      * Download: [flash\_nuke.uf2 GitHub Repository](https://github.com/dwelch67/raspberrypi-pico/blob/main/flash_nuke.uf2)
  * **Related Community Resources:**
      * [CAN bus community setup reference](https://github.com/HRading/klipper_canbus_setup)
      * [SB2040 Katapult installation discussion](https://www.teamfdm.com/forums/topic/851-install-canboot-on-sb2040)

-----

## License

This project is licensed under the MIT License - see the LICENSE file for details.

-----

Would you like me to help you format this file further or structure a standard LICENSE file for your repository?
