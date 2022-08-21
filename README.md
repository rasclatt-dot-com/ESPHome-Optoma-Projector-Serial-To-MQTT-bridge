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

