# Migration analysis: MOVA Mower 0.1.2 to Dreame Mower 2.0.3

## Scope and repositories audited

The audit used the complete Git histories and source trees of:

| Role | Repository / ref | Commit | Finding |
|---|---|---:|---|
| Existing fork | `blasteffect/mova-mower` `main` / `v0.1.2` | `cfde169` | Three commits ahead of the old upstream release |
| Requested comparison | `bhuebschen/dreame-mower` `main` | `33a89dd` | Tagged `v0.0.5-alpha`; it is the direct ancestor of the fork |
| Requested comparison | `bhuebschen/dreame-mower` `dev` | `e100f01` | Twelve commits beyond its `main`, all dated 2025-06-09; never released |
| Official modern source | `antondaubert/dreame-mower` `v2.0.3` | `542e365` | Latest stable tag, dated 2026-11-05 |
| Official current branch | `antondaubert/dreame-mower` `main` | `8fd446d` | One documentation-only merge after `v2.0.3` |

`bhuebschen/main` is not newer than the fork: the fork is exactly three commits
ahead. Its `dev` branch contains camera, emergency-stop, cloud-response and
lens-brush fixes, but no released 2.x architecture. The 2.x map architecture
required by the mission exists in `antondaubert/dreame-mower`.

The 0.1.x and 2.x histories have no Git merge base. A file-by-file merge would
therefore combine two incompatible implementations. The migration branch starts
from the stable `v2.0.3` tag and reapplies the still-useful fork behaviour as
small, tested changes.

## Behaviour unique to the old fork

The semantic (whitespace-independent) fork changes were:

1. `dreame_mower.mower_run_map`: select a saved map by ID, optionally wait, then start mowing.
2. `dreame_mower.mower_run_map_by_name`: resolve a displayed map name and perform the same operation.
3. Fork repository metadata and MOVA-focused installation/automation examples.
4. Defensive logging for unknown property identifiers. The old implementation
   attempted to construct `DreameMowerProperty(did)` while logging; an opaque
   device identifier such as `-115170541` could raise `ValueError` and interrupt
   an update.

The large diffs in `device.py` and `lawn_mower.py` in 0.1.x were mostly line-ending
or formatting changes. They are not independent features to port.

## What 2.0.3 already replaces

The official 2.0.3 implementation replaces the inherited vacuum-oriented map
stack with mower-specific modules and tests:

- Multi-map parsing retains every map and tracks the active map separately.
- `set_current_map()` validates the ID, refuses changes during a task, sends the
  mower's map-switch payload, and reads the map list back until the switch is
  confirmed.
- All-area mowing accepts an explicit `map_id`; zone, edge and spot tasks have
  dedicated payloads.
- Map hashes are polled so maps added, renamed, edited or deleted outside Home
  Assistant are detected without overwriting the other maps.
- The included 2D/3D dashboard card exposes live position, trail, zones,
  contours, no-go areas, obstacles and obstacle photos when the mower supplies
  them.
- Map rename, zone split/merge, no-go and spot areas, maintenance points,
  backup, restore and delete operations are implemented.
- Work history, schedules, mowing preferences and per-zone settings are covered
  by the official test suite.
- Unknown MQTT `did` values are opaque metadata. Properties are dispatched by
  `siid` and `piid`, so `-115170541` is no longer converted to an enum.
- Known but unused properties are explicitly acknowledged, preventing them from
  breaking state updates.

The old monolithic map renderer, generic vacuum properties, camera entity and
manual map-selection internals are obsolete and were not copied into 2.x.

## Changes retained or adapted in this migration

