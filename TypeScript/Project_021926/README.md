# TypeScript: Visual Reporting — Baseline Setup for Offline Classroom (Project 021926)

## Project overview

A classroom operating offline provides the setting for this visual reporting exercise. The code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

A classroom operating offline needs a repeatable way to handle visual reporting from local records. Case `CLASS-09` illustrates baseline setup.

## Project objectives

- Create a proportional text bar chart with exact values and a labeled scale.
- Establish a repeatable reference run for one input record.
- Verify the sample result, repeat the run, and observe how bad input is handled.

## Technologies used

TypeScript, Node.js, JSON, and built-in Node.js modules. The runtime requires no external packages.

## System architecture

`Local JSON → file reader → field validation → visual reporting rules → baseline setup response → terminal`

## Features

- Create a proportional text bar chart with exact values and a labeled scale.
- The terminal shows a reference result for a single snapshot.

## Prerequisites

Install Node.js 24 or later. Use an editor and a writable practice directory with synthetic records or data you have permission to use.

## Complete TypeScript source code

```typescript
// Local reference implementation for project 021926.
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
  const values = object(record.values, "values");
  const entries = Object.keys(values).sort().map(key => ({ key, count: integer(values[key], key) }));
  if (!entries.length) throw new Error("At least one category is required");
  const width = integer(record.width, "width", 1, 60);
  const peak = entries.reduce((max, entry) => Math.max(max, entry.count), 0);
  const bars = entries.map(({ key, count }) =>
    `${key}: ${"#".repeat(peak ? Math.round(count * width / peak) : 0)} (${count})`);
  return { peak, max_bar_width: width, bars };
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

    const record = load(input);
    return reply({ baseline_for: text(record.asset, "asset"), result: evaluate(record) });
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

`evaluate()` applies the visual reporting rules to one record. Input JSON supplies one record and the CLI identifies the case so later runs can be compared. The exported `run()` function handles arguments and returns the status and output used by the command-line entry point. TypeScript uses explicit types for the command-line interface and treats parsed input as unknown until validation.

## Sample input data

Save the following synthetic data as `sample.json`.

```json
{
  "asset": "CLASS-09",
  "values": {
    "available": 12,
    "reserved": 7,
    "repair": 3
  },
  "width": 12
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
  "peak": 12
}
```

## Testing and validation

Actual checks used Node.js 24 and temporary fixtures. The CLI returned status 0 for valid input. A repeat run produced the same JSON and the suite asserted the observed field below. An incomplete record returned status 2 with a diagnostic. The source also passed TypeScript strict type checking.

```json
{
  "peak": 12
}
```

## Error handling

The CLI returns status 2 for a bad invocation or a file or input error. The terminal shows a reference result for a single snapshot.

## Troubleshooting

- Use `node --version` to check the runtime and confirm the program and sample names.
- Check the diagnostic and compare field types with the sample.
- Run the original sample again before testing a different setting.

## Limitations

The exercise uses local synthetic records and covers the workflow described in the sample.

## Learning outcomes

Practice designing local command-line workflows, validating records, applying visual reporting rules, and testing baseline setup.

## Conclusion

The sample demonstrates visual reporting in the offline classroom scenario with a repeatable baseline setup run.

[Back to the TypeScript project index](../README.md)
