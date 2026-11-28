# Power BI: Scheduled Reporting — Baseline Setup for Offline Classroom (Project 032476)

## Project overview

A classroom operating offline provides the setting for this scheduled reporting exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The offline classroom needs a refreshable Power BI table for scheduled reporting with baseline setup.

## Project objectives

- Calculate a due date and the number of days overdue.
- Show the checked result and its reference case.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Power BI Desktop, Power Query M, JSON, and a table visual.

## System architecture

`Embedded JSON → Power Query parsing and validation → workflow calculation → report table`

## Features

- Calculate a due date and the number of days overdue.
- Show the checked result and its reference case.

## Prerequisites

Use Power BI Desktop on Windows. Use a text editor and a local practice directory.

## Complete Power BI source code

```powerquery
// Power Query reference implementation for project 032476.
let
    Metric = "days_overdue",
    InputJson = "{""asset"":""CLASS-09"",""previous_report"":""2026-06-01"",""as_of"":""2026-06-16"",""interval_days"":7}",
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
    Day = (value as any) as date =>
        let
            input = Nonblank(value, "date"),
            parsed = Date.FromText(input, [Format = "yyyy-MM-dd", Culture = "en-US"])
        in if Date.ToText(parsed, [Format = "yyyy-MM-dd", Culture = "en-US"]) <> input
           then error "Expected a date in YYYY-MM-DD format" else parsed,
    Evaluate = (r as record) as record =>
        let
            previous = Day(r[previous_report]),
            asOf = Day(r[as_of]),
            interval = Whole(r[interval_days], "interval_days", 1, 365),
            due = if asOf < previous then error "as_of precedes previous_report" else Date.AddDays(previous, interval),
            result = [due_on = Date.ToText(due, [Format = "yyyy-MM-dd"]), due = asOf >= due,
                      days_overdue = List.Max({0, Duration.Days(asOf - due)})]
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

`Evaluate` applies the scheduled reporting rules. `TryResult` serializes evaluated records inside a try expression so field errors can be surfaced. A typed output table exposes the metric and diagnostics for the baseline setup view.

## Sample input data

The source embeds this synthetic JSON fixture so the query can run without a separate file.

```json
{
  "asset": "CLASS-09",
  "previous_report": "2026-06-01",
  "as_of": "2026-06-16",
  "interval_days": 7
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
  "days_overdue": 8
}
```

## Testing and validation

Microsoft's Power Query parser accepted the embedded M code. An independent Python reference check returned the expected field shown below. Power BI Desktop calculation and rendering remain unverified.

```json
{
  "days_overdue": 8
}
```

## Error handling

The query turns evaluation failures into rejected rows with a diagnostic. Recovery uses the supplied fallback if its primary fails.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The query calculates a due date when refreshed. Scheduled Power BI dataset refresh requires separate configuration.

## Learning outcomes

Practice scheduled reporting using Power BI and check a repeatable baseline setup workflow in the offline classroom scenario.

## Conclusion

The project records an observable scheduled reporting result for training case `CLASS-09`.

[Back to the Power BI project index](../README.md)
