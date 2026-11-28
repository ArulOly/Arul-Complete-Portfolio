# TypeScript: Monitoring — Automation for Branch Office Simulator (Project 021072)

## Project overview

A simulated branch office provides the setting for this monitoring exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

A simulated branch office needs a repeatable way to handle monitoring from local records. Case `BRANCH-08` illustrates automation.

## Project objectives

- Summarize utilization readings and count readings above the limit.
- Process a batch in order and keep valid results when individual records fail.
- Verify the sample result, repeat the run, and observe how bad input is handled.

## Technologies used

TypeScript, Node.js, JSON, and built-in Node.js modules. The runtime requires no external packages.

## System architecture

`Local JSON → file reader → field validation → monitoring rules → automation response → terminal`

## Features

- Summarize utilization readings and count readings above the limit.
- The CLI processes multiple rows in one call and returns status 1 if any row fails.

## Prerequisites

Install Node.js 24 or later. Use an editor and a writable practice directory with synthetic records or data you have permission to use.

## Complete TypeScript source code

```typescript
// Local reference implementation for project 021072.
import { readFileSync } from "node:fs";
import { pathToFileURL } from "node:url";
import { parseArgs } from "node:util";

type Data = Record<string, unknown>;
type Reply = { status: number; output: string; error?: string };

function object(value: unknown, name: string): Data {
  if (typeof value !== "object" || value === null || Array.isArray(value)) {
    throw new Error(`${name} must be an object`);
  }
  return value as Data;
}

function text(value: unknown, name: string, trim = true): string {
  if (typeof value !== "string" || !value.trim()) throw new Error(`${name} must be non-empty text`);
  return trim ? value.trim() : value;
}

function integer(value: unknown, name: string, min = 0, max = 1_000_000_000): number {
  if (typeof value !== "number" || !Number.isSafeInteger(value) || value < min || value > max) {
    throw new Error(`${name} must be an integer from ${min} to ${max}`);
  }
  return value;
}

function array(value: unknown, name: string, allowEmpty = false): unknown[] {
  if (!Array.isArray(value) || (!allowEmpty && !value.length)) {
    throw new Error(`${name} must be ${allowEmpty ? "an array" : "a non-empty array"}`);
  }
  return value;
}

function numbers(value: unknown, name: string, max = 1_000_000_000): number[] {
  return array(value, name).map((n, i) => integer(n, `${name}[${i}]`, 0, max));
}

function round(value: number, places = 2): number {
  return Number(value.toFixed(places));
}

function errorMessage(error: unknown): string {
  return error instanceof Error ? error.message : String(error);
}

function load(path: string): Data {
  const value: unknown = JSON.parse(readFileSync(path, "utf8"));
  return object(value, "input");
}

function evaluate(record: Data): Data {
  const readings = numbers(record.readings_pct, "readings_pct", 100);
  const limit = integer(record.limit_pct, "limit_pct", 0, 100);
  const total = readings.reduce((sum, n) => sum + n, 0);
  return { mean_pct: round(total / readings.length), peak_pct: readings.reduce((peak, n) => Math.max(peak, n), 0),
    above_limit: readings.filter(n => n > limit).length };
}

function reply(value: Data, status = 0): Reply {
  return { status, output: JSON.stringify(value, null, 2) };
}

export function run(args: string[]): Reply {
  const usage = "Usage: node project.mts sample.json";
  try {
    const { values, positionals } = parseArgs({
      args, allowPositionals: true, options: { help: { type: "boolean" } },
    });
    if (values.help) return { status: 0, output: usage };
    if (positionals.length !== 1) throw new Error(usage);
    const [input] = positionals;

    const records = array(load(input).records, "records");
    let status = 0;
    const rows = records.map((value, index) => {
      try {
        const record = object(value, "batch row");
        return { row: index + 1, asset: text(record.asset, "asset"), result: evaluate(record) };
      } catch (error) {
        status = 1;
        return { row: index + 1, error: errorMessage(error) };
      }
    });
    return reply({ rows }, status);
  } catch (error) {
    return { status: 2, output: "", error: "Input or output error: " + errorMessage(error) };
  }
}

// Keep the command-line entry point separate from the testable application logic.
if (process.argv[1] && import.meta.url === pathToFileURL(process.argv[1]).href) {
  const result = run(process.argv.slice(2));
  if (result.output) console.log(result.output);
  if (result.error) console.error(result.error);
  process.exitCode = result.status;
}
```

## Implementation explanation

`evaluate()` applies the monitoring rules to one record. The outer records array keeps each result or error associated with its original row number. The exported `run()` function handles arguments and returns the status and output used by the command-line entry point. TypeScript uses explicit types for the command-line interface and treats parsed input as unknown until validation.

## Sample input data

Save the following synthetic data as `sample.json`.

```json
{
  "records": [
    {
      "asset": "BRANCH-08",
      "readings_pct": [
        35,
        81,
        54
      ],
      "limit_pct": 80
    },
    {
      "asset": "BRANCH-08-B",
      "readings_pct": [
        35,
        81,
        54
      ],
      "limit_pct": 80
    }
  ]
}
```

## Execution instructions

1. Copy the source block into `project.mts`.
2. Save the sample input alongside it.
3. Run this command in that directory.

```sh
node project.mts sample.json
```

## Example output

Illustrative excerpt. The CLI response includes additional fields.

```json
{
  "above_limit": 1
}
```

## Testing and validation

Actual checks used Node.js 24 and temporary fixtures. The CLI returned status 0 for valid input. A repeat run produced the same JSON and the suite asserted the observed field below. A mixed batch preserved its valid row and returned status 1 with a row-level error. The source also passed TypeScript strict type checking.

```json
{
  "above_limit": 1
}
```

## Error handling

The CLI returns status 2 for a bad invocation or a file or input error. The CLI processes multiple rows in one call and returns status 1 if any row fails.

## Troubleshooting

- Use `node --version` to check the runtime and confirm the program and sample names.
- Check the diagnostic and compare field types with the sample.
- Run the original sample again before testing a different setting.

## Limitations

The exercise uses local synthetic records and covers the workflow described in the sample.

## Learning outcomes

Practice designing local command-line workflows, validating records, applying monitoring rules, and testing automation.

## Conclusion

The sample demonstrates monitoring in the branch office simulator scenario with a repeatable automation run.

[Back to the TypeScript project index](../README.md)