| Old behaviour | 2.1.0 decision |
|---|---|
| `mower_select_map` | Retained as a compatibility alias of the verified 2.x `select_map` action |
| `mower_run_map` | Retained; validates the ID, confirms the map switch, then starts an explicit all-area task on that ID |
| `mower_run_map_by_name` | Retained; resolves names case-insensitively, rejects missing or duplicate names, then uses `mower_run_map` |
| `start_delay` | Still accepted (0–60 seconds, default 5) so existing automations remain valid; 2.x already confirms the switch before the delay |
| Unknown `did=-115170541` | Covered by an end-to-end MQTT regression test; 2.x ignores the opaque `did` for property dispatch |
| Status `Unknown (19)` | Mapped to `repositioning`; the lawn-mower activity becomes `paused` rather than incorrectly appearing docked |
| Existing mower entity ID | Registry unique ID is migrated from `<mac>_dreame_mower` to `<mac>_lawn_mower` in place |
| Fork ownership | Manifest, Marketplace/HACS metadata, links and validation workflow point to `blasteffect/mova-mower` |

## MOVA 1200 and map capability limits

The v2 cloud discovery accepts the `mova.mower.*` family and obtains supported
data from the device. The migration does not force camera, obstacle-photo,
point-cloud or 3D features on the MOVA 1200. The card renders only the geometry
and telemetry that the mower supplies. This avoids claiming that a MOVA 1200
has the LiDAR/camera capabilities of another model.

The repository has no captured cloud session from the user's physical MOVA 1200.
Account discovery, live position, obstacle data and command acceptance therefore
still require the hardware validation checklist in `MIGRATION_GUIDE.md`.

## Home Assistant 2026.11 and Marketplace audit

- The integration was checked against the exact Home Assistant Core `2026.11.1`
  API source. Its lawn-mower activity enum, entity-service registration,
  config-entry forwarding and config-flow APIs are present.
- The 2.x options flow already uses the current automatic `config_entry`
  property; it avoids the setter incompatibility in the old 0.1.x flow.
- `manifest.json` keeps the domain `dreame_mower`, cloud-polling classification,
  required Home Assistant dependencies and the single runtime dependency
  `paho-mqtt>=2.0.0`.
- `hacs.json` declares Home Assistant 2026.4 or newer and remains usable by the
  custom-repository Marketplace flow. No Home Assistant `.storage` file is
  created or modified by this migration.
- The integration folder remains `custom_components/dreame_mower`.

## Conflicts and migration risks

1. The 2.x rewrite intentionally replaces many old vacuum-derived diagnostic
   entities. The primary lawn-mower entity is migrated in place, but automations
   using retired accessory entities must be updated according to the guide.
2. A map name is not guaranteed unique. Name-based mowing now fails safely when
   two maps have the same displayed name; using `map_id` is recommended.
3. The old arbitrary delay could hide a failed switch. The new action first uses
   the official confirmation loop; it never starts on a map after a confirmed
   refusal.
4. Cloud protocol and physical mower behaviour cannot be fully validated without
   the user's account and MOVA 1200. No test sends a real command.
5. `bhuebschen/dev` is useful historical evidence but is unreleased and older
   than the 2.x code, so merging it after 2.0.3 would reintroduce obsolete files.

## Maintenance strategy

Future synchronization should merge or rebase from stable tags of
`antondaubert/dreame-mower`, not copy individual 0.x files. Fork-specific work is
kept in a small surface: compatibility actions, the status-19 mapping, registry
migration, repository metadata, tests and these migration documents.

## Automated validation completed

- `pytest`: **1,314 passed** on Python 3.14 with the repository's pinned Home
  Assistant test environment.
- Focused migration tests: **179 passed**, including status 19, the opaque
  `-115170541` `did`, legacy actions and entity-registry migration.
- `mypy`: **36 source files checked, no issues**.
- Frontend: **91 tests passed** with Node 22.22.2, plus syntax checks for both
  map-card bundles.
- Home Assistant 2026.11.1: all integration platform modules imported against
  the exact PyPI release; required entity-service APIs, status mapping,
  translations and manifest metadata passed a dedicated smoke check.
- Python compilation, JSON parsing, YAML parsing and `git diff --check` passed.

Tests not performed automatically are cloud login/discovery with the user's
account, live MQTT messages from the physical MOVA 1200, real map switching and
motion commands. Those are explicitly covered by the migration guide checklist.
