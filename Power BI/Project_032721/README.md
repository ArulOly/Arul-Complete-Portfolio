# Power BI: Error Handling — Baseline Setup for Branch Office Simulator (Project 032721)

## Project overview

A simulated branch office provides the setting for this error handling exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The branch office simulator needs a refreshable Power BI table for error handling with baseline setup.

## Project objectives

- Calculate rates and retain per-row input errors.
- Show the checked result and its reference case.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Power BI Desktop, Power Query M, JSON, and a table visual.

## System architecture

`Embedded JSON → Power Query parsing and validation → workflow calculation → report table`

## Features

- Calculate rates and retain per-row input errors.
- Show the checked result and its reference case.

## Prerequisites

Use Power BI Desktop on Windows. Use a text editor and a local practice directory.

## Complete Power BI source code

```powerquery
// Power Query reference implementation for project 032721.
let
    Metric = "rates",
    InputJson = "{""asset"":""BRANCH-08"",""requests"":[{""units"":24,""hours"":3},{""units"":7,""hours"":0}]}",
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
    RequiredList = (value as any, name as text) as list =>
        if not Value.Is(value, type list) or List.IsEmpty(value)
        then error (name & " requires a non-empty list") else value,
    Evaluate = (r as record) as record =>
        let
            requests = RequiredList(r[requests], "requests"),
            attempts = List.Transform(List.Positions(requests), (i) =>
                let
                    attempt = try Number.Round(Whole(requests{i}[units], "units", 0, 1000000000)
                        / Whole(requests{i}[hours], "hours", 1, 1000000000), 2)
                in if attempt[HasError] then [row = i + 1, diagnostic = attempt[Error][Message]]
                   else [row = i + 1, units_per_hour = attempt[Value]]),
            result = [rates = List.Select(attempts, each Record.HasFields(_, "units_per_hour")),
                      errors = List.Select(attempts, each Record.HasFields(_, "diagnostic"))]
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
    Shown = TryResult(Input),
    Rows = {MakeRow(Input, Shown, "Reference baseline", null, 1)},
    Output = Table.TransformColumnTypes(Table.FromRecords(Rows),
        {{"Row", Int64.Type}, {"Case", type text}, {"View", type text}, {"Metric", type text},
         {"Value", type text}, {"Status", type text}, {"Diagnostic", type text}, {"ResultJson", type text}})
in
    Output
```

## Implementation explanation

`Evaluate` applies the error handling rules. `TryResult` serializes evaluated records inside a try expression so field errors can be surfaced. A typed output table exposes the metric and diagnostics for the baseline setup view.

## Sample input data

The source embeds this synthetic JSON fixture so the query can run without a separate file.

```json
{
  "asset": "BRANCH-08",
  "requests": [
    {
      "units": 24,
      "hours": 3
    },
    {
      "units": 7,
      "hours": 0
    }
  ]
}
```

## Execution instructions

In Power BI Desktop, select **Home → Transform data**. Create a **Blank Query** and open **Advanced Editor**. Replace the editor contents with the M code above and select **Done**. Choose **Close & Apply** to load the output table.

Add a **Table** visual using **Case**, **Metric**, and **Value**.

The source includes the JSON sample in `InputJson`. Edit that text literal to try another fixture, preserving the doubled quotes required by M. Refresh the query to update the table.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "rates": [
    {
      "row": 1,
      "units_per_hour": 8.0
    }
  ]
}
```

## Testing and validation

Microsoft's Power Query parser accepted the embedded M code. An independent Python reference check returned the expected field shown below. Power BI Desktop calculation and rendering remain unverified.

```json
{
  "rates": [
    {
      "row": 1,
      "units_per_hour": 8.0
    }
  ]
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

Practice error handling using Power BI and check a repeatable baseline setup workflow in the branch office simulator scenario.

## Conclusion

The project records an observable error handling result for training case `BRANCH-08`.

[Back to the Power BI project index](../README.md)
