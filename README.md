# ESP32-CAM XRZ00D1-V240 HTTP Server example with release/v4.1 of ESP-IDF
HTTP Camera Server example with latest ESP-IDF and chineese (clone?) Ai-Thinker ESP32-CAM module with XRZ00D1-V240 camera module

## What is ESP32-CAM?
It's module based on ESP32 SOC with camera and SD-card slot

## What is XRZ00D1-V240
It's OV2640 (or compatible?) camera module which you can get with latest ESP32-CAM modules from Aliexpress, Banggod, etc.

## Is there troubles?
Yep, it doesn't work out of the box with any example firmware. Module sells without any source codes ¯\_(ツ)_/¯

## How to build
1. You need release/v4.1 of the [ESP-ID SDK](https://github.com/espressif/esp-idf). [How to install it](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/index.html).
   Make sure to checkout the release/v4.1 branch before going through the install steps!
2. Clone repo
3. Run `idf.py menuconfig` and set WIFI SSID and password in the `Example Configuration` option (myssid/mypassword is default)
4. Run `idf.py build`

## How to flash
1. Short IO0 and GND on ESP-32 module to boot into flash mode
2. Connect ESP32-CAM to any serial adapter (U0R <— TX, U0T —> RX)
3. To flash you may run `idf.py -p /dev/ttyUSB0 flash` where `/dev/ttyXXX` is your serial port
4. After flashing you shoud unconnect IO0 and GND and push RESET button on module to boot new firmware

## How to check
1. You can use `idf.py monit` to check boot logs.  If this does not work you can also use screen
   `screen /dev/ttyUSB0 115200`
2. Connect over wifi to the network that the ESP 32 created.  This will be whatever you set the SSID to.
3. Open `http://192.168.4.1/jpg` to make shot 

## Troubleshooting
Check power source. My module works with 3.3v even 5v, but when powering with 3.3v there is artifacts on image.

If nothing works, try to flash pre-built binary with [esptool](https://github.com/espressif/esptool):

```
esptool.py -p YOUR_SERIAL_PORT_PATH -b 460800 --after hard_reset write_flash --flash_mode dio --flash_size detect --flash_freq 40m 0x1000 binary/bootloader.bin 0x8000 binary/partition-table.bin 0x10000 binary/ESP32-CAM_HTTP_AP.bin
```

## How it made?
Built with sources of [WiFi softAP example](https://github.com/espressif/esp-idf/tree/master/examples/wifi/getting_started/softAP), [Simple HTTPD Server Example](https://github.com/espressif/esp-idf/tree/master/examples/protocols/http_server/simple), [ESP32 Camera Driver](https://github.com/espressif/esp32-camera) (modified)

ESP32 Camera Driver modified due to this error while starting:
```
E (1710) ledc: requested frequency and duty resolution can not be achieved, try reducing freq_hz or duty_resolution. div_param=3
E (1720) camera_xclk: ledc_timer_config failed, rc=ffffffff
```

Need to move `periph_module_enable(PERIPH_LEDC_MODULE);` from top to end of function in `/components/esp32-camera/driver/xclk.c` file (thanks @masterd89 for [solution](https://github.com/espressif/esp32-camera/issues/66#issuecomment-526283681))

In menuconfig enabled PSRAM (Serial flasher config ---> Flash size (4 MB))

In menuconfig flash size setted to 4M (Component config ---> ESP32-specific ---> [*] Support for external, SPI-connected RAM)

