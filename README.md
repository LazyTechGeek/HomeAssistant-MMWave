# Home Assistant + MMWave

In this video, we take a deep dive into MMWave with Home Assistant for room-level presence detection.

## Watch the video here:
▶️ [Home Assistant + mmWave (Room-Level Presence & Automations) – Everything Presence Lite](https://youtu.be/bNUvHnP21iY)

## 📖 Written Articles
- **[Offical Every Presence Lite Documentation](https://docs.everythingsmart.io/s/products/doc/getting-started-gLYWWonJV2)**

## 🔗 Links to Everything Presence Lite
- **[Everything Presense Lite](ENTER-URL)**


## 🔗 Links to Everything Presence Lite Mount
- **[Everything Presense Lite 3D print mount](https://github.com/LazyTechGeek/HomeAssistant-MMWave/blob/main/epl_mount.3mf)**

## Automations used in this video

**Below are complete working ESPHome Bluetooth proxy configurations for each board.**

### Cinema Mode – Toggle
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
      - conditions:
          - condition: state
            entity_id: input_boolean.cinema_mode
            state: "off"
        sequence:
          - target:
              entity_id: input_boolean.cinema_mode
            action: input_boolean.turn_on
          - target:
              entity_id:
                - cover.YOUR_LOUNGE_CURTAINS
                - cover.YOUR_DINING_CURTAINS
            action: cover.close_cover
            data: {}
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
          - action: number.set_value
            target:
              entity_id:
                - >-
                  number.YOUR_ZONE_1_OCCUPANCY_DELAY
                - number.YOUR_ZONE_2_OCCUPANCY_DELAY
            data:
              value: "3"
          - action: notify.notify
            metadata: {}
            data:
              message: Cinema Mode (enabled)
          - action: notify.alexa_media_YOUR_ALEXA_DEVICE
            metadata: {}
            data:
              message: Cinema mode enabled
      - conditions:
          - condition: state
            entity_id: input_boolean.cinema_mode
            state: "on"
        sequence:
          - target:
              entity_id: input_boolean.cinema_mode
            action: input_boolean.turn_off
          - action: switch.turn_on
            metadata: {}
            target:
              entity_id:
                - switch.YOUR_KITCHEN_SWITCH
                - switch.YOUR_LOUNGE_SWITCH
            data: {}
          - action: number.set_value
            target:
              entity_id:
                - >-
                  number.YOUR_ZONE_1_OCCUPANCY_DELAY
                - number.YOUR_ZONE_2_OCCUPANCY_DELAY
            data:
              value: "15"
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

### Cinema Mode - Toggle (with comments)
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
  - trigger: state
    entity_id:
      - binary_sensor.YOUR_ZONE_1_OCCUPANCY
    to:
      - "on"
    id: "1"
  - trigger: state
    entity_id:
      - binary_sensor.YOUR_ZONE_2_OCCUPANCY
    to:
      - "on"
    id: "2"
conditions:
  - condition: state
    entity_id: input_boolean.cinema_mode
    state:
      - "on"
actions:
  - choose:
      - conditions:
          - condition: trigger
            id:
              - "1"
        sequence:
          - action: switch.turn_off
            metadata: {}
            target:
              entity_id: switch.YOUR_KITCHEN_SWITCH
            data: {}
      - conditions:
          - condition: trigger
            id:
              - "2"
        sequence:
          - action: switch.turn_on
            metadata: {}
            target:
              entity_id: switch.YOUR_KITCHEN_SWITCH
            data: {}
mode: single
```

### Cinema Mode – Presence Lighting (with comments)
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

### Co2 Sensor Automation (draft)
```yaml
alias: CO2 Air Quality Alert
description: >-
Sets the Everything Presence Lite RGB LED green when CO₂ is below 600 ppm and red when above 600 ppm.
Sends a notification to open the window if CO₂ stays high for 5 minutes while the room is occupied between 08:00 and 22:00.

triggers:
  - trigger: numeric_state
    entity_id:
      - sensor.everything_presence_lite_1_co2
    below: 600
    id: "1"
    for:
      hours: 0
      minutes: 5
      seconds: 0
  - trigger: numeric_state
    entity_id:
      - sensor.everything_presence_lite_1_co2
    above: 600
    id: "2"
    for:
      hours: 0
      minutes: 5
      seconds: 0
conditions: []
actions:
  - choose:
      - conditions:
          - condition: trigger
            id:
              - "1"
        sequence:
          - action: light.turn_on
            metadata: {}
            data:
              rgb_color:
                - 0
                - 255
                - 0
              brightness_pct: 100
            target:
              entity_id: light.everything_presence_lite_1_epl_rgb_led
      - conditions:
          - condition: trigger
            id:
              - "2"
        sequence:
          - action: light.turn_on
            metadata: {}
            data:
              rgb_color:
                - 255
                - 0
                - 0
              brightness_pct: 100
            target:
              entity_id: light.everything_presence_lite_1_epl_rgb_led
          - choose:
              - conditions:
                  - condition: state
                    entity_id: binary_sensor.everything_presence_lite_2_occupancy
                    state:
                      - "on"
                  - condition: time
                    after: "08:00:00"
                    before: "22:00:00"
                sequence:
                  - action: notify.mobile_app_dave_s_s23_mobile
                    metadata: {}
                    data:
                      title: Air Quality
                      message: CO₂ is rising — open the window.
                  - action: notify.alexa_media_dave_s_echo_spot
                    metadata: {}
                    data:
                      message: Dave,  the CO2 levels are rising. Open the window
mode: single
```

```
