# Whole House Lighting

Lighting for every room, driven by occupancy. It takes into account whether you are home, whether you are asleep (iPhone Sleep focus), lux levels and sun elevation.

| File | What it is |
|---|---|
| `whole_house_lighting.yaml` | Blueprint for one room. Make one automation from it per room. |
| `whole_house_lighting_package.yaml` | Package with the helpers plus an automation for each room (Lounge, Dining, Hallway, Landing/Stairs, Office, Bedroom, Spare, Junk, Toilet). |

## Behaviour

| Situation | Non-bedroom rooms | Bedroom |
|---|---|---|
| Motion/presence detected and room is dark | On at the brightness for the current sun level | Same |
| Every sensor clear for the timeout | Off | Off |
| Asleep (Sleep focus on) | Lights that are on dim to sleep brightness (5%). Motion turns them on at 5%. | Left alone. Motion never turns them on. |
| Sleep focus ends | Lights that are on go back to normal brightness | Unchanged |
| Nobody home | Off, and motion is ignored | Off, and motion is ignored |
| Sun crosses a level boundary | Lights that are on adjust brightness and colour temperature | Same |
| Override switch on | Automation does nothing | Same |

**Is the room dark?** If the room has a lux sensor with a valid reading, lights turn on only below the lux threshold (30 lx by default). Otherwise it uses sun elevation, below 5° by default. If a room that is already occupied gets dark, the lights turn on without needing new motion.

**Sun levels:** above 5° is day (100%, 4000 K), 5° to -6° is twilight (80%, 3000 K), and below -6° is night (50%, 2200 K).

Lux only decides whether lights turn on. Once they're on, the lights themselves raise the lux reading, so lux is never used to turn them off. That avoids flicker loops.

If any light in a room is already on, motion won't change its brightness, so lights you set by hand aren't overridden.

## Install

1. Copy `whole_house_lighting.yaml` to `config/blueprints/automation/cossierob/whole_house_lighting.yaml`.
2. Copy `whole_house_lighting_package.yaml` to `config/packages/`. You need this in `configuration.yaml`:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
3. **Check the entity IDs.** They were derived from entity names. Lines marked `TODO` are motion or ESP32 presence sensors whose entities were not visible, so their IDs are guesses. Landing and Stairs had no sensor visible at all.
4. Restart, or reload Automations and Input Booleans.

Alternatively, skip the package and create each room from the blueprint in the UI (Settings → Automations → Create → Use blueprint).

## iPhone Sleep focus → Home Assistant

The Companion app's `binary_sensor.<phone>_focus` is `on` for *any* focus mode, so it can't tell Sleep apart from Work. Use an iOS Shortcuts automation to set `input_boolean.rob_sleep_focus` instead:

1. Open Shortcuts → Automation → **+** → **Sleep** (under Focus) → *When turning on* → *Run immediately*.
2. Add the Home Assistant action **Perform action**: `input_boolean.turn_on` with entity `input_boolean.rob_sleep_focus`.
3. Create a second automation for *When turning off* that runs `input_boolean.turn_off`.

If Sleep is the only focus you ever use, you could point `sleep_entity` at `binary_sensor.<phone>_focus` directly instead.

## Tuning per room

Every option in the blueprint can be overridden per room in the package, for example `vacancy_timeout`, `lux_threshold`, `elevation_threshold`, the `*_brightness` and `*_kelvin` values, and `transition`. Set `use_color_temp: false` for rooms that only have dimmable white bulbs.
