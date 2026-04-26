# Reolink iOS Detection Selector Blueprint

Home Assistant blueprint for Reolink notifications to iOS devices with:

- Per-camera detection selection
- Split behavior for always-on vs away-only detections
- Optional global notification gating helper
- Optional Silence THIS / Silence ALL actions
- Snapshot image attachments that work with iOS notifications

This README is written so someone can set this up from scratch without guesswork.

## Credits and lineage

This project is a derivative of NS086's iOS Reolink blueprint work:

- NS086 repo: https://github.com/NS086/HomeAssistantBlueprints
- Original iOS blueprint path: `IOSAlpha`

This derivative keeps the iOS notification reliability concepts and extends them with per-camera multi-sensor selection and presence-aware split logic.

## Blueprint file in this repo

- `blueprints/automation/jmeryment/reolink_ios_detection_selector.yaml`

## Import into Home Assistant

Use this raw URL:

`https://raw.githubusercontent.com/jmeryment/reolink-ios-multi-detection-blueprint/main/blueprints/automation/jmeryment/reolink_ios_detection_selector.yaml`

In Home Assistant:

1. Go to **Settings -> Automations & Scenes -> Blueprints**
2. Select **Import Blueprint**
3. Paste the raw URL

## Requirements

These requirements combine this repo's behavior with the original iOS guidance:

- Home Assistant Companion App installed on each target iPhone/iPad
- Reolink camera entities and detection binary sensors in Home Assistant
- A reachable external Home Assistant URL (HTTPS strongly recommended)
- A public-facing local media path for snapshots at `/local/reolink_alerts/...`
- Optional helper booleans:
	- presence helper (example `input_boolean.me_present`)
	- global notification helper per recipient (example `input_boolean.cameranotificationsjustin`)
	- silence helpers if you want actionable snooze-like behavior

## Critical iOS image setup (do not skip)

This blueprint writes snapshots to:

- Filesystem: `/config/www/reolink_alerts/<filename_stem>.jpg`
- iOS URL: `<base_url>/local/reolink_alerts/<filename_stem>.jpg?ts=<cache_bust>`

If this directory is not present or your URL is not reachable from the phone, iOS notifications will arrive without images.

### 1. Create the media directory

Create this directory in Home Assistant:

- `/config/www/reolink_alerts`

Any of these methods work:

- Terminal & SSH add-on
	- `mkdir -p /config/www/reolink_alerts`
- Samba/File Editor add-on
	- create folder `www/reolink_alerts` under `/config`

### 2. Verify external URL reachability

In each automation instance, set `base_url` to your externally reachable HA URL, for example:

- `https://haexternal.example.com`
- or your Nabu Casa remote URL

From the iPhone (mobile data and Wi-Fi), confirm you can open:

- `<base_url>/local/reolink_alerts/`

If you get auth prompts, that is fine. If unreachable, fix networking/URL before troubleshooting the blueprint.

### 3. Verify image path quickly

After creating one automation, trigger a detection and then open:

- `<base_url>/local/reolink_alerts/<filename_stem>.jpg`

If the image loads in Safari, iOS notification attachments should work.

## Input reference (what each field does)

### Core camera/detection inputs

- `camera_name`: Human-readable camera name used in notifications
- `camera`: Reolink camera entity
- `detection_sensors`: Always notify from these sensors
- `away_only_detection_sensors`: Only notify when `presence_boolean` is `off`
- `presence_boolean`: Input boolean that is `on` when home

### Notification target inputs

- `notify_service` (recommended): Example `notify.justin_mobile_devices`
- `notify_devices`: Fallback if no notify service is set

If both are provided, the blueprint uses `notify_service`.

### iOS/snapshot inputs

- `base_url`: External HA URL used to build image links
- `filename_stem`: Snapshot filename basis per camera
- `cooldown_seconds`: Minimum seconds between alerts for this automation

### Optional action/filter inputs

- `enable_open_url_button`: Show Open action in notification
- `open_url_path`: Path opened by the Open action
- `global_notifications_enabled`: Optional global on/off helper gate
- `silence_this_boolean`: Optional helper toggled by Silence THIS
- `silence_all_boolean`: Optional helper toggled by Silence ALL

## Recommended automation design

Create one automation per camera per recipient.

For each recipient/camera automation:

1. Set camera and sensors for that camera only
2. Put high-priority detections in `detection_sensors` (always-on)
3. Put lower-priority detections in `away_only_detection_sensors`
4. Set `presence_boolean` if using away-only behavior
5. Set recipient global helper in `global_notifications_enabled`
6. Set `notify_service` to recipient group service
7. Use a unique `filename_stem` per automation

## Example patterns

### Example: Justin driveway (mixed behavior)

- Always-on: vehicle, animal
- Away-only: person
- Global helper: `input_boolean.cameranotificationsjustin`
- Notify service: `notify.justin_mobile_devices`

### Example: Candice garage (away-only)

- Always-on: none
- Away-only: person, vehicle, animal
- Global helper: `input_boolean.cameranotificationscandice`
- Notify service: `notify.candice_mobile_devices`

## Copy-ready example values

Use these as known-good values when creating one automation per camera per recipient.

Assumptions:

- NVR-backed entities are the non-`_2` entities
- `presence_boolean` is `input_boolean.me_present`
- `base_url` is `https://haexternal.meryment.com`
- Camera dashboard uses `open_url_path` with query style: `/dashboard-cameras?entity_id=<camera_entity>`

### Justin examples

