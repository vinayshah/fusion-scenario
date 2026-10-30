# Company Billing Rates

Syncs billing rates from **Workfront Planning** into **Workfront's native billing rate system**, keyed by company + job role.

![Scenario canvas](./fusion_blueprint_screenshot.png)

## What It Does

Reads "company rates" records from a Planning workspace and upserts them into Workfront's RATE object. For each Planning record that has a rate set, it either:

- **Creates** a new billing rate entry if the company+role combination has no existing rates
- **Updates** existing rates, appending the new rate with a start date and automatically closing out the previous rate's end date

## Blueprint File

`company_rate_blueprint-302403.json`

## API Reference

The scenario uses two Workfront API calls against the `RATE` endpoint (PUT method).

### setRatesForRole — create a new billing rate

Used when no existing rate exists for the company + role combination.

See [`sample_setRatesForRole_request.json`](./sample_setRatesForRole_request.json) for a full example.

| Field | Description |
|-------|-------------|
| `attachableID` | Workfront Company ID |
| `attachableObjCode` | Always `CMPY` |
| `roleID` | Workfront Job Role ID |
| `rates[].type` | Always `BILLING` |
| `rates[].value` | Billing rate amount |
| `rates[].startDate` | Rate effective start date (`null` = no start) |
| `rates[].endDate` | Rate effective end date (`null` = open-ended) |
| `currencyCode` | Currency (e.g. `USD`) |

### editRatesForRole — update an existing billing rate

Used when a rate already exists. Closes out the previous rate's `endDate` and appends the new rate with a `startDate`.

See [`sample_editRatesForRole_request.json`](./sample_editRatesForRole_request.json) for a full example.

| Field | Description |
|-------|-------------|
| `rates[].ID` | ID of the existing `RATE` record to update |
| `rates[].endDate` | Set to `startDate - 1 day` of the new rate to close it out |
| `rates[].containOverlap` | Always `false` |
| New rate entry | Same shape as `setRatesForRole` but includes `startDate` |

## Data Model (Planning → Workfront)

| Planning Field | Workfront Field |
|---------------|-----------------|
| Company ID | `RATE.companyID` (via `attachableID`) |
| Role ID (from Job Roles) | `RATE.roleID` |
| rate | `RATE.value` |
| Start Date | `RATE.startDate` |

The screenshot below shows what the source records look like in the Workfront Planning record type view:

![Planning records](./csv_record_screenshot.png)

## Import Instructions

1. Open Workfront Fusion
2. Create a new scenario
3. Click `...` → **Import Blueprint**
4. Select `company_rate_blueprint-302403.json`
5. Reconnect the Workfront and Workfront Planning modules using your own connections after import

## Notes

- The scenario queries Planning with a filter for non-empty `rate` — records without a rate value are skipped
- When updating existing rates, the previous rate's `endDate` is set to `startDate - 1 day` to avoid gaps or overlaps
- Errors on individual RATE writes are ignored so the scenario continues processing remaining records
