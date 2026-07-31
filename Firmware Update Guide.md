

## Firmware Update Guide[cite: 5]

PUP boards ship pre-flashed with the latest Katapult bootloader and Klipper firmware[cite: 5]. Under normal circumstances, no firmware updates are required before installation and use[cite: 5]. 

If you need to update firmware, please choose from the methods below[cite: 5].

### Method A: Updating via CAN Bus using Katapult bootloader (Recommended)[cite: 5]
This is the recommended method for updating Klipper firmware over the CAN interface[cite: 5].

**1. Install Katapult**[cite: 5]
* Ensure Katapult is installed on your Klipper host[cite: 5].
* (Reference: Katapult GitHub Repository)[cite: 5].

**2. Compile Klipper Firmware**[cite: 5]
* Access your Klipper host via SSH and run the following commands[cite: 5]:
```

cd ~/klipper
make clean
make menuconfig

```
<img width="941" height="415" alt="image" src="https://github.com/user-attachments/assets/fe00de15-c628-464e-89e5-8b12c8472083" />


* Note: Please refer to the provided configuration image for exact settings.


* You must select Bootloader offset (16KiB bootloader).


* Once configured, press ESC to exit and Y to save.


* Then, compile the firmware:



```
make

```

* This will generate the klipper.bin file.
<img width="613" height="513" alt="image" src="https://github.com/user-attachments/assets/1f7d4250-6912-4d1a-9c57-95c783127add" />



**3. Flash the Firmware**

* Ensure the target PUP board is powered on and visible on the CAN bus.


* Double-click the RST button on the PUP board.


* The RUN LED will begin blinking, indicating the board has entered Katapult bootloader mode.


* Query the bus to find the bootloader UUID:



```bash
python3 ~/katapult/scripts/flashtool.py -i can0 -q

```

* Locate and record the UUID where the application state is listed as Application: Katapult.


* Flash the new firmware using your recorded UUID:



```bash
python3 ~/katapult/scripts/flashtool.py -i can0 -u <your_uuid> -f ~/klipper/out/klipper.bin

```

* Once the process completes, the flash is successful.
<img width="1102" height="695" alt="image" src="https://github.com/user-attachments/assets/3f0c27d0-a982-47cf-98fd-cb5fcdbf80b4" />



---

### Method B: Direct Flashing via USB



This method completely overwrites the flash memory and does not use the Katapult bootloader. This is generally not recommended, as all future updates will require a physical USB connection unless you specifically intend to use the PUP board over a USB communication interface instead of CAN bus.

**1. Compile Klipper Firmware**

```bash
cd ~/klipper
make clean
make menuconfig

```

* Note: Refer to the configuration image.



* Ensure you select No bootloader.


* If you do not want to use CAN bus, set the communication interface to USBSERIAL.
<img width="1056" height="443" alt="image" src="https://github.com/user-attachments/assets/48206e54-5c63-45cf-af69-bb1af7a3fbff" />


* Press ESC, then Y to save.


* Run make to compile.


* This will generate a klipper.uf2 file.
<img width="885" height="764" alt="image" src="https://github.com/user-attachments/assets/f22bcf19-969e-4f50-91c8-d52a3c38c678" />



**2. Enter BOOTSEL Mode**

* Connect the PUP board to your host via a USB cable.


* Press and hold the BOOT button, press the RST button once, wait for at least 1 second, and then release the BOOT button.


* The board is now in BOOTSEL mode.



**3. Verify Device Connection**

* Run the following command in your host's SSH terminal:



```bash
lsusb

```

* If successful, you should see a device listed with the ID 2e8a:0003 (e.g., Bus 006 Device 002: ID 2e8a:0003 Raspberry Pi RP2Boot).
<img width="1023" height="348" alt="image" src="https://github.com/user-attachments/assets/56424dc3-96b9-4daa-9f3e-3b058527c9a2" />



**4. Flash the UF2 Firmware**

* Klipper provides a script to automatically mount the RP2040 and copy the file.


* Run:



```bash
cd ~/klipper
make flash FLASH_DEVICE=2e8a:0003

```

* This command searches for the RPI-RP2 block device under /dev/sd*, copies out/klipper.uf2, and triggers a reboot.
<img width="1023" height="443" alt="image" src="https://github.com/user-attachments/assets/d9218783-ee2a-43d5-84c5-6c3d5c8b00d4" />


