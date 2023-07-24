| Supported Targets | ESP32 | ESP32-C3 | ESP32-S2 | ESP32-S3 |
| ----------------- | ----- | -------- | -------- | -------- |
# Lab 4

This lab looks at another approach for connecting devices to a microcontroller, namely SPI (Serial Peripheral Interface). SPI is a full duplex serial protocol (I2C is half duplex), that uses 4 dedicated I/O lines. 3 of these common to all devices, while a dedicated CS (chip select) line is used by each device. The ESP32 can operate as both a Master and a Slave, although it is most often used as a Master. The ESP32-S3 has 4 [SPI peripherals](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-reference/peripherals/spi_master.html). Two are used to access internal memory and the other 2 are available for general purpose use. One can connect with up to 6 slaves and the other can connect with up to 3 slaves. Additional detials about SPI may be found [here](https://www.corelis.com/education/tutorials/spi-tutorial/).

The example code is derived from two different examples provided by Espressif. These are the [SDSPI SD card example](https://github.com/espressif/esp-idf/tree/master/examples/storage/sd_card/sdspi) and the [Deep sleep example](https://github.com/espressif/esp-idf/tree/master/examples/system/deep_sleep).

The example demonstrates how to use an SD card (SDSC, SDHC, SDXC) with an ESP device over an [SPI interface](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-reference/peripherals/sdspi_host.html). The example does the following steps:

1. Use an "all-in-one" `esp_vfs_fat_sdspi_mount` function to:
    - initialize SDSPI peripheral,
    - probe and initialize the card connected to SPI bus (DMA channel 1, MOSI, MISO and CLK lines, chip-specific SPI host id),
    - mount FAT filesystem using FATFS library (and format card, if the filesystem cannot be mounted),
    - register FAT filesystem in VFS, enabling C standard library and POSIX functions to be used.
2. Print information about the card, such as name, type, capacity, and maximum supported frequency.
3. Create a file using `fopen` and write to it using `fprintf`.
4. Close the file and unmount the SD Card

These steps happen each time the ESP32 is powered, reset, or woken up from deep sleep. The [deep sleep mode](https://docs.espressif.com/projects/esp-idf/en/latest/api-reference/system/sleep_modes.html#sleep-modes) is a power saving mode that causes the CPU, majority of RAM, and digital peripherals that are clocked from APB_CLK to be powered off. Deep sleep mode can be exited using one of multiple [wake up sources](https://docs.espressif.com/projects/esp-idf/en/latest/api-reference/system/sleep_modes.html#wakeup-sources). This example demonstrates how to use the [`esp_sleep.h`](https://docs.espressif.com/projects/esp-idf/en/latest/api-reference/system/sleep_modes.html#api-reference) API to enter deep sleep mode, then wake up form different sources.

The following wake up sources are demonstrated in this example (refer to the [Wakeup Sources documentation](https://docs.espressif.com/projects/esp-idf/en/latest/api-reference/system/sleep_modes.html#wakeup-sources) for more details regarding wake up sources):

- **Timer:** An RTC timer that can be programmed to trigger a wake up after a preset time. This example will trigger a wake up every 20 seconds.
- **EXT0:** External wake up 0 can trigger wakeup when one predefined RTC GPIO is at a predefined logic level. This example uses a GPIO pin in the range 1 to 21 to trigger a wake up when the pin is LOW (default) or HIGH. Note that this wake up source is only available on ESP32, ESP32-S2, and ESP32-S3.
- **EXT1:** External wake up 1 which is tied to multiple RTC GPIOs. This example uses GPIO16 and GPIO17 to trigger a wake up with any one of the two pins are HIGH. Note that this wake up source is only available on ESP32, ESP32-S2, and ESP32-S3.

Other modes that are not demonstrated include:
- **Touch:** Touch pad sensor interrupt.
- **ULP:** Ultra Low Power Coprocessor which can continue to run during deep sleep. 

Note: Some wake up sources can be disabled via configuration. Wake up sources that are unused or unconnected should be disabled in configuration to prevent inadvertent triggering of wake up as a result of floating pins.

<!---
In this example, the `CONFIG_BOOTLOADER_SKIP_VALIDATE_IN_DEEP_SLEEP` Kconfig option is used, which allows you to reduce the boot time of the bootloader during waking up from deep sleep. The bootloader stores in rtc memory the address of a running partition and uses it when it wakes up. This example allows you to skip all image checks and speed up the boot.
--->

Finally, while it is possible to connect an SD card breakout adapter (as is done in this lab), keep in mind that connections using breakout cables are often unreliable and have poor signal integrity. You may need to use lower clock frequency when working with SD card breakout adapters. It is recommended to get familiar with [the document about pullup requirements](https://docs.espressif.com/projects/esp-idf/en/latest/api-reference/peripherals/sd_pullup_requirements.html) to understand Pullup/down resistor support and compatibility of various ESP modules and development boards.


### Hardware Required

* An ESP32-S3-DevKitC-1 development board
* A USB cable for microcontroller power supply and programming
* 8 GB SDHC MicroSD card
* MicroSD card adapter
* Four 10 kΩ 1/4 W resistors
* Three NO pushbuttons
* A solderless breadboard
* Jumper or Dupont wires (M/M, M/F) to connect components

Additional hardware used for lab exercise:

* GY-2561 TSL2561 I2C digital luminosity sensor 
* Two waterproof DS18B20 1-wire digital temperature sensors
* One 4.7 kΩ 1/4 W resistor

Schematic:
![Lab 4 basecode schematic](doc/Lab4-Basecode.png)

### Build and Flash
Run `idf.py menuconfig` in the project directory and open the "Lab 4 Example Configuration" menu to configure the peripherals, ensuring that the pin assignments match those in the schematic.

Run `idf.py -p PORT flash monitor` to build, flash and monitor the project.

(To exit the serial monitor, type ``Ctrl-]``.)

See the [Getting Started Guide](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/index.html) for full steps to configure and use ESP-IDF to build projects.

Alternatively, this project can be developed using [VSCode](https://code.visualstudio.com) with the [PlatformIO IDE](https://platformio.org/platformio-ide) extension and the [Espressif 32 platform](https://registry.platformio.org/platforms/platformio/espressif32) installed.

## Example Output

See example output [here](doc/Output-Basecode.txt)

An example of data written to the card is [here](doc/FOO.TXT)

## Exercise

Build the circuit and insert the 8 GB MicroSD card in the MicroSD card adapter. Load the example code and confirm that the SD card connects, can be initialized, and that data can be written to it.  Use the IDF Monitor to observe the different sleep wake up conditions (EXT0, EXT1, and timer). You should see output that is similar to [this](doc/Output-Basecode.txt). After a number of boot cycles, remove the MicroSD card and use an SD card reader on a PC to read the data that was written to the MicroSD card. Verify that it is similar to this [FOO.TXT](doc/FOO.TXT). Once the example is working properly, modify the example code to add the following functionality:

1. Add a TSL2561 light sensor and two DS18x20 temperature sensors to the circuit.
![Exercise 1 schematic](doc/Lab4-Exercise1.png)
Read the sensor values and output them to the SD card in [CSV format](https://docs.fileformat.com/spreadsheet/csv/), including the header row:

| Boot | Cause | Time | Lux | Temp 1 | Temp 2 |
|------|-------|------|-----|--------|--------|

Where the boot number, wakeup cause, timestamp, and sensor data are written to the file on each scheduled (timer-based) and unscheduled (EXT0 or EXT1) wake up. 
See [FOO.CSV](doc/FOO.CSV) for an example. 

Helpful tips for formatting text in printf statements are presented [here](https://learn.microsoft.com/en-us/cpp/c-runtime-library/format-specification-syntax-printf-and-wprintf-functions).

Note that FreeRTOS tasks do not run properly when the ESP32 goes to sleep, so the relevant code used for the sensors in Labs 2 and 3 will need to be modified to run in `app_main` directly.

2. Change the deep sleep to a light sleep. The ESP32 sleep modes are described [here](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-reference/system/sleep_modes.html). Since the example code (using `esp_deep_sleep_start`) assumes that the entire system is rebooted upon wake up, the code in `app_main` will need to be modified to use a while loop. Note that some tasks (such as initialization of the I2C bus and SD card) should only be executed once, not every wake up cycle. How does the use of light sleep affect the wake up latency?

3. Use the addressable Smart LED that is integrated into the ESP32-S3-DevKitC-1 board (connected to GPIO48) to  periodically indicate proper operation while reading and writing data. Most of the time (while the ESP32 is asleep) the LED should be off. Otherwise, the LED should display a colour as follows:

| ESP32 Operation      | Colour |
|----------------------|--------|
| Initialization       | Orange |
| Reading/writing data | Green  |

You may use code that is similar to what you used for Lab 3 to drive the integrated LED.

4. Replace the pushbutton input for EXT0 with the INT output from the TSL2561.
![Exercise 4 schematic](doc/Lab4-Exercise4.png)
To use the TSL2561 interrupt, you will need to be sure to use the updated TSL2561 component in [Lab3-Basecode](https://github.com/MME9654/Lab3-Basecode). Use the following [TSL2561](https://cdn-shop.adafruit.com/datasheets/TSL2561.pdf) configuration parameters:

| Parameter | Value |
|-----------|-------|
| Gain | 1X |
| Integration time | 101 ms |
| Interrupt control | Level |
| Interrupt persistence | 2 integration periods |

In addition, a suitable low and high threshold should be set so that interrupts will be triggered. The low and high thresholds can be set using the `tsl2561_set_low_threshold` and `tsl2561_set_high_threshold` functions. The `tsl2561_calc_raw_ch0` utility function may be used to approximate the 16-bit CH0 value that corresponds to a target light intensity in lux.

Note that once the INT pin goes LOW, the interrupt control bit on the TSL2561 must be cleared using the `tsl2561_clear_interrupt` function. It is important that this is done at an appropriate time, such as shortly before going to sleep. You may find it helpful to use an oscilloscope to monitor the state of the TSL2561 INT pin so that you can ensure that it is operating properly.

### Other Things to Explore
1. Adjust wakeup timer to account for interrupt driven events. Should maintain regular sampling intervals (e.g., 10 seconds) as much as possible.
2. Experiment with different combinations of integration time and persistence values, e.g. exceed threshold for 50 ms.
3. Use the pushbuttons to start and stop data collection, or adjust operating parameters such as the data collection interval or interrupt thresholds.
4. Create a unique name for the data file so that the existing one is not overwritten.
5. Add an OLED display to aid in configuration and information display.

## Troubleshooting

### Failure to mount filesystem

> The following error message is printed: `example: Failed to mount filesystem. If you want the card to be formatted, set the CONFIG_EXAMPLE_FORMAT_IF_MOUNT_FAILED menuconfig option.`

The example will be able to mount only cards formatted using FAT32 filesystem. If the card is formatted as exFAT or some other filesystem, you have an option to format it in the example code. Enable the `CONFIG_EXAMPLE_FORMAT_IF_MOUNT_FAILED` menuconfig option, then build and flash the example.
