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

```
