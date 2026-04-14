# Dreame / MOVA Mower for Home Assistant

Control your Dreame and MOVA robotic mowers from Home Assistant, including multi-map scheduling support for robots such as the MOVA Lidar 1200.

This integration adds:

* Lawn mower entity support
* Start / Pause / Return to dock
* Multi-map selection
* Start mowing directly on a selected map
* Home Assistant automations for map-based schedules

## Features

* Supports Dreamehome / MOVAhome accounts
* Exposes the mower as a Home Assistant `lawn_mower` entity
* Select the active map from Home Assistant
* New services to automate mowing by map:

  * `dreame_mower.mower_run_map`
  * `dreame_mower.mower_run_map_by_name`
* Compatible with multiple saved maps and zones on MOVA Lidar 1200

Example use case:

* Monday / Wednesday / Friday → mow "Front Garden"
* Tuesday / Thursday / Saturday → mow "Back Garden"

without having to manually switch the active map in the MOVA app.

## Installation

### HACS

1. Open HACS
2. Add this repository as a custom repository
3. Category: `Integration`
4. Install "Dreame / MOVA Mower"
5. Restart Home Assistant

### Manual installation

Copy:

```text
custom_components/dreame_mower/
```

into:

```text
/config/custom_components/dreame_mower/
```

Then restart Home Assistant.

## Configuration

After installation:

1. Go to Settings → Devices & Services
2. Add integration: `Dreame / MOVA Mower`
3. Login using your MOVAhome / Dreamehome account

The mower should appear as a `lawn_mower` entity.

## Available Services

### Select a map only

```yaml
service: dreame_mower.mower_select_map
target:
  entity_id: lawn_mower.mova_lidar_1200
data:
  map_id: 2
```

### Select a map and immediately start mowing

```yaml
service: dreame_mower.mower_run_map
target:
  entity_id: lawn_mower.mova_lidar_1200
data:
  map_id: 2
  start_delay: 5
```

### Select a map by name and start mowing

```yaml
service: dreame_mower.mower_run_map_by_name
target:
  entity_id: lawn_mower.mova_lidar_1200
data:
  map_name: Front Garden
  start_delay: 5
```

## Example Automations

### Front garden

```yaml
alias: Mow front garden
trigger:
  - platform: time
    at: "09:00:00"
condition:
  - condition: time
    weekday:
      - mon
      - wed
      - fri
action:
  - service: dreame_mower.mower_run_map_by_name
    target:
      entity_id: lawn_mower.mova_lidar_1200
    data:
      map_name: Front Garden
      start_delay: 5
```

### Back garden

```yaml
alias: Mow back garden
trigger:
  - platform: time
    at: "09:00:00"
condition:
  - condition: time
    weekday:
      - tue
      - thu
      - sat
action:
  - service: dreame_mower.mower_run_map_by_name
    target:
      entity_id: lawn_mower.mova_lidar_1200
    data:
      map_name: Back Garden
      start_delay: 5
```

## Notes

This integration is currently focused on reliable mower control and multi-map scheduling.

Map rendering, live mower position and session tracking are planned for a future release inspired by the `dreame-mower-better` project.

---

## Changelog

### v0.1.0

* Initial Home Assistant support for Dreame / MOVA mowers
* Added lawn mower entity with:

  * Start mowing
  * Pause
  * Return to dock
* Added support for selecting the active saved map

### v0.1.1

* Added `mower_run_map` service
* Added `mower_run_map_by_name` service
* Allows selecting a map then automatically starting mowing after a configurable delay
* Added safeguards to prevent map switching while the mower is already running

### v0.1.2

* Improved compatibility with MOVA Lidar 1200 multi-map setups
* Better handling of map names and unknown map errors
* Added Home Assistant examples for scheduled mowing on multiple maps

### Planned

* Live map view
* Robot position overlay
* Current zone information
* Session progress and mowing history
* Zone-based mowing from Home Assistant
