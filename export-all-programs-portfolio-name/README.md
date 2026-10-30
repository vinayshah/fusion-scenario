# Export All Programs with Portfolio Names

Reads programs from **Adobe Workfront** and builds a CSV containing each program's ID and name alongside its portfolio's ID and name.

## What It Does

1. Sets the requested page size (`maxLimit`) to `2000`.
2. Counts programs using `GET /PRGM/count`.
3. Repeats `ceil(count / maxLimit)` times, starting at index `0`.
4. Fetches each page using `GET /PRGM/search`, with an offset of `index * maxLimit`.
5. Iterates the returned programs and maps their program and portfolio fields.
6. Aggregates the rows into CSV with a header row, using the repeater as the aggregation source so rows from multiple pages are collected together.

The scenario only reads Workfront data. It ends at the CSV aggregator; it does not save, upload, or email the generated CSV.

## Blueprint File

[`export-programs-portfolio-name.json`](./export-programs-portfolio-name.json)

The exported scenario is named `Integration Tools`.

## API Reference

Both calls use the Workfront connection and the `GET` method.

### /PRGM/count

Returns the program count used to calculate the number of pages. The repeater reads the count from `body.data.count`.

### /PRGM/search

| Query parameter | Value | Purpose |
|-----------------|-------|---------|
| `$$FIRST` | `{{floor(3.i * 1.maxLimit)}}` | Zero-based offset for the current page |
| `$$LIMIT` | `{{1.maxLimit}}` | Requested page size, initially `2000` |
| `fields` | `ID,name,portfolio:name` | Program identifiers and names, plus the related portfolio name |

The iterator reads the returned array from `body.data`.

## Data Model (Workfront to CSV)

| Workfront program field | CSV column |
|-------------------------|------------|
| `ID` | `programID` |
| `name` | `programName` |
| `portfolio.ID` | `portfolioID` |
| `portfolio.name` | `portName` |

The CSV aggregator includes these column names in its first row. The blueprint maps portfolio fields directly without a fallback for missing portfolio data.

## Import Instructions

1. Open Workfront Fusion and create a new scenario.
2. Click `...` > **Import Blueprint**.
3. Select `export-programs-portfolio-name.json`.
4. Reconnect both Workfront API modules using your own Workfront connection.
5. Review `maxLimit` in the first module and verify that the connected Workfront API honors the requested page size. The page offsets assume this size.
6. Run the scenario and inspect the CSV aggregator's output.
7. To deliver a CSV file, append a module that saves, uploads, or emails the generated CSV.

## Notes

- The export includes programs accessible to the connected Workfront account; no additional program or portfolio filters are configured.
- The final CSV module carries the Fusion designer warning: "A transformer should not be the last module in the route." Add an output destination to consume its result.
- Pagination uses a count taken before fetching the pages and does not specify a sort order. Avoid changing the source dataset during a run when checking completeness.
- Validate a multi-page run by comparing the CSV data-row count and unique `programID` values with the initial count. JSON validation alone does not verify API response fields or pagination behavior.
