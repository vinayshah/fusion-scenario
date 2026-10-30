# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Adding a New Blueprint

Create a subfolder containing:
- The exported blueprint `.json` file
- A `README.md` describing the scenario (follow `company-billing-rates/README.md` as the pattern)
- A `fusion_blueprint_screenshot.png` — screenshot of the Fusion scenario canvas
- A `csv_record_screenshot.png` — screenshot of the source record view (e.g. the Planning record type table)

Then add a row to the scenarios table in the root `README.md`.

## Conventions

- Blueprint JSON filenames use the format `<name>_blueprint-<id>.json`
- Screenshots are embedded in the folder README — canvas screenshot at the top, source data screenshot under the data model section
- Connection IDs in blueprints are environment-specific; document required reconnection under Import Instructions without including personal names, account labels, or tenant domains
