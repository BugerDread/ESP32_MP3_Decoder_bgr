ESP32 Web Radio forked by BugerDread
====================================
Original project was made using old ESP-IDF so I decided to make it compatible with ESP-IDF 4.0, which is current stable release.

README also needs to be fixed, because:
- ESP-IDF installation procedure changed - presented steps are no longer valid, please DO NOT FOLLOW "Get the SDK" section of original README, it will NOT WORK WITH ESP-IDF 4.0
- this is fork of https://github.com/esa-n/ESP32_MP3_Decoder which is fork of https://github.com/MrBuddyCasino/ESP32_MP3_Decoder, the Esa-n's version adds some features not mentioned in readme at all such as LCD

Therefore I decided to keep original README at the bottom and to write new information from the top.

## Required Software

Get the ESP-IDF v4.0: https://docs.espressif.com/projects/esp-idf/en/v4.0/get-started/index.html#installation-step-by-st

If you get error during installation about "--no-site-packages", you need to remove this option from tools/idf_tools.py and rerun install.sh. See this bugfix: https://github.com/espressif/esp-idf/commit/8cb3cc515477e777144acb3b58f5156c4ad69124

Clone this repository:

    git clone https://github.com/BugerDread/ESP32_MP3_Decoder_bgr.git
    cd ESP32_MP3_Decoder_bgr/
    git submodule init && git submodule update

## Configuration, building, flashing, controls

Follow the procedures described in original README (see bellow).

## Connecting the I2S codec

I did not tested the external I2S codec support, because I have no such chip currently. If you are able to test this fork using I2S codec please let me know if it works (or not).

## Bluetooth Speaker Mode

I did not tested the bluetooth speaker mode yet.

## Related Hardware & pictures

Tested using ESP32 DevKit / ESP32-WROOM from Aliexpress using PDM / internal-DAC output.

[Few pictures from testing](./pics)

You can see few resistors and capacitors around DAC output, these are the low-pass RC filter I experimented with. Im currently using 4x 1.5k resistors and 4x 6.8nF caps plus 2x 3.9k to sum the stereo signal into mono.

# Original README
=================
This is a simple web radio streamer. It connects to a web radio station via wifi, decodes the stream and plays the sound via I2S codec or
to a speaker directly connected to the DAC pins.

Features:

* Decodes MP3 and AAC (Beta)
* Bluetooth Speaker mode

## Origin

This project is based on Sprite\_TM's awesome MP3 web radio project for the ESP8266: https://github.com/espressif/ESP8266_MP3_DECODER

## Required Software

Get the SDK:

    git clone https://github.com/espressif/esp-idf.git
    cd esp-idf
    git submodule update --init

Set the IDF_PATH environment variable, and point it to this directory.

    export IDF_PATH=/path/to/esp-idf

Download the toolchain from: https://github.com/espressif/esp-idf#setting-up-esp-idf
You will need version 5.2.0.
Add /path/to/xtensa-esp32-elf/bin to your PATH:

    export PATH=/path/to/xtensa-esp32-elf/bin:$PATH

Clone this repository:

    git clone https://github.com/MrBuddyCasino/ESP32_MP3_Decoder.git
    cd ESP32_MP3_Decoder/
    git submodule init && git submodule update

## Configuration

The serial port and wifi credentials are configured using make.
Type `make menuconfig` and 

* configure your serial port in `Serial flasher config` submenu
* select `Web Radio / Bluetooth Speaker` submenu
  * configure wifi credentials
  * select audio output mode
  * activate / disable bt speaker mode
* 'save', then exit

You can edit the list of radio stations in the `/main/playlist.pls` playlist file.

## Building
If this is the first time, initialize the submodules:
`git submodule init && git submodule update`.
Then, just type `make`.

## Flashing
Connect your serial cable and run 'make flash'. To see serial console output run 'make monitor'.

## Controls
You can advance to the next track in the playlist using the "Boot" button that is present on most development boards (GPIO0).

## UI
You can connect a <a href="https://www.adafruit.com/product/1312">NeoPixel</a> LED to pin 32. Its currently not doing much except blinking while wifi is connecting.

## Bluetooth Speaker Mode

Enable `Classic Bluetooth` in `Component config > Bluetooth > Bluedroid Bluetooth stack enabled > Classic Bluetooth`, enable `A2DP > A2DP ROLE config (SINK)`, then enable `Bluetooth Speaker Mode` via `make menuconfig`. After flashing, you should see a Bluetooth device called "ESP_SPEAKER". If you don't like that name, you can change it via menuconfig.

## Connecting the I2S codec

If you don't know about the I2S standard, it is a special protocol for transferring digital audio data between chips, similar to I2C. There are many I2S chips you can choose from, the most important differences are:

1. Amplification: some chips only decode the audio to a low analog level, so you need a separate amp, but some also have a built-in amplifier. Most of these 2-in-1 chips are made for smartphones so their energy output is in the range of 2-4W, but some other ones made for domestic audio appliances can go a lot higher.
2. MCLK: this is a separate clock signal that sometimes needs to be a precise number in the MHz range that depends on the current sample rate, sometimes can be a single constant value ("asynchronous") independent of the current sample rate, and sometimes is not required at all. The ESP32 does not output a MCLK signal, so a chip that does not require MCLK is most convenient. If you already have an asynchronous one lying around (e.g. ES9023), you will need a quartz oscillator, usually in the range of 20-50MHz.

I tested several I2S codecs, and was happiest with the MAX98357A, because it does not require MCLCK and also amplifies the audio to speaker levels. It also seemed to be more immune to signal integrity issues, which do occur on breadboards. There is a convenient breakout board from Adafruit: https://www.adafruit.com/product/3006. Be aware that it is mono only, for stereo you'll want this one: https://www.adafruit.com/product/3346.
However, any I2S codec should work.

Generic wiring:

```
ESP pin   - I2S signal
----------------------
GPIO25/DAC1   - LRCK
GPIO26/DAC2   - BCLK
GPIO22        - DATA
```

If you're using the MAX98357A, connect GND to ground and Vin to +5V (or +3.3V if +5V is unavailable). SD can remain unconnected, and GAIN too unless you want to make it louder or lower. I also recommend using a potentiometer for volume regulation.

## Running without the I2S DAC

If you don't have an I2S codec on hand, there are two options:
- Built-In DAC (low quality)
- PDM (high quality)

Run `make menuconfig` and choose one of them in the `Audio Output Mode` menu, then re-flash.
You can now connect a speaker to ground and the pins 25 and 26 for the left and right channels.

## Known Issues

* Some AAC streams may not be playable

## More Information

There is a thread over at the ESP32 forum:
https://esp32.com/viewtopic.php?f=17&t=1026

## Related Hardware

If you're looking for a purpose-built board, check out the "ESP32 Audio Developing Board" from Microwavemont:
https://www.tindie.com/products/microwavemont/esp32-audio-developing-board-esp32-adb/

## Breadboard Example

I used the Watterott ESP-WROOM-32-Breakout, which is pin-compatible to the Espressif Core Board (DevKitC).
Please note that in this picture, the JTAG header is connected too, but you can safely ignore that.

<img src="doc/breadboard_wiring.jpg" width="50%" height="50%">
