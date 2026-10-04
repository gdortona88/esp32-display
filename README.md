# Reflective Dashboard

A single-file Arduino dashboard for the **Waveshare ESP32-S3-RLCD-4.2**: a large clock, hourly weather, local temperature and humidity, Prague public transport, nearby aircraft and news on a reflective monochrome LCD.

**Firmware: v1.4.0** · **400 × 300 landscape display** · **Browser-based setup and OTA updates**

## Pages

| Page | What it shows |
| --- | --- |
| Clock | Large date/time, the next five hourly forecasts with day/night icons, temperature and rain probability, plus local sensor readings. `+1` marks tomorrow. |
| Weather | Three-day forecast and a local temperature/humidity history view. |
| Transport | PID / Golemio departures for a configured Prague-region stop. |
| Aircraft | Nearby aircraft, callsign, altitude, distance and available route information; automatic carousel every 10 seconds. |
| News | Headlines from an HTTPS RSS 2.0 feed; BBC Europe by default. |
| System | Network information, RTC status, battery voltage estimate and diagnostics. |

The onboard SHTC3 measures **local** temperature and humidity. Outdoor forecasts come from Open-Meteo. Address search updates the location used for weather and nearby aircraft and helps select a nearby PID stop. Public transport coverage is limited to the PID network.

## Hardware

- Waveshare ESP32-S3-RLCD-4.2, ESP32-S3 with 16 MB flash and 8 MB OPI PSRAM.
- ST7305 reflective monochrome LCD, 300 × 400 native resolution.
- SHTC3 temperature/humidity sensor and PCF85063 RTC.
- USB power or the board's supported 18650 battery arrangement.

This sketch targets this board's wiring, not a generic ESP32 display module. See the [Waveshare board repository](https://github.com/waveshareteam/ESP32-S3-RLCD-4.2) for hardware reference material.

## Build and first upload

1. Install Arduino IDE and **esp32 by Espressif Systems 3.3.11** through Boards Manager. If needed, add `https://espressif.github.io/arduino-esp32/package_esp32_index.json` to Additional Boards Manager URLs.
2. Install these libraries through Library Manager:
   - **U8g2 2.36.19** (2.36.18 is also supported by the sketch).
   - **ArduinoJson 7.4.3**.
3. Open [`ReflectiveDashboard/ReflectiveDashboard.ino`](ReflectiveDashboard/ReflectiveDashboard.ino). Keep the sketch in its same-named folder.
4. Select these settings:

| Arduino setting | Value |
| --- | --- |
| Board | ESP32S3 Dev Module |
| Flash size | 16 MB |
| PSRAM | OPI PSRAM |
| USB CDC On Boot | Enabled |
| USB Mode | Hardware CDC and JTAG |
| Flash mode / frequency | QIO / 80 MHz |
| Partition Scheme | 16M Flash (3MB APP/9.9MB FATFS) |
| CPU frequency | 240 MHz; firmware selects 80 MHz at runtime |

5. Connect the board over USB, select its port, then compile and upload.

For USB recovery, hold **BOOT** while reconnecting USB, release it, then upload.

Optional Arduino CLI build, from the repository root after installing the dependencies:

```sh
arduino-cli compile --fqbn "esp32:esp32:esp32s3:FlashSize=16M,PSRAM=opi,CDCOnBoot=cdc,PartitionScheme=app3M_fat9M_16MB" ReflectiveDashboard
```

## Configure the dashboard