| Camera | camera | detection_sensors (always-on) | away_only_detection_sensors | notify_service | global_notifications_enabled | filename_stem | open_url_path |
|---|---|---|---|---|---|---|---|
| Driveway | `camera.driveway_fluent` | `binary_sensor.driveway_vehicle`, `binary_sensor.driveway_animal` | `binary_sensor.driveway_person` | `notify.justin_mobile_devices` | `input_boolean.cameranotificationsjustin` | `driveway_justin` | `/dashboard-cameras?entity_id=camera.driveway_fluent` |
| Garage | `camera.garage_fluent` | _(empty)_ | `binary_sensor.garage_person`, `binary_sensor.garage_vehicle`, `binary_sensor.garage_animal` | `notify.justin_mobile_devices` | `input_boolean.cameranotificationsjustin` | `garage_justin` | `/dashboard-cameras?entity_id=camera.garage_fluent` |
| Deck | `camera.deck_fluent` | _(empty)_ | `binary_sensor.deck_person`, `binary_sensor.deck_vehicle`, `binary_sensor.deck_animal` | `notify.justin_mobile_devices` | `input_boolean.cameranotificationsjustin` | `deck_justin` | `/dashboard-cameras?entity_id=camera.deck_fluent` |
| Patio | `camera.patio_fluent` | _(empty)_ | `binary_sensor.patio_person`, `binary_sensor.patio_vehicle`, `binary_sensor.patio_animal` | `notify.justin_mobile_devices` | `input_boolean.cameranotificationsjustin` | `patio_justin` | `/dashboard-cameras?entity_id=camera.patio_fluent` |
| Backstairs | `camera.backstairs_fluent` | _(empty)_ | `binary_sensor.backstairs_person`, `binary_sensor.backstairs_vehicle`, `binary_sensor.backstairs_animal` | `notify.justin_mobile_devices` | `input_boolean.cameranotificationsjustin` | `backstairs_justin` | `/dashboard-cameras?entity_id=camera.backstairs_fluent` |
| Doorbell | `camera.doorbell_fluent` | `binary_sensor.doorbell_person`, `binary_sensor.doorbell_visitor` | `binary_sensor.doorbell_vehicle`, `binary_sensor.doorbell_pet` | `notify.justin_mobile_devices` | `input_boolean.cameranotificationsjustin` | `doorbell_justin` | `/dashboard-cameras?entity_id=camera.doorbell_fluent` |

### Candice examples

| Camera | camera | detection_sensors (always-on) | away_only_detection_sensors | notify_service | global_notifications_enabled | filename_stem | open_url_path |
|---|---|---|---|---|---|---|---|
| Driveway | `camera.driveway_fluent` | `binary_sensor.driveway_vehicle`, `binary_sensor.driveway_animal` | `binary_sensor.driveway_person` | `notify.candice_mobile_devices` | `input_boolean.cameranotificationscandice` | `driveway_candice` | `/dashboard-cameras?entity_id=camera.driveway_fluent` |
| Garage | `camera.garage_fluent` | _(empty)_ | `binary_sensor.garage_person`, `binary_sensor.garage_vehicle`, `binary_sensor.garage_animal` | `notify.candice_mobile_devices` | `input_boolean.cameranotificationscandice` | `garage_candice` | `/dashboard-cameras?entity_id=camera.garage_fluent` |
| Deck | `camera.deck_fluent` | _(empty)_ | `binary_sensor.deck_person`, `binary_sensor.deck_vehicle`, `binary_sensor.deck_animal` | `notify.candice_mobile_devices` | `input_boolean.cameranotificationscandice` | `deck_candice` | `/dashboard-cameras?entity_id=camera.deck_fluent` |
| Patio | `camera.patio_fluent` | _(empty)_ | `binary_sensor.patio_person`, `binary_sensor.patio_vehicle`, `binary_sensor.patio_animal` | `notify.candice_mobile_devices` | `input_boolean.cameranotificationscandice` | `patio_candice` | `/dashboard-cameras?entity_id=camera.patio_fluent` |
| Backstairs | `camera.backstairs_fluent` | _(empty)_ | `binary_sensor.backstairs_person`, `binary_sensor.backstairs_vehicle`, `binary_sensor.backstairs_animal` | `notify.candice_mobile_devices` | `input_boolean.cameranotificationscandice` | `backstairs_candice` | `/dashboard-cameras?entity_id=camera.backstairs_fluent` |
| Doorbell | `camera.doorbell_fluent` | `binary_sensor.doorbell_person`, `binary_sensor.doorbell_visitor` | `binary_sensor.doorbell_vehicle`, `binary_sensor.doorbell_pet` | `notify.candice_mobile_devices` | `input_boolean.cameranotificationscandice` | `doorbell_candice` | `/dashboard-cameras?entity_id=camera.doorbell_fluent` |

### Common values for all rows

- `notify_devices`: leave empty when `notify_service` is set
- `cooldown_seconds`: `60`
- `enable_open_url_button`: `true`
- `silence_this_boolean`: leave empty unless you use a per-camera silence helper
- `silence_all_boolean`: leave empty unless you use a global silence helper

## Troubleshooting

### No notification received

- Confirm triggering sensor actually changed to `on`
- Confirm `global_notifications_enabled` is `on` (if configured)
- Confirm silence helpers are `off` (if configured)
- Confirm cooldown is not suppressing back-to-back events

### Notification arrives but no image

- Confirm directory exists: `/config/www/reolink_alerts`
- Confirm `base_url` is valid and reachable from phone
- Confirm file exists after a trigger: `/config/www/reolink_alerts/<filename_stem>.jpg`
- Open `<base_url>/local/reolink_alerts/<filename_stem>.jpg` manually

### Away-only rules not behaving as expected

- `presence_boolean` must be `on` when home and `off` when away
- Away-only detections only notify when that helper is `off`
- Always-on detections ignore presence state

## Security notes

- Prefer HTTPS for `base_url`
- Avoid publishing private camera names, URLs, or helper IDs in public examples
- Use least-privilege exposure for remote Home Assistant access