* Once rebooted, the RUN LED will turn on.


* Alternative (PC/Mac Drag & Drop): If you encounter issues with the command line, connect the board to your PC/Mac via USB and enter BOOTSEL mode (Step 2).


* The board will mount as a USB storage drive.


* Simply copy the klipper.uf2 file from your Klipper host and drag it into this drive.


* The device will automatically reboot, and the RUN LED will light up to confirm success.



---

### Method C: Flashing Katapult via USB & Updating via CAN Bus



Use this method if you need to manually reinstall the Katapult bootloader from scratch.

**1. Install and Compile Katapult**

```bash
git clone [https://github.com/Arksine/katapult](https://github.com/Arksine/katapult)
cd katapult
make menuconfig

```

* Note: Configure the settings according to the provided reference image.
<img width="865" height="504" alt="image" src="https://github.com/user-attachments/assets/b4b09863-b2bd-4a90-8097-f35a8a434072" />


* Compile the Katapult:



```bash
make

```

* This will generate a katapult.uf2 (or Canboot.uf2) file.

<img width="777" height="718" alt="image" src="https://github.com/user-attachments/assets/b734f0fb-5d41-4936-adc5-fcabf657f826" />


**2. Verify BOOTSEL Mode**

* Enter BOOTSEL mode (Hold BOOT, press RST, wait 1s, release BOOT).


* Check the connection:



```bash
lsusb

```

* Verify the 2e8a:0003 Raspberry Pi RP2Boot device is listed.



**3. Flash the Katapult Bootloader**

* Use Katapult's automatic mounting script:



```bash
cd ~/katapult
make flash FLASH_DEVICE=2e8a:0003

```
<img width="865" height="462" alt="image" src="https://github.com/user-attachments/assets/d91f9022-ec21-4886-8616-657adb8af68c" />

* This copies the out/katapult.uf2 file to the board and triggers a reboot.


* Upon reboot, the RUN LED will start blinking, indicating a successful Katapult bootloader installation.


* Alternative (PC/Mac Drag & Drop): You can also connect to a PC/Mac, enter BOOTSEL mode, and drag-and-drop the katapult.uf2 file directly into the mounted storage drive.


* The RUN LED will blink upon a successful reboot.



**4. Install Klipper Firmware**

* Now that Katapult bootloader is installed, return to Method A to flash the actual Klipper firmware over the CAN bus.



---

### Related links



* https://github.com/HRading/klipper_canbus_setup
* https://www.teamfdm.com/forums/topic/851-install-canboot-on-sb2040
* https://github.com/dwelch67/raspberrypi-pico/blob/main/flash_nuke.uf2



### Appendix: Downloading Firmware to your PC/Mac



* If you need to flash the firmware via the USB Drag & Drop method, you must first copy the generated .uf2 file from your Klipper host to your local computer.


* You can do this using one of the following methods:


* Using an SFTP Client (Recommended for Windows/Mac)


* Download and install an SFTP client like WinSCP (Windows) or Cyberduck (Mac).


* Create a new SFTP connection using your Klipper host's IP address, username (e.g., pi, biqu, or mks), and password.


* Once connected, navigate to the Klipper or katapult output directory:


* /home/<your_username>/klipper/out/ (or ~/klipper/out/)


* /home/<your_username>/katapult/out/ (or ~/cd katapult/out/).




* Locate the klipper.uf2, or katapult.uf2 file and drag it to your local desktop.



### Troubleshooting Tip: Clearing the Flash



* If you are unable to successfully flash any uf2 files, the flash memory may need to be completely erased.


* You can do this by entering BOOTSEL mode and flashing the flash_nuke.uf2 file using the Drag & Drop method on your PC/Mac.


* (Download here: https://github.com/dwelch67/raspberrypi-pico/blob/main/flash_nuke.uf2)


* If you select the "Bootloader offset (16KiB bootloader)" option when compiling the Klipper firmware, the compilation process will generate a `.bin` file instead of a `.uf2` file, meaning you cannot flash the firmware via drag-and-drop.


* You must use Katapult's `flashtool.py` to flash the firmware.



```

```