1. Hold **BOOT for 2.5 seconds** to open setup.
2. Join the Wi-Fi network **RLCD-Setup-XXXX**. Its password is the **setup code displayed on the LCD**; there is no shared default password.
3. Open **http://192.168.4.1** in a browser. Log in with username **admin** and the same setup code as the password.
4. Save your home Wi-Fi credentials. After restart, reopen setup to use online address search.
5. Search for an address, select a result, choose **Use this location**, review the nearby stop and save the settings. You can also enter coordinates and choose a stop manually.
6. For departures, enter your own [Golemio API key](https://api.golemio.cz/api-keys).

The portal also provides display inversion, a location label, sensor correction, update intervals, aircraft radius, RSS feed and timezone settings. The Prague POSIX timezone default includes daylight saving time: `CET-1CEST,M3.5.0,M10.5.0/3`. Hourly labels follow the configured clock timezone.

The configuration access point closes after **10 minutes**, except during a bounded active OTA upload. Hold BOOT again whenever you need it. Settings are saved on the device; normal configuration does not require editing Wi-Fi passwords or API keys into the sketch.

## Buttons

| Action | Result |
| --- | --- |
| KEY click | Next page |
| BOOT click | Previous page |
| KEY hold, about 0.9 seconds | Page-specific detail/history, next aircraft or eligible refresh |
| BOOT hold, about 2.5 seconds | Open or close setup |
| PWR | Board's hardware power function |

## Update over Wi-Fi (OTA)

1. Open the new `.ino` in Arduino IDE and use the same board and partition settings as the initial USB upload.
2. Choose **Sketch → Export Compiled Binary**.
3. Open the dashboard's setup portal and log in.
4. Upload **ReflectiveDashboard.ino.bin** through the firmware update form.
5. Keep power stable and wait for the upload and restart to finish.

Upload the application binary only, **not** a merged image, bootloader or partition binary. Compatible updates retain saved settings. Changing the partition layout requires a USB upload.

## Behavior and limitations

- Aircraft origins/destinations are **callsign route database matches**, not confirmed live itineraries. Missing callsigns, missing database entries and partial matches can leave either endpoint unavailable.
- Battery percentage is an approximate voltage-based estimate. Charging affects the reading, and the stock wiring does not expose a reliable charging-status signal to the ESP32.
- The reflective LCD has no backlight. White/black inversion is a visual preference, not an OLED-style power-saving feature.
- MCU sleep remains disabled for display stability. Wi-Fi stops between request batches, and only changed display strips are transferred. Battery runtime has not been benchmarked for this firmware.
- Sensor readings can be influenced by heat from the board and charging.
- Cached data survives network failures but not a restart. `STALE` marks old data or a failed refresh; expired hourly forecasts are not presented as upcoming weather.
- HTTPS requests verify certificates and require a valid clock. The RTC stores UTC and NTP periodically synchronizes time.
- The embedded PID stop catalogue is a snapshot from **2026-09-14**. Stop names and network coverage can change.

## Source layout

All firmware code is in one `.ino`, including the display adapter, dashboard, setup portal and graphics. Appendix A contains the offline stop catalogue; Appendix B contains the original display-adapter license. The large file size is largely explained by that embedded catalogue.

The v1.4.0 build was checked with ESP32 core 3.3.11, U8g2 2.36.19 and ArduinoJson 7.4.3. The ESP32 build used approximately **1.81 MB**, within the 3 MB application partition. Host-side checks covered forecast parsing and rendered layouts; these do not replace testing on the device.

## Credits and licensing

- Display adapter adapted from [Waveshare's ESP32-S3-RLCD-4.2 example](https://github.com/waveshareteam/ESP32-S3-RLCD-4.2). Its original **Apache-2.0 license is retained in Appendix B of the sketch**, along with the modification notice near the top.
- Forecasts: [Open-Meteo](https://open-meteo.com/), attribution under CC BY 4.0. Check the provider's current service terms for your intended use.
- Aircraft positions: [ADSB.lol](https://www.adsb.lol/), ODbL.
- Route matches: [adsbdb](https://www.adsbdb.com/); route-data attribution to David Taylor and Jim Mason.
- Address lookup: [Photon](https://photon.komoot.io/) / komoot, using OpenStreetMap contributor data (ODbL).
- Public transport: [PID](https://pid.cz/) / [Golemio](https://api.golemio.cz/).
- Default news feed: [BBC News](https://www.bbc.com/news).
- Libraries: [U8g2](https://github.com/olikraus/u8g2) and [ArduinoJson](https://arduinojson.org/), under their respective licenses.

No additional project-wide license has been selected. The included Waveshare notice applies to that third-party code; it should not be read as assigning a license to every part of this project.
