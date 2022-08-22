# ESPHome-Optoma-Projector-Serial-To-MQTT-bridge

This project uses an ESP32 to provide a serial to MQTT gateway and associated configuration for Home Assistant control and automation of an Optoma projector.

# Tested and eliminated options

## HDMI/ CEC

A lot of the (cheaper) Optoma Projectors do not have ethernet or wifi (which would have simplified the project!), and trying to use HDMI to start and stop Optoma projectors seems to cause issues ranging from the projector freezing up (bluescreen) or screwing up and only showing images in pink. Both situaations only resolved with a power cycle - which is not ideal and can damage projectors.

HDMI/ CEC would have been convenient, also tried Home Assistant HDMI CEC integration, and some community repos to see if this could be maanaged from HAOS rather than hardware. This had simular and unreliable results and often bluescreened/ pinkscreened the projector too.

For some reason HAOS HDMI integration is not available/ working on RPI4, having tried a number of community repositories this was the only one that worked (https://github.com/samueltardieu/homeassistant-addon-pi-cec) - worth a try first to see if you are luckier with HDMI before building this.

Researching this HDMI issue - there are is a lot of reports these issues are fixed with a firmware update, however Optoma have removed firmware downloads from their website, aand are non-responsive to support requests for firmare, so HDMI/ CEC was eliminaated early in this project.

## IR

Optoma projectors use NEC IR signal setup, and was easy to set up an ESP32 to act as an IR remote, however it was a bit of a clunky solution, and for some straange reason it seems that Optoma projectors dont like to receive IR commands too quickly - this also led to the same bluescreen/ pink screen lockups a few times for no aparent reason.

I gave up on using an IR baased solution due to the unreliability.

## Other options

Optoma projectors have a well documented set of RS232 commands which can control the projector via serial port, but i didn't want to run another cable accross the ceiling to hard wire this, which is why this project uses an ESP32 to allow HAOS to send commands to the projector over wifi.

There is an option to configure the projector to start when the power to the projector is switched on, however this only addresses half of the control - as it is not a sensible option to switch a running projector off from the mains - over time this would cause dammage by not allowing the cooling cycle to finish.

# The setup.

This project assumes you have installed the following;

- Home Assistant (tested on Home Assistant OS version, on a RPi4)
- MQTT broker (this uses the HAAOS MQTT integration.
- ESPHome, used to configure the ESP32, and for Over The Air (OTA) updates to the code.

## Bill of materials

You will need the following components;

- 1 x ESP32 module 
- 1 x Serial (TTS) adaptor

<img
  src="https://github.com/rasclatt-dot-com/ESPHome-Optoma-Projector-Serial-To-MQTT-bridge/blob/bcc0036cc4fa2783c63b56ebc33b174ee40c04e2/assets/Max232%20seriaal%20TTL%20adaptor.png"
  alt="RS232 TTL MAAX232 module"
  title="RS232 TTL MAX232 convertor"
  style="display: inline-block; margin: 0 auto; width: 250px">
  
This project was tested with a few different ESP32 boards  (ESP-WROVER-devkit-C board (too big for final box) and an M5Stack Stamp (tiny but needed a separate power regulator/ buck convertor so not practical). Finally settled on and M%Stack atom - as these are small, cheap and can be powered from USB-C port. You may need to change the GPIO pins in the config for other boards.

A full blown implimentation of RS232 is not required for this to work, the Max232 (TTL) adaptor converts TX and RX (at a sutiable voltage to not fry the ESP!).

## Cabling

Setup is pretty straight forward, the MAX232 board can be powered from the 5V and GND ports of your ESP32, with TX of the module connected to RX on your ESP32 and RX on the serial board to TX on your ESP32 as shown below;


  
# ESPHome (ESP32 configuration)

This documentation assumes you are familiar with using ESPHome to configure and manage ESP devices for Home Assistant.

Once your device is set up for OTA you need to copy the YAML, aqnd edit the following;

- device specific wifi/ and Static IP addressing (if required)
- make sure not to overwrite the ESP api and OTA keys so ESPHome can still manage the device.
- you may need to update the board details (this is based on using M5 Atom so yours may be different.
- You may need to update your GPIO pins (depending on your ESP32) for TX and RX.
- Update your MQTT broker & crednetials

ESPHome UART implimentation is geared towards writing to UART, in order to read and update the status this project uses and additional Uart Read Line Sensor library (uart_read_line_sensor.h included above).

Before the ESPHome will compile and upload this file needs to be copied to the ESPHome directory on your Home Assistant server that contains youe ESP configurations, (default is usually /config/esphome/), make sure that the "include: uart_read_line_sensor.h" line has been copied to your ESPHome configuration.

Once you are happy run the OTA process, once sucessful your device should de automatically detected by MQTT, (see below);

<img
  src="https://github.com/rasclatt-dot-com/ESPHome-Optoma-Projector-Serial-To-MQTT-bridge/blob/80feb1aeed47a8416ac781a16ca254c6eb693d56/assets/MQTT%20view.png"
  alt="Serial adaptor to M5Stack Atom cabling"
  title="Serial adaptor to M5Stack Atom cabling"
  style="display: inline-block; margin: 0 auto; width: 400px">

## RS232 commands

Optoma provide documentation of their RS232/ serial command set - and this can be used to controll pretty much anything the menu and remote controll can do, however i have limited this project to power/ standby, as the projector is plugged into an AV receiver and the rest of the features (i.e. inputs/ settings) dont need to be changed.

IAdding additional sensors/ switces/ commands to the config should be straight forward - heres the Optoma Rs232 guide if you want to go down that path;

https://optoma.de/uploads/RS232/DS309-RS232-en.pdf

Also should be noted that that different Optoma projectors seem to support different serial settings, the documsntation is a bit generic. 
You may have to change the baud rate to fit your projector, i could not get this working until i changed my baud to 9600, but some projectors work much higher.

If in doubt/ commands not working try changing this setting first!

## connecting to the projector

Connection is pretty straight forward, plug the serial cable into the adaptor.

<img
  src="https://github.com/rasclatt-dot-com/ESPHome-Optoma-Projector-Serial-To-MQTT-bridge/blob/e54ede8d1789d1e28bcab44ca5ca86529715b698/assets/hd39hdrrear.jpg"
  alt="Serial adaptor to M5Stack Atom cabling"
  title="Serial adaptor to M5Stack Atom cabling"
  style="display: inline-block; margin: 0 auto; width: 400px">

# Power considerations

My projector is ceiling mounted so i use the USB port on the back of the Optoma prtojector to power this project. There is a setting in the Optoma menu to switch on power to the usb - and i initially assumed this would remain powered in standby mode, however this is not the case.

This means once the projector switches off there is no power for the ESP unless you are able to provide a spearate power supply to keep it on, i hve 2 options to address this issue.

1. Battery backup implimentation
2. Separate smart power plug (for powering on controlls)

## Battery Backup 

Testing out option one 1 changed the ESP32 module to an Adafruit Huzzah ESP32 feather https://learn.adafruit.com/adafruit-huzzah32-esp32-feather as these modules have an integrated power managment setup that allows a LiPo battery to be plugged in. When usb power is available the module runs on usb power source and charges the battery in the background, when the usb power goes off the power managment automatically switches to run from battery backup.

Even with a large capacity battery, there are other considderations to this option, the battery lifespan would be significantly reduced if the ESP32 was left running 100% of the time, LiPo batteries thaat are constantly drained and reharged will start to loose efficiency and need to be replaced often.

It would make sense to impliment Deep Sleep if you go this path, however there are trade offs in terms of responsivness if the device is only online for a brief time every minute of couple of minutes which may not be ideal for some people. When using Deep Sleep I also add an additional MQTT switch to turn off deep sleep cycles - with a pub/sub message thats read on wakeup to disable the deep sleep cycle, otherwise managing the device and performing OTA updates from ESPHome is almost impossiable!

I looked at a setup that used an IR PIR presence sensor to trigger a wake up on pin when motion is detected, however for the sake of simplicity - and avoid falling in to a rabbit hole of deep sleep and time based aautomation logic i chose to go the simpler path with option 2.

## Separate mains power 

Optoma projectors have a setting that allows projectors to automatically power on when the mains supply is switched on. This seemed the easiest way to close the gap on the ESP configuration (which already allows the projector to be powered down by serial.

After setting up this option in the Optoma menu, I added a smart power plug/ switch to the projector power supply, and configured home automation automations to switch the power off and then back on again to trigger the projector to power on, and use the esp32 setup to trigger power off by serial.

I have added some logic to the power on automation to prevent the power being switched (off/on) within a minute of the projector being switched off to prevent dammage to the projector that may happen if you interupt the cooldown cycle.

# Example Home Automation setup

Example dashboard and automation files have been included, these are based on dashboard buttons that switch all entertainment and living room systems on/ off (including AV receiver & selecting inputs, projector, media player and lighting. 

You will need to edit these to suit your setup - but have incuded these as a starting point.


