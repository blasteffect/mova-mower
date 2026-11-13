# Migration guide: MOVA Mower 0.1.x to 2.1.0

## Before upgrading

1. Create a normal Home Assistant backup.
2. Export or copy any automations that call `dreame_mower` actions.
3. Record the current mower entity ID and the map names/IDs shown by the old
   integration.
4. Do not edit `.storage` manually. The integration performs the supported
   entity-registry migration through Home Assistant's API.

The migration was developed on branch `codex/migrate-v2.0.3` and published as
release `v2.1.0`.

## Install the migrated fork

1. In the Home Assistant Marketplace custom repositories (or HACS on an older
   installation), use `https://github.com/blasteffect/mova-mower` as an
   integration repository.
2. Install version 2.1.0 and restart Home Assistant.
3. Open **Settings → Devices & services → Dreame Mower**. Existing entries with
   username, password, country, account type, device ID and MAC address remain
   compatible with the v2 coordinator.
4. If Home Assistant requests authentication, select the same cloud as before:
   MOVAhome for MOVA devices or Dreamehome for Dreame devices.

The main lawn-mower registry entry is migrated in place from the old
`<mac>_dreame_mower` unique ID to `<mac>_lawn_mower`; its user-facing entity ID
is retained.

## Automation compatibility

The three map actions used by the fork remain available with their old names:

| Existing action | Status in 2.1.0 | Preferred v2 alternative |
|---|---|---|
| `dreame_mower.mower_select_map` | Compatibility alias retained | `dreame_mower.select_map` |
| `dreame_mower.mower_run_map` | Retained | Keep it when a single action must switch and start |
| `dreame_mower.mower_run_map_by_name` | Retained | Keep it, or use the ID action for an immutable target |

Example by stable map ID:

```yaml
action: dreame_mower.mower_run_map
target:
  entity_id: lawn_mower.mova_lidar_1200
data:
  map_id: 2
  start_delay: 5
```

Example by displayed name:

```yaml
action: dreame_mower.mower_run_map_by_name
target:
  entity_id: lawn_mower.mova_lidar_1200
data:
  map_name: Front Garden
  start_delay: 5
```

The action refuses to switch while any mowing session is active. A name must
identify exactly one map. After upgrading, prefer map IDs for long-lived
automations because users can rename maps.

Other official action names are shorter in v2. Common replacements include:

| Old 0.1.x action | v2 action |
|---|---|
| `mower_clean_segment` | `start_zone_mowing` |
| `mower_clean_spot` | `start_spot_mowing` |
| `mower_rename_map` | `rename_map` |
| `mower_backup_map` | `backup_map` |
| `mower_restore_map` | `restore_map` |
| `mower_delete_map` | `delete_map` |

Review any automation that uses an old action not listed as retained.

## Entity and dashboard changes

The v2 integration replaces the vacuum-derived camera, buttons, numbers and
map selectors with a live Dreame Mower card and mower-specific entities. The
primary mower entity is preserved, but retired accessory entity IDs may no
longer exist. Replace their dashboard cards or automation references with:

- the `maps`, `current_map_id`, `zones` and `contours` attributes of the mower;
- the included `custom:dreame-mower-card`;
- the v2 sensors, switches, selects and time entities;
- the actions listed in Developer tools → Actions.

## Validation with the MOVA 1200

After the first restart, validate in this order:

1. The existing mower entity loads and stays available.
2. The account sensor shows the intended MOVAhome account and country.
3. The mower entity's `maps` attribute contains every saved map; note their IDs.
4. `dreame_mower.select_map` changes `current_map_id` without losing another map.
5. `mower_run_map` starts the requested map, then test `mower_run_map_by_name`.
6. Rename a map in the app and wait for the map refresh; confirm the new name is
   visible and the other maps remain.
7. While the robot repositions, confirm the status sensor shows
   **Repositionnement** instead of **Unknown (19)**.
8. Confirm that a task in progress blocks a map switch with a clear error.
9. Check zones, no-go areas, obstacles, trail and history. Missing camera or 3D
   data is not a failure when the MOVA 1200 does not provide it.
10. Review the Home Assistant log for unhandled `siid:piid` properties or cloud
    timeouts.

Account login, real MQTT traffic and movement commands are deliberately not run
by the automated test suite.

## Rollback

If physical validation finds a blocking problem:

1. Stop new mower automations.
2. Reinstall the previous fork release `v0.1.2`.
3. Restart Home Assistant.
4. Restore the pre-upgrade Home Assistant backup if registry or dashboard state
   also needs to be reverted.

Do not copy an old `device.py` or `map.py` into the 2.x tree; their protocols and
data structures are incompatible.
