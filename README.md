# T1 πro
Work in progress to convert a FLSUN T1 Pro to T1 πro:

- Replace the stock host board with a Raspberry Pi
- Replace the stock screen with Fysetc Raspberry Pi 5 inch DSI Display
- Document stock hardware connections for troubleshooting/upgrades
- Reuse other stock hardware as much as practical
- Clean installation
- Up to date OS and Klipper [Ecosystem](https://klipper.discourse.group/t/klipper-architecture-ecosystem/6313)
- Currently irreversible after flashing Klipper MCU hardware 

### Current Status

* Pi 4 and the screen installed, fully operational
* Uses the stock 5v power supply on lower core board
* USB camera hub board reused by Pi
* Macros need a lot of work
* Help wanted, contributions welcomed

### TODO

- [ ] Cleanup the printer.cfg file and macros
- [ ] Document upper core board MCU / PCB
- [ ] Document lower core board
- [ ] Document effector PCB and determine cable pinout
- [ ] Document custom cables
- [ ] Modifications to improve bed probe [performance](https://klipper.discourse.group/t/load-cell-probing-algorithm-testing/9751/102)
- [ ] Modification for filament <u>movement</u> sensor
- [ ] Better camera (mine broke already)
- [ ] Chamber temperature, RH, sensor
- [ ] Chamber heater support
- [ ] Improve boot speed of Pi
- [ ] Does Pi need heatsinks or extra cooling
- [ ] Overlay file system or similar to make more robust to power loss
- [ ] Installation guides
- [ ] Standard preinstalled Pi image
- [ ] Redesign the screen bracket
- [ ] Find stock firmware for the MCU

### STL Models

Fusion360 models and STL located on pintables:

https://www.printables.com/model/1191414-flsun-t1-pro-pi-and-screen-bracket

Some assorted M2, M2.5 and M3 screws are needed for assembly

### Custom Cables / Parts

* Custom cable from a Pi usb connector to the stock Usb hub board
* Custom cable from the stock lower core board 5v to the Pi *GPIO*
* USB Extension cable for MCU
* Optional: Ethernet and bulkhead adapter
* Industrial / rugged SD card

### Installation

Current high level guide to implement:

* Get Pi setup and Klipper installed and tested before physical install into machine
* Print the brackets for printer installation
* Make the custom cables
* Remove stock host PCB and screen
* Install Pi and 5" screen
* Flash Klipper firmware to upper core board

Details:

* Fresh Bookworm Install:  Use Raspberry Pi Imager software to install latest Bookwork Lite Os to SD card

* Modify the Pi config.txt

    ```
    #Enable i2c for future use
    dtparam=i2c_arm=on
    
    # Enable pwmchip sysfs interface for hardware PWM on the GPIO pins
    dtoverlay=pwm-2chan,pin=12,func=4,pin2=13,func2=4
    
    #Not likely to use cable to Pi camera interface
    camera_auto_detect=0
    
    #Screen changes
    
    # Enable DRM VC4 V3D driver
    # dtoverlay=vc4-kms-v3d
    # max_framebuffers=2
    
    disable_fw_kms_setup=1
    
    #Disable compensation for displays with overscan
    disable_overscan=1
    ```

> [!NOTE]
>
> Potential issues with touchscreen and [wifi firmware](https://github.com/KlipperScreen/KlipperScreen/discussions/1491).  I did not encounter any issues with the wifi.
>
> ``` Remove problematic wifi
> sudo apt remove firmware-brcm80211
> sudo apt-mark hold firmware-brcm80211
> sudo reboot now 
> ```

* First boot, use SSH or usb keyboard to update

  ``` 
  sudo apt update
  sudo apt upgrade
  sync
  sudo reboot now
  ```

* Install ts_lib for touch calibration

  ```
  sudo apt-get install libts-bin
  sudo ts_calibrate
  sudo ts_test
  sync
  sudo reboot now
  ```

* Install Git, KIAUH, Klipper, moonraker, mainsail, and klipperscreen

    ```
    sudo apt-get install git
    cd ~ 
    git clone https://github.com/dw-0/kiauh.git
    ./kiauh/kiauh.sh
    ```

* Build MCU firmware

  ```
  cd ~
  cd klipper
  make menuconfig
  make clean
  make
  ./scripts/update_mks_robin.py out/klipper.bin out/Robin_nano35.bin
  ```

  > [!NOTE]
  >
  > menuconfig settings:
  >
  > * Enable extra low-level configuration options
  > * Micro-controller Arch (STM32)
  > * Processor model (STM32F103)
  > * Bootloader offset (28KiB Bootloader)
  > * Clock Reference (8 MHz)
  > * Communication interface (Serial (on USART3 PB11/PB10))
  > * (250000) Baud for serial port

* Wifi enabled (optional)

    ``` 
    sudo raspi-config
    ```

    * Setup wifi under System Options
    * Select wifi country in Localisation Options
    ```
    sudo nmtui
    ```

    * Check wifi setup and add additional networks if desired

* Enable host mcu
  * Setup hardware PWM using this guide https://www.klipper3d.org/RPi_microcontroller.html

* Modify printer.cfg
  * Copy the printer.cfg from this repository and update with your existing calibration (endstops, bedmesh, PID, etc)
  * or.. edit your printer.cfg to remove all the FLSUN additions

* Make custom cables

* Print custom brackets

* Install pi in custom bracket

* Install screen in cutom bracket

* Remove the stock screen, host PCB, ribbon cable on lower core board, power cable from lower core board, cable from the usb hub

* Install the PI, slide over existing standoffs, secure with M3 screws

* Connect usb adapter extension from pi to the usb cable running to upper core board

* Connect custom usb hub cable to pi usb and gpio

* (Optional) Connect ethernet cable and fit the bulkhead adapter somewhere.

* Connect the custom power cable from lower core board to GPIO

* Initial test, at this point pi and screen should boot when printer turned on.  Klipperscreen should load and show *MCU Protocol Error*.  The MCU firmware needs to be updated (or Klipper downgraded) to be be compatible

> [!WARNING]
>
> After flashing the Klipper MCU firmware returning printer to original stock configuration is not possible!
* Flash upper core board MCU
  * Copy the ~/klipper/out/Robin_nano35.bin file to an empty fat32 formatted sd card
  * With printer powered off, install the sd card in the upper core board
  * Power on printer and wait 30 seconds
  * Power off printer and remove the sd card
  * Robin_nano35.bin should be renamed to Robin_nano35.cur
