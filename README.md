# Home Assistant + mmWave - Part 1

In this video, we take a deep dive into MMWave with Home Assistant for room-level presence detection.

## Watch the video here:
▶️ [Home Assistant + mmWave (Room-Level Presence & Automations) – Everything Presence Lite](https://youtu.be/bNUvHnP21iY)

## 📖 Documentation
- **[Official Every Presence Lite Documentation](https://docs.everythingsmart.io/s/products/doc/getting-started-gLYWWonJV2)**

## 🔗 Links to Everything Presence Lite
- **[Everything Presense Lite](https://shop.everythingsmart.io/products/everything-presence-lite?srsltid=AfmBOoq66PhPZbI7PAFx0u_JaenQnDzx6FaZshkF2W-IjATUrEgS1ap9)**


## 🔗 3D Printed Mount
- **[Everything Presence Lite 3D print mount](https://github.com/LazyTechGeek/HomeAssistant-MMWave/blob/main/epl_mount.3mf)**

This mount allows you to reposition the sensor using Velcro, without losing calibration or zone alignment.

## Automations used in this video

### Cinema Mode - Toggle
```yaml
alias: Cinema Mode – Toggle
description: >
  Toggles Cinema Mode using a vibration sensor.
  When enabled, it closes curtains, turns off lights and switches, and reduces occupancy delays.
  When disabled, it turns selected switches back on and resets occupancy delays to normal.
triggers:
  - entity_id: binary_sensor.YOUR_VIBRATION_SENSOR
    to: "on"
    trigger: state
actions:
  - choose:

######################
# ENABLE CINEMA MODE #
######################
# Triggered when Cinema Mode is currently OFF

      - conditions:
          - condition: state
            entity_id: input_boolean.cinema_mode
            state: "off"
        sequence:

          # Turn Cinema Mode ON
          - target:
              entity_id: input_boolean.cinema_mode
            action: input_boolean.turn_on

          # Close curtains for a darker environment
          - target:
              entity_id:
                - cover.YOUR_LOUNGE_CURTAINS
                - cover.YOUR_DINING_CURTAINS
            action: cover.close_cover
            data: {}

          # Turn off lights and switches
          - target:
              entity_id:
                - light.YOUR_LIGHT
            action: light.turn_off
            data: {}

          - action: switch.turn_off
            metadata: {}
            target:
              entity_id:
                - switch.YOUR_KITCHEN_SWITCH
                - switch.YOUR_OTHER_SWITCH
            data: {}
         
          # Reduce occupancy delay for faster light response
          - action: number.set_value
            target:
              entity_id:
                - >-
                  number.YOUR_ZONE_1_OCCUPANCY_DELAY
                - number.YOUR_ZONE_2_OCCUPANCY_DELAY
            data:
              value: "3"

          # Notifications (optional)
          - action: notify.notify
            metadata: {}
            data:
              message: Cinema Mode (enabled)

          - action: notify.alexa_media_YOUR_ALEXA_DEVICE
            metadata: {}
            data:
              message: Cinema mode enabled

#######################
# DISABLE CINEMA MODE #
#######################
     
      - conditions:
          - condition: state
            entity_id: input_boolean.cinema_mode
            state: "on"
        sequence:

          # Turn Cinema Mode OFF
          - target:
              entity_id: input_boolean.cinema_mode
            action: input_boolean.turn_off

          # Restore normal lighting
          - action: switch.turn_on
            metadata: {}
            target:
              entity_id:
                - switch.YOUR_KITCHEN_SWITCH
                - switch.YOUR_LOUNGE_SWITCH
            data: {}

          # Restore normal occupancy delay
          - action: number.set_value
            target:
              entity_id:
                - >-
                  number.YOUR_ZONE_1_OCCUPANCY_DELAY
                - number.YOUR_ZONE_2_OCCUPANCY_DELAY
            data:
              value: "15"

          # Notifications (optional)
          - action: notify.notify
            metadata: {}
            data:
              message: Cinema Mode (disabled)
     
          - action: notify.alexa_media_YOUR_ALEXA_DEVICE
            metadata: {}
            data:
              message: Cinema mode disabled
mode: single
```

### Cinema Mode – Presence Lighting
```yaml
alias: Cinema Mode – Presence Lighting
description: >-
  Controls lighting based on presence zones while Cinema Mode is active.
  Entering Zone 1 turns the light off, and entering Zone 2 turns it on.
triggers:

#################
# ZONE TRIGGERS #
#################

  - trigger: state
    entity_id:
      - binary_sensor.YOUR_ZONE_1_OCCUPANCY
    to: "on"
    id: "1"
  - trigger: state
    entity_id:
      - binary_sensor.YOUR_ZONE_2_OCCUPANCY
    to: "on"
    id: "2"

#######################################
# ONLY RUN WHEN CINEMA MODE IS ACTIVE #
#######################################

conditions:
  - condition: state
    entity_id: input_boolean.cinema_mode
    state: "on"


actions:
  - choose:

###########################
# IF ZONE 1 IS TRIGGERED  #
# (Watching Area)         #
###########################

      - conditions:
          - condition: trigger
            id: "1"
        sequence:
          - action: switch.turn_off
            metadata: {}
            target:
              entity_id: switch.YOUR_KITCHEN_SWITCH
            data: {}

##########################
# IF ZONE 2 IS TRIGGERED #
# (Movement Area)        #
##########################   
      
      - conditions:
          - condition: trigger
            id: "2"
        sequence:
          - action: switch.turn_on
            metadata: {}
            target:
              entity_id: switch.YOUR_KITCHEN_SWITCH
            data: {}
mode: single
```

# Home Assistant + Everything Presence Lite (CO₂ Air Quality Alerts) - Part 2

Turn your Everything Presence Lite into a smart air quality monitor. In this video, we add the optional CO₂ module, hook up a custom RGB LED, and build a Home Assistant automation that visually shows air quality and alerts you when it’s time to open a window.

▶️ [Watch the video here](https://youtu.be/Uzqvpj5Va3w)  

### ⚙️ configuration files used in this video

## 🔌 How to Wire the RGB LED

<img src="https://github.com/LazyTechGeek/HomeAssistant-MMWave/blob/main/CO2-Air-Quality/epl-rgb-led-wiring.png" width="150">

<table>
  <tr>
    <td>
      <img src="https://github.com/LazyTechGeek/HomeAssistant-MMWave/blob/main/CO2-Air-Quality/epl-rgb-led-placement.png" width="300">
    </td>
    <td>
      <img src="https://github.com/LazyTechGeek/HomeAssistant-MMWave/blob/main/CO2-Air-Quality/epl-rgb-led-installed.png" width="300">
    </td>
  </tr>
</table>

## ⚙️ ESPHome Configuration – RGB LED (GPIO19)

⚠️ **Important – ESPHome Packages**

If you are using ESPHome packages, make sure this section matches your setup.

If the CO₂ package is missing or overwritten, your CO₂ sensor will stop working.

🎥 [Watch explanation at 10:16](https://youtu.be/Uzqvpj5Va3w?t=616)

```yaml
packages:
  EverythingSmartTechnology.Everything Presence Lite: github://everythingsmarthome/everything-presence-lite/everything-presence-lite-ha-co2.yaml@main
```

### ⚙️ Add RGB LED to ESPHome

Add the following to your ESPHome configuration:
```yaml
light:
  - platform: esp32_rmt_led_strip
    chipset: ws2812
    rgb_order: GRB
    pin: GPIO19
    num_leds: 1
    name: "EPL RGB LED"
```

### Full ESPHome Example (with RGB LED)
```yaml
substitutions:
  name: everything-presence-lite
  friendly_name: Everything Presence Lite
packages:
  EverythingSmartTechnology.Everything Presence Lite: github://everythingsmarthome/everything-presence-lite/everything-presence-lite-ha-co2.yaml@main # Ensure this matches your setup
esphome:
  name: ${name}
  name_add_mac_suffix: false
  friendly_name: ${friendly_name}
api:
  encryption:
    key: YOUR_ENCRYPTION_KEY

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

light:
  - platform: esp32_rmt_led_strip
    chipset: ws2812
    rgb_order: GRB
    pin: GPIO19
    num_leds: 1
    name: "EPL RGB LED"
```

### CO2 Air Quality - LED & Alerts
```yaml
alias: CO2 Air Quality - LED & Alerts
description: Updates LED colour from CO₂ levels and sends alerts if high and occupied.
mode: single

#################
#   TRIGGERS    #
#################

triggers:
  # Trigger 1: CO₂ has been GOOD (below 700) for 10 minutes
  - trigger: numeric_state
    entity_id:
      - sensor.YOUR_CO2_SENSOR # CHANGE THIS
    for:
      hours: 0
      minutes: 10
      seconds: 0
    below: 700
    id: "1"

  # Trigger 2: CO₂ has been HIGH (above 700) for 10 minutes
  - trigger: numeric_state
    entity_id:
      - sensor.YOUR_CO2_SENSOR # CHANGE THIS
    for:
      hours: 0
      minutes: 10
      seconds: 0
    above: 700
    id: "2"

conditions: []

#################
#    ACTIONS    #
#################

actions:
  - choose:

      ############################
      # GOOD AIR QUALITY (GREEN) #
      ############################
      - conditions:
          - condition: trigger
            id:
              - "1" # Trigger 1 = CO₂ below 700
        sequence:
          - action: light.turn_on
            metadata: {}
            target:
              entity_id: light.YOUR_CUSTOM_RGB_LED  # CHANGE THIS
            data:
              rgb_color:
                - 0
                - 255
                - 0
              brightness_pct: 100

      ###########################
      # BAD AIR QUALITY (RED)   #
      ###########################
      - conditions:
          - condition: trigger
            id:
              - "2" # Trigger 2 = CO₂ above 700
        sequence:
          - action: light.turn_on
            metadata: {}
            target:
              entity_id: light.YOUR_CUSTOM_RGB_LED  # CHANGE THIS
            data:
              rgb_color:
                - 255
                - 0
                - 0
              brightness_pct: 100

          ########################################
          # OPTIONAL ALERTS (ONLY IF OCCUPIED)   #
          ########################################

          # Only send alerts if presence is detected
          # and the time is between 8am and 10pm

          - choose:
              - conditions:
                  - condition: state
                    entity_id: binary_sensor.YOUR_PRESENCE_SENSOR_OCCUPANCY # CHANGE THIS
                    state:
                      - "on"
                  - condition: time
                    after: "08:00:00"
                    before: "22:00:00"

                sequence:

                  # Alexa announcement (optional)
                  - action: notify.alexa_media_YOUR_ALEXA_DEVICE # CHANGE THIS
                    metadata: {}
                    data:
                      message: >-
                        Dave, the air quality is very poor. Open the window to
                        let some fresh air in.

                  # Mobile notification (optional)
                  - action: notify.notify
                    metadata: {}
                    data:
                      title: Air Quality
                      message: CO₂ is high — open a window and get some fresh air
```
