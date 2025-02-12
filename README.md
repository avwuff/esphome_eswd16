# ESPHome Config for Etekcity ESWD16

Using the Etekcity ESWD16 dimmer with ESPHome

![dimmer2.jpg](photos%2Fdimmer2.jpg)

⚠ May require this change to ESPHome:
https://github.com/esphome/esphome/pull/8144

## Installation

1. Copy the 5 files in the `/src/` directory into the `/esphome/include` directory on your HA installation. 
If you don't have a copy of Studio Code Server, I highly recommend it.  If the `include` directory does not exist, create it.

2. Create a new default configuration for your device.  Skip all the offers and pick ESP8266.  Don't upload, just exit.

3. Now replace the entire yaml with the following:
```yaml
substitutions:
  device_name: etekcity-dimmer-14
  friendly_name: Etekcity Dimmer 14 - Dining Room
  encryption_key: !secret api_key
  ota: !secret ota_pass
  
packages:
  base: !include include/etekcity-dimmer-base.yaml
  main: !include include/etekcity-dimmer-main.yaml
```

You may have to define `ota_pass` and `api_key` in your secrets in ESPHome, use whatever value you like. 

See below for knowing which packages to include.

4. Pop open the cover of your dimmer with a spudger -- no screwdriver needed.

Using a 3.3v FTDI cable and ESPHome Web, you can flash basic firmware onto the ESP like this:
![flashing.jpg](photos%2Fflashing.jpg)
*Do NOT power the dimmer from 120VAC when doing this.*

5. Power on your dimmer.  It will be ready to use!

### Configurations

There are several options for how the dimmer can be used.
- You always need to include `etekcity-dimmer-base.yaml`
- If you want a normal dimmer, also include `etekcity-dimmer-main.yaml`
- If you want a dimmer that only turns on and off (for controlling non-dimmable loads), include `etekcity-dimmer-onoff.yaml` instead of `main`.
- If you want a dimmer that doesn't control the load at all, for example at the secondary location of a 3-way setup, include `etekcity-dimmer-slave.yaml`.

### 3-way and more via MQTT
This system supports multiple switches controlling the same light via MQTT.
First, add 3 new secrets to your ESPHome config:
- `mqtt_host`: The IP of your MQTT server
- `mqtt_user`: Your MQTT user name
- `mqtt_pass`: Your MQTT password

Next, make your configuration look like this for the dimmer that actually has the load attached to it:
```yaml
substitutions:
  device_name: etekcity-dimmer-3
  friendly_name: Etekcity Dimmer 03 - Workshop
  encryption_key: !secret api_key
  ota: !secret ota_pass
  group_with: "workshop"
  
packages:
  base: !include include/etekcity-dimmer-base.yaml
  slave: !include include/etekcity-dimmer-main.yaml
  mqtt: !include include/etekcity-dimmer-mqtt.yaml
```

Note the addition of the `mqtt` include, as well as the `group_with` field. All dimmers that should control the same light should have the same `group_with`.

On any dimmer that does not control the load, use the following configuration:
```yaml
substitutions:
  device_name: etekcity-dimmer-4
  friendly_name: Etekcity Dimmer 04 - Workshop Slave 1
  encryption_key: !secret api_key
  ota: !secret ota_pass
  group_with: "workshop"

packages:
  base: !include include/etekcity-dimmer-base.yaml
  slave: !include include/etekcity-dimmer-slave.yaml
  mqtt: !include include/etekcity-dimmer-mqtt.yaml
```

![wall.jpg](photos%2Fwall.jpg)

## Usage

This firmware will provide the following entities for each dimmer:

![devices.jpg](photos%2Fdevices.jpg)

- **Main Light**: The actual load that the dimmer controls
- **Ring Light**: The RGB ring around the dimmer displays this color.
- **Ring Light when On**: If the dimmer is switched on, the ring will be this color. Otherwise, it will be the **Ring Light** color.  If the lamp is dimmed, it will be a relative transition between the two colors.
- **Ring Light Override**: When on, the value of this light takes over and replaces the other Ring colors.  This makes it easy to display animated notifications just by turning this light on, and turning it off again when the notification has cleared. 