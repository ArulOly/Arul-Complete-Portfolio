# Power BI: Recovery Test — Usability Improvement for Branch Office Simulator (Project 032975)

## Project overview

A simulated branch office provides the setting for this recovery test exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The branch office simulator needs a refreshable Power BI table for recovery test with usability improvement.

## Project objectives

- Compare restored snapshot fields with expected values.
- Provide a readable metric label and keep details available for inspection.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Power BI Desktop, Power Query M, JSON, and a table visual.

## System architecture

`Embedded JSON → Power Query parsing and validation → workflow calculation → report table`

## Features

- Compare restored snapshot fields with expected values.
- Provide a readable metric label and keep details available for inspection.

## Prerequisites

Use Power BI Desktop on Windows. Use a text editor and a local practice directory.

## Complete Power BI source code

```powerquery
// Power Query reference implementation for project 032975.
let
    Metric = "verified",
    InputJson = "{""asset"":""BRANCH-08"",""snapshot"":{""version"":3,""active"":true},""expected"":{""version"":3,""active"":true}}",
    Input = Json.Document(InputJson),
    Nonblank = (value as any, name as text) as text =>
        if not Value.Is(value, type text) or Text.Trim(value) = ""
        then error (name & " must be non-empty text") else Text.Trim(value),
    AsRecord = (value as any) as record =>
        if Value.Is(value, type record) then value else error "Expected a record",
    Evaluate = (r as record) as record =>
        let
            restored = r[snapshot],
            expected = r[expected],
            keys = List.Union({Record.FieldNames(restored), Record.FieldNames(expected)}),
            differences = List.Sort(List.Select(keys, each not Record.HasFields(restored, _)
                or not Record.HasFields(expected, _) or not Value.Equals(Record.Field(restored, _), Record.Field(expected, _)))),
            result = [restored = restored, differences = differences, verified = List.IsEmpty(differences)]
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
    Attempt = TryResult(Input),
    Summary = MakeRow(Input, Attempt, "Quick view", null, 1),
    Rows = {Record.AddField(Summary, "DisplayLabel", Text.Proper(Text.Replace(Metric, "_", " ")))},
    Output = Table.TransformColumnTypes(Table.FromRecords(Rows),
        {{"Row", Int64.Type}, {"Case", type text}, {"View", type text}, {"Metric", type text},
         {"Value", type text}, {"Status", type text}, {"Diagnostic", type text}, {"ResultJson", type text}})
in
    Output
```

## Implementation explanation

`Evaluate` applies the recovery test rules. `TryResult` serializes evaluated records inside a try expression so field errors can be surfaced. A typed output table exposes the metric and diagnostics for the usability improvement view.

## Sample input data

The source embeds this synthetic JSON fixture so the query can run without a separate file.

```json
{
  "asset": "BRANCH-08",
  "snapshot": {
    "version": 3,
    "active": true
  },
  "expected": {
    "version": 3,
    "active": true
  }
}
```

## Execution instructions

In Power BI Desktop, select **Home → Transform data**. Create a **Blank Query** and open **Advanced Editor**. Replace the editor contents with the M code above and select **Done**. Choose **Close & Apply** to load the output table.

Add a **Table** using **DisplayLabel**, **Value**, and **Status**. Use **ResultJson** to inspect full values.

The source includes the JSON sample in `InputJson`. Edit that text literal to try another fixture, preserving the doubled quotes required by M. Refresh the query to update the table.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "verified": true
}
```

## Testing and validation

Microsoft's Power Query parser accepted the embedded M code. An independent Python reference check returned the expected field shown below. Power BI Desktop calculation and rendering remain unverified.

```json
{
  "verified": true
}
```

## Error handling

The query turns evaluation failures into rejected rows with a diagnostic. Recovery uses the supplied fallback if its primary fails.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

It compares a fictional restored snapshot with expected fields.

## Learning outcomes

Practice recovery test using Power BI and check a repeatable usability improvement workflow in the branch office simulator scenario.

## Conclusion

The project records an observable recovery test result for training case `BRANCH-08`.

[Back to the Power BI project index](../README.md)
