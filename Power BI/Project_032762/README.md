# Power BI: Data Validation — Automation for Home Development (Project 032762)

## Project overview

A home development workspace provides the setting for this data validation exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The home development needs a refreshable Power BI table for data validation with automation.

## Project objectives

- Accept valid values and capture invalid or duplicate codes.
- Refresh a batch and keep the outcome of each input row.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Power BI Desktop, Power Query M, JSON, and a table visual.

## System architecture

`Embedded JSON → Power Query parsing and validation → workflow calculation → report table`

## Features

- Accept valid values and capture invalid or duplicate codes.
- Refresh a batch and keep the outcome of each input row.

## Prerequisites

Use Power BI Desktop on Windows. Use a text editor and a local practice directory.

## Complete Power BI source code

```powerquery
// Power Query reference implementation for project 032762.
let
    Metric = "accepted",
    InputJson = "{""records"":[{""asset"":""DEV-05"",""rows"":[{""code"":""KIT-01"",""count"":4},{""code"":""KIT-01"",""count"":2},{""code"":""KIT-02"",""count"":-1}]},{""asset"":""DEV-05-B"",""rows"":[{""code"":""KIT-01"",""count"":4},{""code"":""KIT-01"",""count"":2},{""code"":""KIT-02"",""count"":-1}]}]}",
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
            rows = RequiredList(r[rows], "rows"),
            state = List.Accumulate(List.Positions(rows), [Accepted = [], Rejected = {}], (previous, i) =>
                let
                    entry = rows{i},
                    attempt = try
                        let
                            code = Text.Upper(Nonblank(entry[code], "code"), "en-US"),
                            amount = Whole(entry[count], "count", 0, 1000000000)
                        in if Record.HasFields(previous[Accepted], code) then error "Duplicate code"
                           else Record.AddField(previous[Accepted], code, amount)
                in if attempt[HasError] then [Accepted = previous[Accepted],
                    Rejected = previous[Rejected] & {[row = i + 1, diagnostic = attempt[Error][Message]]}]
                   else [Accepted = attempt[Value], Rejected = previous[Rejected]]),
            result = [accepted = state[Accepted], rejected = state[Rejected]]
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
    Inputs = RequiredList(Input[records], "records"),
    Rows = List.Transform(List.Positions(Inputs), (i) =>
        let
            item = Inputs{i},
            attempted = TryResult(item)
        in MakeRow(item, attempted, "Batch", null, i + 1)),
    Output = Table.TransformColumnTypes(Table.FromRecords(Rows),
        {{"Row", Int64.Type}, {"Case", type text}, {"View", type text}, {"Metric", type text},
         {"Value", type text}, {"Status", type text}, {"Diagnostic", type text}, {"ResultJson", type text}})
in
    Output
```

## Implementation explanation

`Evaluate` applies the data validation rules. `TryResult` serializes evaluated records inside a try expression so field errors can be surfaced. A typed output table exposes the metric and diagnostics for the automation view.

## Sample input data

The source embeds this synthetic JSON fixture so the query can run without a separate file.

```json
{
  "records": [
    {
      "asset": "DEV-05",
      "rows": [
        {
          "code": "KIT-01",
          "count": 4
        },
        {
          "code": "KIT-01",
          "count": 2
        },
        {
          "code": "KIT-02",
          "count": -1
        }
      ]
    },
    {
      "asset": "DEV-05-B",
      "rows": [
        {
          "code": "KIT-01",
          "count": 4
        },
        {
          "code": "KIT-01",
          "count": 2
        },
        {
          "code": "KIT-02",
          "count": -1
        }
      ]
    }
  ]
}
```

## Execution instructions

In Power BI Desktop, select **Home → Transform data**. Create a **Blank Query** and open **Advanced Editor**. Replace the editor contents with the M code above and select **Done**. Choose **Close & Apply** to load the output table.

Add a **Table** with **Row**, **Case**, **Status**, and **Value**. Filter **Status** to inspect rejected rows.

The source includes the JSON sample in `InputJson`. Edit that text literal to try another fixture, preserving the doubled quotes required by M. Refresh the query to update the table.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "accepted": {
    "KIT-01": 4
  }
}
```

## Testing and validation

Microsoft's Power Query parser accepted the embedded M code. An independent Python reference check returned the expected field shown below. Power BI Desktop calculation and rendering remain unverified.

```json
{
  "accepted": {
    "KIT-01": 4
  }
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

Practice data validation using Power BI and check a repeatable automation workflow in the home development scenario.

## Conclusion

The project records an observable data validation result for training case `DEV-05`.

[Back to the Power BI project index](../README.md)
