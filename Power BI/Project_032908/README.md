# Power BI: Visual Reporting — Recovery for Small Business (Project 032908)

## Project overview

Local shop operations provides the setting for this visual reporting exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The small business needs a refreshable Power BI table for visual reporting with recovery.

## Project objectives

- Build a scaled text chart with exact values.
- Select the fallback when the primary record cannot be evaluated.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Power BI Desktop, Power Query M, JSON, and a table visual.

## System architecture

`Embedded JSON → Power Query parsing and validation → workflow calculation → report table`

## Features

- Build a scaled text chart with exact values.
- Select the fallback when the primary record cannot be evaluated.

## Prerequisites

Use Power BI Desktop on Windows. Use a text editor and a local practice directory.

## Complete Power BI source code

```powerquery
// Power Query reference implementation for project 032908.
let
    Metric = "peak",
    InputJson = "{""primary"":{""asset"":""SHOP-22"",""values"":{""available"":12,""reserved"":7,""repair"":3},""width"":12},""fallback"":{""asset"":""SHOP-22"",""values"":{""available"":12,""reserved"":7,""repair"":3},""width"":12}}",
    Input = Json.Document(InputJson),
    Nonblank = (value as any, name as text) as text =>
        if not Value.Is(value, type text) or Text.Trim(value) = ""
        then error (name & " must be non-empty text") else Text.Trim(value),
    Whole = (value as any, name as text, minimum as number, maximum as number) as number =>
        if not Value.Is(value, type number) or value < minimum or value > maximum
            or Number.RoundDown(value) <> value
        then error (name & " must be a whole number within its configured range") else value,
    AsRecord = (value as any) as record =>
        if Value.Is(value, type record) then value else error "Expected a record",
    Evaluate = (r as record) as record =>
        let
            fields = Record.FieldNames(r[values]),
            names = if List.IsEmpty(fields) then error "At least one category is required" else List.Sort(fields),
            counts = List.Transform(names, each Whole(Record.Field(r[values], _), "category value", 0, 1000000000)),
            width = Whole(r[width], "width", 1, 60),
            peak = List.Max(counts),
            bars = List.Transform(List.Positions(names), (i) =>
                names{i} & ": " & Text.Repeat("#", if peak = 0 then 0 else Number.Round(counts{i} * width / peak, 0, RoundingMode.AwayFromZero))
                & " (" & Number.ToText(counts{i}, "0", "en-US") & ")"),
            result = [peak = peak, max_bar_width = width, bars = bars]
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
    PrimaryInput = Record.FieldOrDefault(Input, "primary", null),
    Primary = TryResult(PrimaryInput),
    Selected = if Primary[HasError] then Record.FieldOrDefault(Input, "fallback", null) else PrimaryInput,
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

`Evaluate` applies the visual reporting rules. `TryResult` serializes evaluated records inside a try expression so field errors can be surfaced. A typed output table exposes the metric and diagnostics for the recovery view.

## Sample input data

The source embeds this synthetic JSON fixture so the query can run without a separate file.

```json
{
  "primary": {
    "asset": "SHOP-22",
    "values": {
      "available": 12,
      "reserved": 7,
      "repair": 3
    },
    "width": 12
  },
  "fallback": {
    "asset": "SHOP-22",
    "values": {
      "available": 12,
      "reserved": 7,
      "repair": 3
    },
    "width": 12
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
  "peak": 12
}
```

## Testing and validation

Microsoft's Power Query parser accepted the embedded M code. An independent Python reference check returned the expected field shown below. Power BI Desktop calculation and rendering remain unverified.

```json
{
  "peak": 12
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

Practice visual reporting using Power BI and check a repeatable recovery workflow in the small business scenario.

## Conclusion

The project records an observable visual reporting result for training case `SHOP-22`.

[Back to the Power BI project index](../README.md)
