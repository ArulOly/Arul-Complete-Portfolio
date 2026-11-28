# TypeScript: Dashboard — Usability Improvement for School Lab (Project 021355)

## Project overview

Shared classroom equipment provides the setting for this dashboard exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

Shared classroom equipment needs a repeatable way to handle dashboard from local records. Case `LAB-17` illustrates usability improvement.

## Project objectives

- Aggregate ticket counts and show open workload as a share of all tickets.
- Provide a discoverable help screen and a readable terminal summary.
- Verify the sample result, repeat the run, and observe how bad input is handled.

## Technologies used

TypeScript, Node.js, JSON, and built-in Node.js modules. The runtime requires no external packages.

## System architecture

`Local JSON → file reader → field validation → dashboard rules → usability improvement response → terminal`

## Features

- Aggregate ticket counts and show open workload as a share of all tickets.
- The operator can use --help to inspect arguments and --json for programmatic review.

## Prerequisites

Install Node.js 24 or later. Use an editor and a writable practice directory with synthetic records or data you have permission to use.

## Complete TypeScript source code

```typescript
// Local reference implementation for project 021355.
import { readFileSync } from "node:fs";
import { pathToFileURL } from "node:url";
import { parseArgs } from "node:util";

type Data = Record<string, unknown>;
type Reply = { status: number; output: string; error?: string };

const SCENARIO = "School Lab";

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
  const queue = object(record.queue, "queue");
  const counts = {
    open: integer(queue.open, "open"),
    in_progress: integer(queue.in_progress, "in_progress"),
    closed: integer(queue.closed, "closed")
  };
  const active = counts.open + counts.in_progress;
  const total = active + counts.closed;
  return { counts, total, active_pct: total ? round(100 * active / total, 1) : 0 };
}

function reply(value: Data, status = 0): Reply {
  return { status, output: JSON.stringify(value, null, 2) };
}

export function run(args: string[]): Reply {
  const usage = "Usage: node project.mts sample.json [--json]";
  try {
    const { values, positionals } = parseArgs({
      args, allowPositionals: true, options: { json: { type: "boolean" }, help: { type: "boolean" } },
    });
    if (values.help) return { status: 0, output: usage };
    if (positionals.length !== 1) throw new Error(usage);
    const [input] = positionals;

    const record = load(input);
    const asset = text(record.asset, "asset");
    const result = evaluate(record);
    if (values.json) return reply({ asset, result });
    const lines = Object.entries(result).map(([key, value]) =>
      `${key.replaceAll("_", " ")}: ${JSON.stringify(value)}`);
    return { status: 0, output: [`${SCENARIO} | ${asset}`, ...lines].join("\n") };
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

`evaluate()` applies the dashboard rules to one record. The CLI converts each result field into a labeled line or outputs JSON with a flag. The exported `run()` function handles arguments and returns the status and output used by the command-line entry point. TypeScript uses explicit types for the command-line interface and treats parsed input as unknown until validation.

## Sample input data

Save the following synthetic data as `sample.json`.

```json
{
  "asset": "LAB-17",
  "queue": {
    "open": 5,
    "in_progress": 3,
    "closed": 12
  }
}
```

## Execution instructions

1. Copy the source block into `project.mts`.
2. Save the sample input alongside it.
3. Run this command in that directory.

Omit `--json` for labeled text. Pass `--help` to list the arguments.

```sh
node project.mts sample.json --json
```

## Example output

Illustrative excerpt. The CLI response includes additional fields.

```json
{
  "active_pct": 40.0
}
```

## Testing and validation

Actual checks used Node.js 24 and temporary fixtures. The CLI returned status 0 for valid input. A repeat run produced the same JSON and the suite asserted the observed field below. An incomplete record returned status 2. Help documented the JSON flag, and the default view labeled its fields. The source also passed TypeScript strict type checking.

```json
{
  "active_pct": 40
}
```

## Error handling

The CLI returns status 2 for a bad invocation or a file or input error. The operator can use --help to inspect arguments and --json for programmatic review.

## Troubleshooting

- Use `node --version` to check the runtime and confirm the program and sample names.
- Check the diagnostic and compare field types with the sample.
- Run the original sample again before testing a different setting.

## Limitations

The exercise uses local synthetic records and covers the workflow described in the sample.

## Learning outcomes

Practice designing local command-line workflows, validating records, applying dashboard rules, and testing usability improvement.

## Conclusion

The sample demonstrates dashboard in the school lab scenario with a repeatable usability improvement run.

[Back to the TypeScript project index](../README.md)
