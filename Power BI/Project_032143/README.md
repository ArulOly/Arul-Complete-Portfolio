# Power BI: Audit — Recovery for Help Desk (Project 032143)

## Project overview

A help desk training queue provides the setting for this audit exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The help desk needs a refreshable Power BI table for audit with recovery.

## Project objectives

- Flag records reviewed before the cutoff date.
- Select the fallback when the primary record cannot be evaluated.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Power BI Desktop, Power Query M, JSON, and a table visual.

## System architecture

`Embedded JSON → Power Query parsing and validation → workflow calculation → report table`

## Features

- Flag records reviewed before the cutoff date.
- Select the fallback when the primary record cannot be evaluated.

## Prerequisites

Use Power BI Desktop on Windows. Use a text editor and a local practice directory.

## Complete Power BI source code

```powerquery
// Power Query reference implementation for project 032143.
let
    Metric = "follow_up",
    InputJson = "{""primary"":{""asset"":""DESK-14"",""reviewed_on"":""2026-05-03"",""cutoff"":""2026-06-01"",""owner"":""Lab Operations""},""fallback"":{""asset"":""DESK-14"",""reviewed_on"":""2026-05-03"",""cutoff"":""2026-06-01"",""owner"":""Lab Operations""}}",
    Input = Json.Document(InputJson),
    Nonblank = (value as any, name as text) as text =>
        if not Value.Is(value, type text) or Text.Trim(value) = ""
        then error (name & " must be non-empty text") else Text.Trim(value),
    AsRecord = (value as any) as record =>
        if Value.Is(value, type record) then value else error "Expected a record",
    Day = (value as any) as date =>
        let
            input = Nonblank(value, "date"),
            parsed = Date.FromText(input, [Format = "yyyy-MM-dd", Culture = "en-US"])
        in if Date.ToText(parsed, [Format = "yyyy-MM-dd", Culture = "en-US"]) <> input
           then error "Expected a date in YYYY-MM-DD format" else parsed,
    Evaluate = (r as record) as record =>
        let
            reviewed = Day(r[reviewed_on]),
            cutoff = Day(r[cutoff]),
            owner = Nonblank(r[owner], "owner"),
            result = [owner = owner, reviewed_on = Date.ToText(reviewed, [Format = "yyyy-MM-dd"]),
                      follow_up = reviewed < cutoff]
        in result,
    TryResult = (value as any) as record =>
        try let
            recordValue = AsRecord(value),
            asset = Nonblank(recordValue[asset], "asset"),
            calculated = Evaluate(Record.AddField(Record.RemoveFields(recordValue, {"asset"}), "asset", asset))
        in Text.FromBinary(Json.FromValue(calculated), TextEncoding.Utf8),
    MakeRow = (value as any, attempt as record, view as text, prior as nullable text, position as number) as record =>
        let
            candidate = try Nonblank(AsRecord(value)[asset], "asset") otherwise "Unknown case",
            result = if attempt[HasError] then null else Json.Document(attempt[Value]),
            measured = if attempt[HasError] then null else
                Text.FromBinary(Json.FromValue(Record.Field(result, Metric)), TextEncoding.Utf8)
        in [Row = position, Case = candidate, View = view, Metric = Metric, Value = measured,
            Status = if attempt[HasError] then "Rejected" else "OK",
            Diagnostic = if attempt[HasError] then attempt[Error][Message] else prior,
            ResultJson = if attempt[HasError] then null else attempt[Value]],
    Primary = TryResult(Input[primary]),
    Selected = if Primary[HasError] then Input[fallback] else Input[primary],
    Attempt = if Primary[HasError] then TryResult(Selected) else Primary,
    Diagnostic = if Primary[HasError] then Primary[Error][Message] else null,
    Rows = {MakeRow(Selected, Attempt, if Primary[HasError] then "Fallback" else "Primary", Diagnostic, 1)},
    Output = Table.TransformColumnTypes(Table.FromRecords(Rows),
        {{"Row", Int64.Type}, {"Case", type text}, {"View", type text}, {"Metric", type text},
         {"Value", type text}, {"Status", type text}, {"Diagnostic", type text}, {"ResultJson", type text}})
in
    Output
```

## Implementation explanation

`Evaluate` applies the audit rules. `TryResult` serializes evaluated records inside a try expression so field errors can be surfaced. A typed output table exposes the metric and diagnostics for the recovery view.

## Sample input data

The source embeds this synthetic JSON fixture so the query can run without a separate file.

```json
{
  "primary": {
    "asset": "DESK-14",
    "reviewed_on": "2026-05-03",
    "cutoff": "2026-06-01",
    "owner": "Lab Operations"
  },
  "fallback": {
    "asset": "DESK-14",
    "reviewed_on": "2026-05-03",
    "cutoff": "2026-06-01",
    "owner": "Lab Operations"
  }
}
```

## Execution instructions

In Power BI Desktop, select **Home → Transform data**. Create a **Blank Query** and open **Advanced Editor**. Replace the editor contents with the M code above and select **Done**. Choose **Close & Apply** to load the output table.

Add a **Table** with **Case**, **View**, **Value**, and **Diagnostic**. Replace the primary data with an incomplete record to observe the fallback view after refresh.

The source includes the JSON sample in `InputJson`. Edit that text literal to try another fixture, preserving the doubled quotes required by M. Refresh the query to update the table.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "follow_up": true
}
```

## Testing and validation

Microsoft's Power Query parser accepted the embedded M code. An independent Python reference check returned the expected field shown below. Power BI Desktop calculation and rendering remain unverified.

```json
{
  "follow_up": true
}
```

## Error handling

The query turns evaluation failures into rejected rows with a diagnostic. Recovery uses the supplied fallback if its primary fails.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The query works with small embedded synthetic fixtures and loads a flat table for local reporting.

## Learning outcomes

Practice audit using Power BI and check a repeatable recovery workflow in the help desk scenario.

## Conclusion

The project records an observable audit result for training case `DESK-14`.

[Back to the Power BI project index](../README.md)
