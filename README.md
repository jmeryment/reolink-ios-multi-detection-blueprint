# Reolink iOS Detection Selector Blueprint

Home Assistant automation blueprint for Reolink camera alerts with:

- Proxy image snapshots for iOS notifications
- Per-camera detection selection split into always-on and away-only sensors
- Cooldown control
- Optional "Silence THIS" and "Silence ALL" actions

## Credits

This blueprint is based on the original blueprint by [NS086](https://github.com/NS086):

- Original blueprint path: `NS086/IOSAlpha.yaml`

This repository is a community derivative that extends the original with per-camera multi-detection sensor selection.

Please keep attribution to the original creator when redistributing derivative versions.

## Blueprint file

- `blueprints/automation/jmeryment/reolink_ios_detection_selector.yaml`

## Import into Home Assistant

Import using the raw URL from the default branch:

Example:

`https://raw.githubusercontent.com/jmeryment/reolink-ios-multi-detection-blueprint/main/blueprints/automation/jmeryment/reolink_ios_detection_selector.yaml`

In Home Assistant:

1. Go to **Settings -> Automations & Scenes -> Blueprints**
2. Click **Import Blueprint**
3. Paste the raw URL above

## Suggested setup

Create one automation per camera using this blueprint, and for each camera choose:

- The camera entity
- One or more always-on detection sensors for that camera
- Optional away-only detection sensors for that camera
- Optional presence helper, for example `input_boolean.me_present`, if you want away-only detections
- Either a notify service (recommended for groups, eg `notify.justin_mobile_devices`) or mobile app notify devices
- Optional helper booleans for silence behavior; these can be left blank if you do not want silence actions

## Presence-aware pattern

This blueprint can support a single automation per camera and recipient while still splitting notification behavior:

- Put sensors like driveway vehicle into the always-on list
- Put sensors like person or animal into the away-only list
- Set `presence_boolean` to a helper that is `on` when the person is home, such as `input_boolean.me_present`

Away-only sensors will only notify when that helper is `off`.
