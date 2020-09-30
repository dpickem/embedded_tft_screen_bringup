This tutorial describes the setup of a 2.8 inch TFT screen with builtin display controller and touch screen controller. Since the goal is to run GUIslice graphical menus, we forgo the setup on Arduino controllers since most microcontrollers from the Arduino family are too resource-constrained to run the more elaborate GUIslice examples (with the possible exception of the Arduino Mega or newer models). The focus instead lies on the ESP32 microcontroller which has both plenty of RAM and ROM as well as IO pins.

# Dependencies

The following dependencies can be installed via the Arduino IDE's library manager:

* Adafruit libraries
  * Adafruit_HX8357_Library (for use with Adafruit's 3.5" screen) 
  * Adafruit_TouchScreen
  * Adafruit_BusIO
* GUIslice
* MCUFRIEND_kbv
* TFT_eSPI (https://github.com/Bodmer/TFT_eSPI)
* XPT2046_Touchscreen

# Hardware

For this tutorial, only two pieces of hardware are required:

* ESP32 NodeMCU v1.1 development board
  * Datasheet: http://wiki.ai-thinker.com/_media/esp32/docs/nodemcu-32s_product_specification.pdf
  * Official documentation: https://nodemcu.readthedocs.io/en/dev-esp32/
  * Example product: https://www.amazon.com/HiLetgo-ESP-WROOM-32-Development-Microcontroller-Integrated/dp/B0718T232Z/ref=sr_1_2_sspa?dchild=1&keywords=esp32+nodemcu&qid=1601252894&sr=8-2-spons&psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUEyNE9SSE8yWDBCSldUJmVuY3J5cHRlZElkPUEwMTIwNjcwMk9EN1JKQTVSTVNSWiZlbmNyeXB0ZWRBZElkPUEwMjA4NDMxMk8zUVNWVlRSVEExTiZ3aWRnZXROYW1lPXNwX2F0ZiZhY3Rpb249Y2xpY2tSZWRpcmVjdCZkb05vdExvZ0NsaWNrPXRydWU=
* 2.8 inch screen with ILI9341 dislay controller and XPT2046 touch controller
  * Features
    * Compatible with Arduino UNO and Mega2560, and can be connected directly by inserting the pin into the interface without wire.
    * Compatible with all kinds of 5V or 3V MCU with 5V-3.3V change-over circuit.
    *  320X240 HD resolution, can be used as a touch screen.
    * Adopting 8-bit parallel bus, quicker and smoother refresh than SPI.
    *  With Micro-SD card circuit, easy to expand the scope of the test.
  * Specs
    * Type: Touch Screen Driver; ILI9341
    * Input Voltage: 3.3V / 5V
    * Size: 2.8" SPI Serial
    * Display area: 36.72(W)X48.96(H)mm
    * Size: 8.5 x 4.8cm
    * Driver element: a-Si TFT active matrix
    * Pixel arrangement: RGB vertical stripe
    * Driver IC: ILI9341
    * Backlight: White LED
    * Viewing Direction: 6 o'clock
    * Color Depth: 65K
    * Resolution (dots): 240RGB*320Dots
    * 5V compatible, use with 3.3V or 5V logic
    * Need at least 4 IOs from your MCU
    *Interface: Analog serial port / SPI interface
  * Example product
    * https://www.amazon.com/gp/product/B087C3PP9G/ref=ppx_yo_dt_b_asin_title_o03_s01?ie=UTF8&psc=1
    * https://www.amazon.com/gp/product/B08C53X3RF/ref=ppx_yo_dt_b_asin_title_o03_s00?ie=UTF8&psc=1

This particular screen can be connected to the ESP32 via SPI (i.e. 3 pins - CLK, MOSI, MISO) and a handful of control pins (D/C, chip select, reset). The benefit of the builtin touch controller is that it only requires one additional pin (a chip select pin for the XPT2046 touch controller) while it connects to the same SPI bus the display controller is on.

The particular TFT screen in this tutorial has a dedicated touch controller and can therefore share the SPI bus. However, even if such a dedicated controller were missing, most TFT touch screens offer a 4-pin interface to connect the touch screen directly to the microcontroller. Those interfaces (typically indicated via 4 pins labeled X+, X-, Y+, Y-) require two analog pins as well as two digital pins, which eliminates the ESP8266 microcontroller from consideration (since it only offers a single analog input pin).

Note that similar ESP32 boards as well as screens should work just as well. However, the pinout of the microcontroller and screen may differ and require to adjust the wiring.

# Wiring

The wiring tables below describe uses the SPI interface of the display. Some displays come with both an SPI interface as well as an 8-bit interface. The advantage of the 8-bit parallel interface is speed at the expense of requiring 8 pins for data alone (plus all the required control pins). The display used in this tutorial only provides an SPI interface.

## Display (ILI9341)

| ESP32 pin   | Display Pin |  Description  |
| ----------- | ----------- | ------------- |
|  GPIO 19    |  SDO (MISO) |  (SPI)        |
|  VCC        |  LED        |  connect to VCC |
|  GPIO 18    |  SCK        |  (SPI)        |
|  GPIO 23    |  SDI (MOSI) |  (SPI)        |
|  GPIO  2    |  DC         |               |
|  GPIO  4    |  RESET      |  not required |
|  GPIO  5    |  CS         |  (SPI)        |
|  GND        |  GND        |               |
|  VCC        |  VCC        |  3.3 or 5V    |

## Touch controller (XPT2046)

| ESP32 pin   | Display Pin |  Description |
| ----------- | ----------- | ------------ |
|             |  T_IRQ      |  not needed  |
| GPIO 19     |  T_DO       |  MISO (SPI)  |
| GPIO 23     |  T_DIN      |  MOSI (SPI)  |
| GPIO  3     |  T_CS       |  chip select | 
| GPIO 18     |  T_CLK      |  SCK  (SPI)  |

Note that the ESP32 has two SPI interfaces (VSPI and HSPI). The wiring above uses the VSPI interface.

# Testing and Setup

Before GUIslice can be setup with a new display it is recommended to test the wiring and display hardware setup with lower level libraries such as Adafruit's library (graphicstest as part of Adafruit HX8357 library) or the TFT_eSPI (which requires somewhat more involved configuration but is a prerequisite for GUIslice if the display is used in connection with an ESP32 (see https://github.com/ImpulseAdventure/GUIslice/wiki/Configure-GUIslice-for-ESP8266-&-ESP32).

The TFT_eSPI config is included in the TFT_eSPI_config directory and needs to be included in TFT_eSPI's main configuration file (User_Setup.h). The configuration file is derived from the many provided examples that are part of the TFT_eSPI library and has been trimmed down to include only the necessary options for the ESP32, ILI9341, and XPT2046 combination of devices.

# Touch Configuration

The GUIslice library provides two examples that facilitate the touch screen configuration.

* diag_ard_touch_detect
* diag_ard_touch_calib

The first sketch for detecting the touch controller is not necessary since the XPT2046 controller is expected to be connected via SPI and one additional chip select pin. The second sketch, however, is required for calibrating the touch screen, i.e. determining x_min, x_max, y_min, and y_max coordinates of the screen as well as associated analog measurement. In addition that sketch determines the orientation of the touch screen with respet to the actual display, which need to be synchronized since the orientation of the screen can be set independently of the touch screen. An example output of the calibration sketch looks as follows and needs to be included in the user-specific GUIslice configuration file (see the GUIslice_config directory):

    // Calibration settings from diag_ard_touch_calib:
    // DRV_TOUCH_XPT2046_PS [240x320]: For 2.8" screen
    #define ADATOUCH_X_MIN    3869
    #define ADATOUCH_X_MAX    230
    #define ADATOUCH_Y_MIN    3911
    #define ADATOUCH_Y_MAX    340
    #define ADATOUCH_REMAP_YX 0

# References
* GUISlice
  * Overview (https://github.com/ImpulseAdventure/GUIslice/wiki)
  * Gallery of examples (https://github.com/ImpulseAdventure/GUIslice/wiki/Gallery-of-Examples)
  * Configuring GUIslice (https://github.com/ImpulseAdventure/GUIslice/wiki/Configuring-GUIslice)
  * Configuring GUIslice with ESP32 (https://github.com/ImpulseAdventure/GUIslice/wiki/Configure-GUIslice-for-ESP8266-&-ESP32)
  * Configuring Touch support for GUIslice (https://github.com/ImpulseAdventure/GUIslice/wiki/Configure-Touch-Support)
