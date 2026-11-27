# Power BI: Log Analysis — Recovery for Branch Office Simulator (Project 032173)

## Project overview

A simulated branch office provides the setting for this log analysis exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The branch office simulator needs a refreshable Power BI table for log analysis with recovery.

## Project objectives

- Count fixed-format log levels and select the most frequent error.
- Select the fallback when the primary record cannot be evaluated.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Power BI Desktop, Power Query M, JSON, and a table visual.

## System architecture

`Embedded JSON → Power Query parsing and validation → workflow calculation → report table`

## Features

- Count fixed-format log levels and select the most frequent error.
- Select the fallback when the primary record cannot be evaluated.

## Prerequisites

Use Power BI Desktop on Windows. Use a text editor and a local practice directory.

## Complete Power BI source code

```powerquery
// Power Query reference implementation for project 032173.
let
    Metric = "top_error",
    InputJson = "{""primary"":{""asset"":""BRANCH-08"",""events"":[""INFO | Started"",""ERROR | Timeout"",""ERROR | Timeout"",""WARN | Retrying""]},""fallback"":{""asset"":""BRANCH-08"",""events"":[""INFO | Started"",""ERROR | Timeout"",""ERROR | Timeout"",""WARN | Retrying""]}}",
    Input = Json.Document(InputJson),
    Nonblank = (value as any, name as text) as text =>
        if not Value.Is(value, type text) or Text.Trim(value) = ""
        then error (name & " must be non-empty text") else Text.Trim(value),
    AsRecord = (value as any) as record =>
        if Value.Is(value, type record) then value else error "Expected a record",
    RequiredList = (value as any, name as text) as list =>
        if not Value.Is(value, type list) or List.IsEmpty(value)
        then error (name & " requires a non-empty list") else value,
    Evaluate = (r as record) as record =>
        let
            events = RequiredList(r[events], "events"),
            parsed = List.Transform(events, each
                let
                    line = Nonblank(_, "event"),
                    delimiter = Text.PositionOf(line, "|"),
                    level = if delimiter < 0 then "" else Text.Trim(Text.Start(line, delimiter)),
                    message = if delimiter < 0 then "" else Text.Trim(Text.Range(line, delimiter + 1))
                in if not List.Contains({"INFO", "WARN", "ERROR"}, level) or message = ""
                   then error "A log line needs a valid level, |, and a message"
                   else [Level = level, Message = message]),
            table = Table.FromRecords(parsed),
            groups = Table.Group(table, {"Level"}, {{"Count", each Table.RowCount(_), Int64.Type}}),
            errors = Table.SelectRows(table, each [Level] = "ERROR"),
            errorGroups = Table.Group(errors, {"Message"}, {{"Count", each Table.RowCount(_), Int64.Type}}),
            ranked = Table.Sort(errorGroups, {{"Count", Order.Descending}, {"Message", Order.Ascending}}),
            counts = Record.FromList(groups[Count], groups[Level]),
            result = [counts = counts, top_error = if Table.IsEmpty(ranked) then null else ranked{0}[Message]]
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

`Evaluate` applies the log analysis rules. `TryResult` serializes evaluated records inside a try expression so field errors can be surfaced. A typed output table exposes the metric and diagnostics for the recovery view.

## Sample input data

The source embeds this synthetic JSON fixture so the query can run without a separate file.

```json
{
  "primary": {
    "asset": "BRANCH-08",
    "events": [
      "INFO | Started",
      "ERROR | Timeout",
      "ERROR | Timeout",
      "WARN | Retrying"
    ]
  },
  "fallback": {
    "asset": "BRANCH-08",
    "events": [
      "INFO | Started",
      "ERROR | Timeout",
      "ERROR | Timeout",
      "WARN | Retrying"
    ]
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
  "top_error": "Timeout"
}
```

## Testing and validation

Microsoft's Power Query parser accepted the embedded M code. An independent Python reference check returned the expected field shown below. Power BI Desktop calculation and rendering remain unverified.

```json
{
  "top_error": "Timeout"
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

Practice log analysis using Power BI and check a repeatable recovery workflow in the branch office simulator scenario.

## Conclusion

The project records an observable log analysis result for training case `BRANCH-08`.

[Back to the Power BI project index](../README.md)
