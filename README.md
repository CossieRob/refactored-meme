# refactored-meme

Home Assistant blueprints and packages, one project per folder.

## Projects

| Project | What it does |
|---|---|
| [WholeHouseLighting](WholeHouseLighting/) | Occupancy-driven lighting for every room, using motion and presence sensors, lux, sun elevation, sleep (iPhone Focus) and away detection. |

Each project folder has its own README with install and update steps.

## Conventions

- **One folder per project**, holding its blueprint, any package and a README.
- **Blueprint file names are unique across the repo.** Every blueprint imported from this repo lands in the same `config/blueprints/automation/cossierob/` folder in Home Assistant, so two files with the same name would overwrite each other.
- **Each blueprint sets `source_url`** to its own file on `main`. Home Assistant then offers **Re-import blueprint** (Settings → Automations & Scenes → Blueprints → ⋮) to pull the latest version.
- **The folder name is lowercase.** Packages refer to blueprints as `cossierob/<file>.yaml`, and the path is case-sensitive. Importing a blueprint from GitHub creates `CossieRob/`, so rename it to `cossierob/`.
- **Packages are copied by hand** into `config/packages/` (Home Assistant has no import for them), with `homeassistant: packages: !include_dir_named packages` in `configuration.yaml`.
