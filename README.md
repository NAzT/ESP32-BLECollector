# ESP32-BLECollector

A BLE Scanner with persistence.

  ![ESP32 BLECollector running on Wrover-Kit](https://raw.githubusercontent.com/tobozo/ESP32-BLECollector/master/screenshots/capture3.png) ![ESP32 BLECollector running on M5Stack](https://raw.githubusercontent.com/tobozo/ESP32-BLECollector/unstable/screenshots/BLECollector-M5Stack.jpeg)

🎬 [Demo video](https://youtu.be/w5V80PobVWs)
------------

BLECollector is just a passive BLE scanner with a fancy UI.
All BLE data found by the BLE Scanner is collected into a [sqlite3](https://github.com/siara-cc/esp32_arduino_sqlite3_lib) format on the SD Card.

Public Mac addresses are compared against [OUI list](https://code.wireshark.org/review/gitweb?p=wireshark.git;a=blob_plain;f=manuf), while Vendor names are compared against [BLE Device list](https://www.bluetooth.com/specifications/assigned-numbers/company-identifiers).

Those two database files are provided in a db format ([mac-oui-light.db](https://github.com/tobozo/ESP32-BLECollector/blob/master/SD/mac-oui-light.db) and [ble-oui.db](https://github.com/tobozo/ESP32-BLECollector/blob/master/SD/ble-oui.db)).

On first run, a default `blemacs.db` file is created, this is where BLE data will be stored.
When a BLE device is found by the scanner, it is populated with the matching oui/vendor name (if any) and eventually inserted in the `blemasc.db` file.

⚠️ This sketch is big! Use the "No OTA (Large Apps)" or "Minimal SPIFFS (Large APPS with OTA)" partition scheme to compile it.
The memory cost of using sqlite and BLE libraries is quite high.

⚠️ Builds using ESP32-Wrover can eventually choose the 3.6MB SPIFFS partition scheme, and have the BLECollector working without the SD Card. Experimental support only since SPIFFS tends to get slower and buggy when the partition becomes full.


Hardware requirements
---------------------
  - [mandatory] ESP32 or ESP32-WROVER (WROVER is recommended)
  - [mandatory] SD Card (breakout or bundled in Wrover-Kit, M5Stack, Odroid-Go, LoLinD32 Pro)
  - [mandatory] Micro SD (FAT32 formatted, **max 4GB**)
  - [mandatory] [mac-oui-light.db](https://github.com/tobozo/ESP32-BLECollector/blob/master/SD/mac-oui-light.db) and [ble-oui.db](https://github.com/tobozo/ESP32-BLECollector/blob/master/SD/ble-oui.db) files copied on the Micro SD Card root
  - [mandatory] ST7789/ILI9341 320x240 TFT (or bundled in Wrover-Kit, M5Stack, Odroid-Go, LoLinD32 Pro)
  - [optional] (but recommended) I2C RTC Module (see "#define HAS_EXTERNAL_RTC" in Settings.h)
  - [optional] Serial GPS Module (see "#define HAS_GPS" in Settings.h)

Software requirements
---------------------
  - [mandatory] Arduino IDE
  - [mandatory] [BLE Library](https://github.com/tobozo/ESP32-BLECollector/releases/download/1.2/BLE.zip) exposing a private method (supersedes the BLE (legacy) version, install it via the library manager)
  - [mandatory] [ESP32-Chimera-Core](https://github.com/tobozo/ESP32-Chimera-Core) (a substitute to M5Stack Core)
  - [mandatory] https://github.com/tobozo/M5Stack-SD-Updater
  - [mandatory] https://github.com/PaulStoffregen/Time
  - [mandatory] https://github.com/siara-cc/esp32_arduino_sqlite3_lib
  - [optional] https://github.com/mikalhart/TinyGPSPlus

Behaviours (auto-selected):
---------------------------
  - **Hobo**: when no TinyRTC module exists in your build, only uptime will be displayed
  - **Rogue**: TinyRTC module adjusted after flashing (build DateTime), shares time over BLE
  - **Chronomaniac**: TinyRTC module adjusts itself via GPS, shares time over BLE

Optional I2C RTC Module requirements
------------------------------------
  - Wire your TinyRTC to SDA/SCL (edit `Wire.begin()` in [Timeutils.h](https://github.com/tobozo/ESP32-BLECollector/blob/master/TimeUtils.h#L173) if necessary)
  - Insert the SD Card
  - Set `#define HAS_EXTERNAL_RTC true` in [Settings.h](https://github.com/tobozo/ESP32-BLECollector/blob/master/Settings.h)
  - Flash the ESP with partition scheme `Minimal SPIFFS (Large APPS with OTA)`

Optional Serial GPS Module requirements
---------------------------------------
  - Wire your GPS module to TX1/RX1 (edit `GPS_RX` and `GPS_TX` in GPS.h
  - Set `#define HAS_GPS true` in [Settings.h](https://github.com/tobozo/ESP32-BLECollector/blob/master/Settings.h)
  - Flash the ESP with partition scheme `Minimal SPIFFS (Large APPS with OTA)`
  - Wait for the GPS to find a fix
  - issue the command `gpstime` in the serial console

Time Sharing
------------
  - Once the time is set using RTC and/or GPS, the BLECollector will start the TimeSharing service and advertise a DateTime characteristic for other BLECollectors to sync with.
  - Builds with no RTC/GPS will try to identify this service during their scan duty cycle and subscribe for notifications.

File Sharing (still experimental)
------------
  - This feature is currently limited to sharing the two necessary .db files, if at least one of those files is missing or corrupted at boot, the BLECollector will start a BLE File server and wait for another BLECollector to send those files.
  - Sending files is still a manual operation, just issue the `blesend` command when another BLECollector is ready to receive the files.
  - Possible outcomes of this feature: sharing/propagating the collected data (e.g. whitelists/blacklists)

Contributions are welcome :-)


Known issues / Roadmap
----------------------
Because sqlite3 and BLE library already use a fair amount of RAM, there isn't much room left for additional tools such as Web Server, db parser, or NTP sync, so WiFi features are kept out of the scope of this project.

As a compensation, the M5Stack-SD-Updater is built-in and will allow hot-loading of any other .bin (default is menu.bin) located on the SD Card, making it easy to separate functional concerns into different apps.

Some ideas I'll try to implement in the upcoming changes:

- Add GPS Coords to entries for better pruning [as suggested by /u/playaspect](https://www.reddit.com/r/esp8266/comments/9s594c/esp32blecollector_ble_scanner_data_persistence_on/e8nipr6/?context=3)
- Have the data easily exported without removing the sd card (wifi, ble, serial)
- Auto downloading/refreshing sqlite databases


Other ESP32 security related tools:
-----------------------------------

  - https://github.com/cyberman54/ESP32-Paxcounter
  - https://github.com/G4lile0/ESP32-WiFi-Hash-Monster
  - https://github.com/justcallmekoko/ESP32Marauder


Credits/requirements:
---------------------

- https://github.com/siara-cc/esp32_arduino_sqlite3_lib
- thanks to https://github.com/chegewara for maintaining the [BLE library](https://github.com/tobozo/ESP32-BLECollector/releases/download/1.2/BLE.zip)
