# Repository architecture

This repository is a collection of Adobe Workfront Fusion scenario exports.
Each scenario is independent: its blueprint JSON contains the executable module
graph, while its README, screenshots, and optional sample requests explain the
integration. The root `README.md` is the scenario catalog and import entry point.

- The company billing rates scenario reads Workfront Planning records with a
  non-empty `rate`, builds a new rate entry, and searches Workfront `RATE` objects
  by company and job role. Routers select `setRatesForRole` for creation or
  `editRatesForRole` for updates, both through custom `PUT RATE` calls.
- The program export scenario counts `PRGM` records, pages through `/PRGM/search`
  using a repeater and `$$FIRST`/`$$LIMIT`, iterates the results, and aggregates
  program and portfolio fields into CSV.

# Editing blueprints

- Preserve Fusion's export structure: top-level `flow` and `metadata`, module
  `id`, `module`, `version`, `parameters`, `mapper`, and UI `metadata`.
  Router branches are nested under `routes[].flow`; write error handlers are
  nested under `onerror`.
- Numeric module IDs are referenced inside `{{...}}` mapping expressions and
  aggregator `parameters.feeder` values. If changing an ID or graph structure,
  update its dependent mappings and feeder references together.
- Mapping strings use Fusion expressions, not JavaScript: retain semicolon
  argument separators, backtick-quoted field names, and array selectors.
  Custom API `mapper.body` values may be JSON-encoded strings containing these
  expressions; preserve both the outer JSON escaping and inner payload.
- Keep designer coordinates, restore labels, expected fields, and interfaces
  consistent with the affected modules so the blueprint remains editable in
  Fusion.
- Connections (`parameters.__IMTCONN__`), Planning workspace/record-type/field
  IDs, and source object IDs refer to the exporting environment. Document the
  required remapping in the scenario's Import Instructions without including
  personal names, account labels, or tenant domains. Connections must be
  reconnected after import; use generic connection and workspace labels.

# Billing rate behavior

- Rates attach to companies using `attachableObjCode: "CMPY"` and are keyed by
  company plus job role, with rate type `BILLING`.
- When updating, preserve existing rates and their existing end dates. An
  open-ended previous rate is closed at the incoming start date minus one day;
  mapped dates use `YYYY-MM-DD`.
- The creation branch uses null start/end dates and `currencyCode: "USD"`.
  Both RATE write modules have `builtin:Ignore` error handlers so processing
  continues after individual write failures.
- Keep the scenario README and `sample_setRatesForRole_request.json` /
  `sample_editRatesForRole_request.json` aligned with changes to the API contract.
  Samples are illustrative payloads, not executable blueprint modules.

# Adding a scenario

Follow `CLAUDE.md` and use `company-billing-rates/README.md` as the documentation
pattern. Create a scenario folder containing:

- An exported JSON named `<name>_blueprint-<id>.json`.
- A README covering behavior, blueprint filename, API/data mappings, import
  instructions, and scenario-specific notes.
- `fusion_blueprint_screenshot.png`, embedded near the top of the README.
- `csv_record_screenshot.png`, showing source records and embedded under the
  data model section.

Add a row linking the folder in the root README's scenarios table.

# Validation

Check an individual edited JSON file with:

```sh
python3 -m json.tool company-billing-rates/company_rate_blueprint-302403.json > /dev/null
```

Use the same command with the path to any changed blueprint or sample request.
For runtime validation, import the blueprint into a new Workfront Fusion
scenario, reconnect the environment-specific connections, select the intended
source records, and run it there. For billing-rate changes, exercise both the
create and update routes and verify the resulting rates and effective dates;
ignored write errors mean scenario completion alone does not prove success.
