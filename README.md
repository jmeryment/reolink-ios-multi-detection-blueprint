# Reolink iOS Multi-Detection Blueprint

Home Assistant automation blueprint for Reolink camera alerts with:

- Proxy image snapshots for iOS notifications
- Per-camera detection selection (person, vehicle, animal, etc)
- Cooldown control
- Optional "Silence THIS" and "Silence ALL" actions

## Blueprint file

- `blueprints/automation/NS086/reolink_ios_proxy_image_silence_multi_detection.yaml`

## Import into Home Assistant

After pushing this repo to GitHub, import using the raw URL from your default branch.

Example:

`https://raw.githubusercontent.com/<your-user>/<your-repo>/main/blueprints/automation/NS086/reolink_ios_proxy_image_silence_multi_detection.yaml`

In Home Assistant:

1. Go to **Settings -> Automations & Scenes -> Blueprints**
2. Click **Import Blueprint**
3. Paste the raw URL above

## Suggested setup

Create one automation per camera using this blueprint, and for each camera choose:

- The camera entity
- One or more detection sensors for that camera
- Your mobile app notify devices
- Optional helper booleans for silence behavior
