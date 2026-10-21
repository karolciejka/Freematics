This Arduino sketch is developed for [Freematics ONE+](https://freematics.com/products/freematics-one-plus/) to collect vehicle telemetry data from OBD, GPS, motion sensor, to log the data in local storage and to transmit the data to a remote server in real-time. It demonstrates most capabilities of Freematics ONE+ and works well with [Traccar](https://www.traccar.org) GPS tracking platform.

Data Collection
---------------

The sketch collects following data.

* Vehicle OBD data (from OBD port)
* Battery voltage (from OBD port)
* Geolocation data (from internal or external GNSS) 
* Accelerometer and gyroscope data (from internal MEMS motion sensor)
* Cellular or WiFi network signal level
* Device temperature

Collected data are stored in a circular buffer in ESP32's IRAM or PSRAM. When PSRAM is enabled, hours of data can be buffered in case of temporary network outage and transmitted when network connection resumes.
  
Data Transmission
-----------------

Data transmission over UDP and HTTP(s) protocols are implemented for the followings.

* WiFi (ESP32 built-in)
* 3G WCDMA (SIM5360)
* 4G LTE CAT-4 (SIM7600)
* 4G LTE CAT-M (SIM7070)

UDP mode implements a telemetry client for [Freematics Hub](https://hub.freematics.com) and [Traccar](https://www.traccar.org). HTTP(s) mode implements [OsmAnd](https://www.traccar.org/osmand/) protocol with additional data sent as POST payload.

Seamless WiFi and cellular network co-working is implemented. When defined WiFi hotspot is available, data is transmitted via WiFi and cellular module is switched off. When no WiFi hotspot can be reached, cellular module is switched on for data transmission until WiFi hotspot available again.

Data Storage
------------

Following types of data storage are supported.

* MicroSD card storage
* ESP32 built-in Flash memory storage (SPIFFS)

BLE & App
---------

A BLE SPP server is implemented in [FreematicsPlus](https://github.com/stanleyhuangyc/Freematics/blob/master/libraries/FreematicsPlus) library. To enable BLE support, change ENABLE_BLE to 1 [config.h](config.h). This will enable remote control and data monitoring via [Freematics Controller App](https://freematics.com/software/freematics-controller/).

Commands are written as text to characteristic 0xFFE1 of service 0xABF0 and answered with a notification on 0xFFE2, terminated by `\r`. Besides the commands handled in `processBLE()`, two commands give apps direct OBD-II access (both answer `N/A` while OBD is not ready):

* `PIDS<pid><pid>...` (e.g. `PIDS0C0D05`) reads each mode 01 PID and answers with the hex data bytes of each reply, comma separated and empty for unsupported PIDs (e.g. `0C34,00,7B`).
* `RAW<request>` (e.g. `RAW0101`, `RAW03`, `RAW04`, `RAW020C00`) sends an OBD-II request (hex digits only, no AT commands) and answers with the reply with spaces removed and lines joined by `;`.

Prerequisites
-------------

* Freematics ONE+ [Model A](https://freematics.com/products/freematics-one-plus/), [Model B](https://freematics.com/products/freematics-one-plus-model-b/), [Model H](https://freematics.com/products/freematics-one-plus-model-h/)
* A micro SIM card if cellular network connectivity required
* [PlatformIO](http://platformio.org/), [Arduino IDE](https://github.com/espressif/arduino-esp32#installation-instructions), [Freematics Builder](https://freematics.com/software/arduino-builder) or [ESP-IDF](https://github.com/espressif/esp-idf) for compiling and uploading code
