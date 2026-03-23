# Home Assistant + MMWave

In this video, we take a deep dive into MMWave with Home Assistant for room-level presence detection.

## Watch the video here:
▶️ [Home Assistant + MMWave (Room-Level Presence)](YOUTUBE-LINK)

## 📖 Written Articles
- **[Download the BLE settings slides used in this video](https://github.com/LazyTechGeek/HomeAssistant-Bermuda/blob/main/Bermuda_BLE_Settings-Video-Slides.pdf)**

## 🔗 Links to Everything Presence Lite
- **[Everything Presense Lite](ENTER-URL)**

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

### Co2 Sensor Automation (draft)
```yaml
alias: Air Quality
description: ""
triggers:
  - trigger: numeric_state
    entity_id:
      - sensor.apollo_air_1_4bad34_co2
    below: 600
    id: "1"
    for:
      hours: 0
      minutes: 5
      seconds: 0
  - trigger: numeric_state
    entity_id:
      - sensor.apollo_air_1_4bad34_co2
    above: 600
    id: "2"
    for:
      hours: 0
      minutes: 5
      seconds: 0
  - trigger: time
    at: "08:00:00"
    id: "20"
  - trigger: time
    at: "22:00:00"
    id: "21"
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
          - action: notify.mobile_app_dave_s_s23_mobile
            metadata: {}
            data:
              title: Air Quality
              message: CO₂ is rising — turn the fan on.
          - action: notify.alexa_media_dave_s_echo_spot
            metadata: {}
            data:
              message: Dave,  the CO2 levels are rising. Turn on the fan
          - action: light.turn_on
            metadata: {}
            data:
              rgb_color:
                - 255
                - 255
                - 0
            target:
              entity_id:
                - light.apollo_air_1_4bad34_rgb_light
                - light.wled_topshelf_4
                - light.wled_bottom_shelf_5
      - conditions:
          - condition: trigger
            id:
              - "2"
          - condition:
              - condition: time
                after: "08:00:00"
                before: "22:00:00"
        sequence:
          - action: notify.mobile_app_dave_s_s23_mobile
            metadata: {}
            data:
              title: Air Quality
              message: CO₂ is high — open a window and get some fresh air
          - action: notify.alexa_media_dave_s_echo_spot
            metadata: {}
            data:
              message: >-
                Dave, the air quality is very poor. Open the window to let style
                fresh air in. 
          - action: light.turn_on
            metadata: {}
            data:
              rgb_color:
                - 255
                - 0
                - 0
            target:
              entity_id:
                - light.apollo_air_1_4bad34_rgb_light
                - light.wled_topshelf_4
                - light.wled_bottom_shelf_5
      - conditions:
          - condition: trigger
            id:
              - "4"
          - condition: state
            entity_id: binary_sensor.dehumidifier_state
            state: "off"
        sequence:
          - choose:
              - conditions:
                  - condition: time
                    after: "07:00:00"
                    before: "23:00:00"
                sequence:
                  - action: notify.mobile_app_dave_s_s23_mobile
                    metadata: {}
                    data:
                      title: Air Quality
                      message: >-
                        Humidity is high — turning on the dehumidifier (day
                        mode)
                  - action: button.press
                    metadata: {}
                    data: {}
                    target:
                      entity_id: button.dehumidifier_power
                  - delay:
                      hours: 0
                      minutes: 0
                      seconds: 2
                      milliseconds: 0
                  - device_id: edd72502a149323f04fb5c083d23c791
                    domain: select
                    entity_id: 599fce9671e9219d2876edbf204499cf
                    type: select_option
                    option: Default
              - conditions:
                  - condition: time
                    after: "23:00:00"
                    before: "07:00:00"
                sequence:
                  - action: notify.mobile_app_dave_s_s23_mobile
                    metadata: {}
                    data:
                      title: Air Quality
                      message: >-
                        Humidity is high — turning on the dehumidifier (night
                        mode)
                  - action: button.press
                    metadata: {}
                    data: {}
                    target:
                      entity_id: button.dehumidifier_power
                  - delay:
                      hours: 0
                      minutes: 0
                      seconds: 2
                      milliseconds: 0
                  - device_id: edd72502a149323f04fb5c083d23c791
                    domain: select
                    entity_id: 599fce9671e9219d2876edbf204499cf
                    type: select_option
                    option: Night
mode: single
```

```
