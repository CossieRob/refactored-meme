# Whole House Lighting

Lighting for every room, driven by occupancy. It takes into account whether you are home, whether you are asleep (iPhone Sleep focus), lux levels and sun elevation.

| File | What it is |
|---|---|
| `whole_house_lighting.yaml` | Blueprint for one room. Make one automation from it per room. |
| `whole_house_lighting_package.yaml` | Package with the override helper plus an automation for each room (Lounge main lights, Lounge Lamp, Lounge Rear Lamp, Loft, Dining, Hallway, Cupboard, Landing/Stairs, Office, Bedroom, Spare, Junk, Toilet). |

## Behaviour

| Situation | Non-bedroom rooms | Bedroom |
|---|---|---|
| Motion/presence detected and room is dark | On at the brightness for the current sun level | Same |
| Every sensor clear for the timeout | Off | Off |
| Asleep (Sleep focus on) | Lights that are on dim to sleep brightness (5%). Motion turns them on at 5%. | Left alone. Motion never turns them on. |
| Sleep focus ends | Lights that are on go back to normal brightness | If you're in the room, it's dark and the lights are off, they turn on (`wake_lights`) |
| Nobody home | Off, and motion is ignored | Off, and motion is ignored |
| You arrive home, the override is switched off, or Home Assistant restarts | The room is re-checked: lights turn on if it's occupied and dark, or off if it's empty | Same |
| A room's sensors come back online after a restart or reboot | Off once every sensor is back and has been clear for the timeout | Same |
| Sun crosses a level boundary | Lights that are on adjust brightness and colour temperature | Same |
| Override switch on | Automation does nothing | Same |

**Is the room dark?** If the room has a lux sensor with a valid reading, lights turn on only below the lux threshold (30 lx by default). Otherwise it uses sun elevation, below 5° by default. If a room that is already occupied gets dark, the lights turn on without needing new motion.

**Sun levels:** above 5° is day (100%, 4000 K), 5° to -6° is twilight (80%, 3000 K), and below -6° is night (50%, 2200 K).

Lux only decides whether lights turn on. Once they're on, the lights themselves raise the lux reading, so lux is never used to turn them off. That avoids flicker loops.

If any light in a room is already on, motion won't change its brightness, so lights you set by hand aren't overridden.

**Lounge:** the Lounge Lamp follows the presence sensor's Couch zone, and the Lounge Rear Lamp follows its Sofa zone. While either zone is occupied, Lounge Light 1 and 2 stay off. They switch off when someone sits down, and come back on once both zones are clear and the room is still occupied and dark. This uses the blueprint's optional `suppress_sensors` input. The Lounge Reading Lamp isn't automated.

## Install

1. Copy `whole_house_lighting.yaml` to `config/blueprints/automation/cossierob/whole_house_lighting.yaml`.
2. Copy `whole_house_lighting_package.yaml` to `config/packages/`. You need this in `configuration.yaml`:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
3. **Disable the old automations these replace**, or they'll fight over the same lights: "Stairs Lamp On at Sunset" (now Landing & Stairs) and every "Motion Activated … Light(s)" automation, including "Motion Activated Loft Lights" (now Loft).
4. Restart, or reload Automations and Input Booleans.

Alternatively, skip the package and create each room from the blueprint in the UI (Settings → Automations → Create → Use blueprint).

## Updating

- **Blueprint:** the blueprint has a `source_url` pointing at this repo, so go to Settings → Automations & Scenes → Blueprints, open the ⋮ menu on "Whole House Lighting - Room Controller" and choose **Re-import blueprint**. Then reload Automations. Home Assistant doesn't check for updates on its own, so do this whenever the blueprint changes here.
- **Package:** Home Assistant can't import packages, so copy `whole_house_lighting_package.yaml` over your copy in `config/packages/` again, then reload Automations and Input Booleans. It only holds the per-room settings, so it changes less often.
- **Folder name:** the package expects the blueprint at exactly `cossierob/whole_house_lighting.yaml`, and the name is case-sensitive. Importing the blueprint fresh from GitHub saves it under `CossieRob/` instead, so rename that folder to lowercase. Re-import keeps the existing path.

## iPhone Sleep focus

Sleep mode is driven by the Companion app's `binary_sensor.rob_siphone_focus`. That sensor is `on` for *any* focus, which is fine here because Focus is only used for Sleep.

If you later add other focus modes (Work, Driving…), switch to an iOS Shortcuts automation instead:

1. Create an `input_boolean` (e.g. `rob_sleep_focus`).
2. In Shortcuts → Automation, set **Sleep** → *When turning on* to run the Home Assistant action `input_boolean.turn_on` on it. Add a second automation for *When turning off* that runs `input_boolean.turn_off`.
3. Point `sleep_entity` in the package at the input_boolean.

## Tuning per room

Every option in the blueprint can be overridden per room in the package, for example `vacancy_timeout`, `lux_threshold`, `elevation_threshold`, the `*_brightness` and `*_kelvin` values, and `transition`. Set `use_color_temp: false` for rooms that only have dimmable white bulbs.
