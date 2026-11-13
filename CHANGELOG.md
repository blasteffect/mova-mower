# Changelog

## 2.1.0 - 2026-11-13

### Base migration

- Rebased the fork's implementation on the stable upstream
  `antondaubert/dreame-mower` 2.0.3 architecture.
- Added the interactive map card, modern multi-map model, active-map
  confirmation, map editing, history, schedules, preferences and map
  backup/restore supplied by 2.0.3.
- Updated repository, issue tracker, code owner and Marketplace/HACS metadata
  for `blasteffect/mova-mower`.

### MOVA compatibility

- Map raw device status `19` to `repositioning` / **Repositionnement**.
- Treat the repositioning lawn-mower activity as paused rather than docked.
- Cover opaque `did=-115170541` MQTT updates without enum decoding or update
  failures.
- Preserve the existing mower entity ID through a registry unique-ID migration.

### Automation compatibility

- Preserve `mower_select_map` as an alias of the verified v2 `select_map` action.
- Preserve `mower_run_map` and `mower_run_map_by_name`, including the optional
  `start_delay` field used by 0.1.x automations.
- Validate map IDs and names, refuse ambiguous names and active-task switches,
  and target the chosen map explicitly when starting.

### Tests and documentation

- Add status, raw MQTT, entity-registry migration and map-action regression tests.
- Add `MIGRATION_ANALYSIS.md` and `MIGRATION_GUIDE.md`.
- Document physical MOVA 1200 checks that cannot be performed without the robot.

## 0.1.2 - 2026-04-14

- Prevent unknown property identifiers from interrupting legacy state updates.

## 0.1.1 - 2026-04-14

- Correct fork Marketplace/HACS metadata.

## 0.1.0 - 2026-04-14

- Add map selection and map-based mowing actions to the original 0.0.5-alpha integration.
