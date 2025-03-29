![./images/t1 πro.jpg](https://raw.githubusercontent.com/mulcmu/T1-pyro/refs/heads/main/images/t1%20%CF%80ro.jpg)

# T1 πro
Document and share progress to convert a FLSUN T1 Pro to T1 πro.

### Objectives

- Replace the stock host board (Core Board) with a Raspberry Pi
- Replace the stock screen with Fysetc Raspberry Pi 5 inch DSI Display
- Document stock hardware connections for troubleshooting/upgrades
- Reuse other stock hardware as much as practical
- Clean installation
- Up to date OS and Klipper [Ecosystem](https://klipper.discourse.group/t/klipper-architecture-ecosystem/6313)

### Current Status

* Pi 4 and the screen installed, fully operational
* Uses the stock 5v power supply on lower adapter board for Pi
* USB camera hub board reused by Pi
* Currently <u>irreversible</u> after flashing Klipper MCU hardware 
* [[load cell]](https://github.com/garethky/klipper/tree/load-cell-probe-community-testing) community testing branch working with stock lower function adapter board
  * `before: range 0.046875, average -0.420110, stDev 0.010482, average delta: 0.011671`
  * `  after: range 0.002557, average -0.304737, stDev 0.000692, average delta: 0.000532`

* Macros need a lot of work
* Help wanted, contributions welcomed

### TODO

- [ ] Cleanup the printer.cfg file and macros
- [ ] Create Schematic (in progress)
- [x] Document lower adapter board
- [x] Fix lower adapter board usart1 rx/tx
- [x] Document effector PCB and determine cable pinout
- [x] Document πro custom cables
- [ ] Hotend thermistor noise at ambient temps
- [ ] Document modifications to improve bed probe [performance](https://klipper.discourse.group/t/load-cell-probing-algorithm-testing/9751/102)
- [ ] Slay underbed dragons (potential regions were probe will not trigger as two load cells readings are exactly balanced out by 3rd load cell reading)
- [ ] Modification for filament <u>movement</u> sensor
- [ ] Better camera (mine broke already)
- [ ] Chamber temperature, RH, sensor
- [ ] Chamber heater support
- [ ] Improve boot speed of Pi
- [ ] Does Pi need heatsinks or extra cooling
- [ ] Overlay file system or similar to make more robust to power loss
- [ ] Installation guides
- [ ] Logo
- [ ] Standard preinstalled Pi image
- [ ] Redesign the screen bracket
- [ ] Find stock firmware for the motherboard MCU
- [x] Stock firmware for the probe MCU

### STL Models

Fusion360 models and STL located on pintables:

https://www.printables.com/model/1191414-flsun-t1-pro-pi-and-screen-bracket

Some assorted M2, M2.5 and M3 screws are needed for assembly

### Custom Cables

* Custom cable from a Pi usb connector to the stock Usb hub board
* Custom cable from the stock lower adapter board 5v to the Pi *GPIO*
* [Details](custom_cables.md)

### Parts

* USB Extension cable for MCU ([Type A male to Type C female](https://www.amazon.com/dp/B07M999Y6T))
* Optional for Ethernet 
  * Low profile [connector cable](https://www.amazon.com/dp/B0D9JVSB2X?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_1) 
  * [Bulkhead connector](https://www.amazon.com/dp/B0D9JVSB2X)
* Industrial / rugged SD card for Pi
* SD card for flashing upper core board firmware
* ST Link v2 clone or similar [SWD debugger](https://github.com/WeActStudio/WeActStudio.MiniDebugger) (Required for [load_cell], maybe needed for motherboard MCU)

### Installation

Current high level guide to implement:

* Get Pi setup and Klipper installed and tested before physical install into machine
* Print the brackets for printer installation
* Make the custom cables
* Remove stock host PCB (Core board) and screen
* Install Pi and 5" screen
* Flash Klipper firmware to motherboard
* Flash Klipper firmware to lower function board

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

* Optional for [load_cell] support: switch to load-cell-probe-community-testing fork

    ```	
    cd ~/kiauh
    nano klipper_repos.txt
    ```

​	add to this file and save:

   ```
   garethky/klipper,load-cell-probe-community-testing
   ```

​	switch klipper branch in kiauh to the load cell testing repository


* Install extra python packages for Measuring Resonances and Load_Cell

  ```
  sudo apt install python3-numpy python3-matplotlib libatlas-base-dev libopenblas-dev python3-scipy
  ~/klippy-env/bin/pip install -v "numpy<1.26"
  ~/klippy-env/bin/pip install -v "scipy"
  ```

* Optional for [load_cell] support: Setup load cell [probe debug tool](https://observablehq.com/@garethky/klipper-load-cell-debugging-tool)

* Build motherboard MCU firmware

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
> Motherboard MCU menuconfig settings:
>
> * Enable extra low-level configuration options
> * Micro-controller Arch (STM32)
> * Processor model (STM32F103)
> * Bootloader offset (28KiB Bootloader)
> * Clock Reference (8 MHz)
> * Communication interface (Serial (on USART3 PB11/PB10))
> * (250000) Baud for serial port

* Optional for load cell support: Build lower function adapter board MCU firmware

  ```
  cd ~
  cd klipper
  make menuconfig
  make clean
  make
  cp out/klipper.bin out/load_cell.bin
  ```

> [!NOTE]
>
> Load Cell MCU menuconfig settings:
>
> * Enable extra low-level configuration options
> * Micro-controller Arch (STM32)
> * Processor model (STM32F103)
> * Bootloader offset (No Bootloader)
> * Clock Reference (8 MHz)
> * Communication interface (Serial (on USART1 PA10/PA9))
> * (250000) Baud for serial port

* Wifi enabled (optional)

    ``` 
    sudo raspi-config
    ```

    * Setup wifi under System Options
    * Select wifi country in Localizations Options
    * Enable serial uart, no console
      * Interface options
      * Serial Port
      * No login shell
      * Yes hardware enabled
    ```
    sudo nmtui
    ```

    * Check wifi setup and add additional networks if desired

* Enable host mcu
  * Setup hardware PWM using this guide https://www.klipper3d.org/RPi_microcontroller.html
  * The case light gets connected to PWM0 with it routed it to gpio12.

* Modify printer.cfg
  * Copy the printer.cfg from this repository and update with your existing calibration (endstops, bedmesh, PID, etc)
  * or.. edit your printer.cfg to remove all the FLSUN additions

* Make custom cables

* Print custom brackets

* Install pi in custom bracket

* Install screen in cutom bracket

* Remove the stock screen, host (core board) PCB, ribbon cable on lower adapter board, power cable from lower adapter board, cable from the usb hub

* Optional for load cell support:

    * Remove the J20 5 pin header pins and replace with right angle head pins on bottom of pcb.

    * A magnet/enamel wire needs added to jumper Pin 30 on STM32F103 to Pin 32

* Install the PI, slide over existing standoffs, secure with M3 screws

* Connect usb adapter extension from pi to the usb cable running to upper adapter board / motherboard

* Connect custom usb hub cable to pi usb and gpio

* (Optional) Connect ethernet cable and fit the bulkhead adapter somewhere.

* Connect the custom power cable from lower adapter board to GPIO

* Initial test, at this point pi and screen should boot when printer turned on.  Klipperscreen should load and show *MCU Protocol Error*.  The MCU firmware needs to be updated (or potentially Klipper downgraded) to be be compatible.

> [!WARNING]
>
> After flashing the Klipper MCU firmware returning printer to original stock configuration is not possible!
* Flash motherboard MCU
  * Copy the ~/klipper/out/Robin_nano35.bin file to an empty fat32 formatted sd card
  * With printer powered off, install the sd card in the motherboard
  * Power on printer and wait 30 seconds
  * Power off printer and remove the sd card
  * Robin_nano35.bin should be renamed to Robin_nano35.cur

* Optional for [load_cell] support: Use ST-Link v2 programmer or similar to flash the load_cell.bin firmware to the STM32
  * Configure [openocd](https://openocd.org/), [stlink tools](https://github.com/stlink-org/stlink), or similar for your programming adapter and OS
  * Connect STLINK v2 adapter to JTAG port on lower function adapter board
  * Save a copy of the existing firmware `st-flash read out.bin 0x0800000 0x20000`
  * Write the load_cell.bin file to the MCU `st-flash write load_cell.bin 0x08000000`
