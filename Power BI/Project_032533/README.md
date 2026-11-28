# Power BI: Access Review — Recovery for Single-Host Lab (Project 032533)

## Project overview

One simulated lab host provides the setting for this access review exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The single-host lab needs a refreshable Power BI table for access review with recovery.

## Project objectives

- Compare synthetic grants with the approved role policy.
- Select the fallback when the primary record cannot be evaluated.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Power BI Desktop, Power Query M, JSON, and a table visual.

## System architecture

`Embedded JSON → Power Query parsing and validation → workflow calculation → report table`

## Features

- Compare synthetic grants with the approved role policy.
- Select the fallback when the primary record cannot be evaluated.

## Prerequisites

Use Power BI Desktop on Windows. Use a text editor and a local practice directory.

## Complete Power BI source code

```powerquery
// Power Query reference implementation for project 032533.
let
    Metric = "excess_grants",
    InputJson = "{""primary"":{""asset"":""HOST-03"",""approved"":[""read"",""submit""],""grants"":[""read"",""admin""]},""fallback"":{""asset"":""HOST-03"",""approved"":[""read"",""submit""],""grants"":[""read"",""admin""]}}",
    Input = Json.Document(InputJson),
    Nonblank = (value as any, name as text) as text =>
        if not Value.Is(value, type text) or Text.Trim(value) = ""
        then error (name & " must be non-empty text") else Text.Trim(value),
    AsRecord = (value as any) as record =>
        if Value.Is(value, type record) then value else error "Expected a record",
    TextList = (value as list) as list =>
        List.Distinct(List.Transform(value, each Nonblank(_, "list item"))),
    Evaluate = (r as record) as record =>
        let
            approved = TextList(r[approved]),
            grants = TextList(r[grants]),
            excess = List.Sort(List.Difference(grants, approved)),
            missing = List.Sort(List.Difference(approved, grants)),
            result = [excess_grants = excess, missing_grants = missing,
                      matches_policy = List.IsEmpty(excess) and List.IsEmpty(missing)]
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

`Evaluate` applies the access review rules. `TryResult` serializes evaluated records inside a try expression so field errors can be surfaced. A typed output table exposes the metric and diagnostics for the recovery view.

## Sample input data

The source embeds this synthetic JSON fixture so the query can run without a separate file.

```json
{
  "primary": {
    "asset": "HOST-03",
    "approved": [
      "read",
      "submit"
    ],
    "grants": [
      "read",
      "admin"
    ]
  },
  "fallback": {
    "asset": "HOST-03",
    "approved": [
      "read",
      "submit"
    ],
    "grants": [
      "read",
      "admin"
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
  "excess_grants": [
    "admin"
  ]
}
```

## Testing and validation

Microsoft's Power Query parser accepted the embedded M code. An independent Python reference check returned the expected field shown below. Power BI Desktop calculation and rendering remain unverified.

```json
{
  "excess_grants": [
    "admin"
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

Practice access review using Power BI and check a repeatable recovery workflow in the single-host lab scenario.

## Conclusion

The project records an observable access review result for training case `HOST-03`.

[Back to the Power BI project index](../README.md)
