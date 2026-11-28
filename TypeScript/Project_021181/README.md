# TypeScript: Log Analysis — Baseline Setup for Single-Host Lab (Project 021181)

## Project overview

One simulated lab host provides the setting for this log analysis exercise. The code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

One simulated lab host needs a repeatable way to handle log analysis from local records. Case `HOST-03` illustrates baseline setup.

## Project objectives

- Count event levels in a synthetic log and identify the most frequent error.
- Establish a repeatable reference run for one input record.
- Verify the sample result, repeat the run, and observe how bad input is handled.

## Technologies used

TypeScript, Node.js, JSON, and built-in Node.js modules. The runtime requires no external packages.

## System architecture

`Local JSON → file reader → field validation → log analysis rules → baseline setup response → terminal`

## Features

- Count event levels in a synthetic log and identify the most frequent error.
- The terminal shows a reference result for a single snapshot.

## Prerequisites

Install Node.js 24 or later. Use an editor and a writable practice directory with synthetic records or data you have permission to use.

## Complete TypeScript source code

```typescript
// Local reference implementation for project 021181.
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

function array(value: unknown, name: string, allowEmpty = false): unknown[] {
  if (!Array.isArray(value) || (!allowEmpty && !value.length)) {
    throw new Error(`${name} must be ${allowEmpty ? "an array" : "a non-empty array"}`);
  }
  return value;
}

function strings(value: unknown, name: string, allowEmpty = false): string[] {
  return array(value, name, allowEmpty).map((item, i) => text(item, `${name}[${i}]`));
}

function errorMessage(error: unknown): string {
  return error instanceof Error ? error.message : String(error);
}

function load(path: string): Data {
  const value: unknown = JSON.parse(readFileSync(path, "utf8"));
  return object(value, "input");
}

function evaluate(record: Data): Data {
  const levels = new Map<string, number>();
  const errors = new Map<string, number>();
  for (const line of strings(record.events, "events")) {
    const separator = line.indexOf("|");
    const level = line.slice(0, separator).trim();
    const message = line.slice(separator + 1).trim();
    if (separator < 0 || !["INFO", "WARN", "ERROR"].includes(level) || !message) {
      throw new Error("Use INFO, WARN or ERROR followed by | and a message");
    }
    levels.set(level, (levels.get(level) ?? 0) + 1);
    if (level === "ERROR") errors.set(message, (errors.get(message) ?? 0) + 1);
  }
  const ranked = [...errors].sort(([a, countA], [b, countB]) =>
    countB - countA || (a < b ? -1 : a > b ? 1 : 0));
  return { counts: Object.fromEntries(levels), top_error: ranked[0]?.[0] ?? null };
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

`evaluate()` applies the log analysis rules to one record. Input JSON supplies one record and the CLI identifies the case so later runs can be compared. The exported `run()` function handles arguments and returns the status and output used by the command-line entry point. TypeScript uses explicit types for the command-line interface and treats parsed input as unknown until validation.

## Sample input data

Save the following synthetic data as `sample.json`.

```json
{
  "asset": "HOST-03",
  "events": [
    "INFO | Started",
    "ERROR | Timeout",
    "ERROR | Timeout",
    "WARN | Retrying"
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
  "top_error": "Timeout"
}
```

## Testing and validation

Actual checks used Node.js 24 and temporary fixtures. The CLI returned status 0 for valid input. A repeat run produced the same JSON and the suite asserted the observed field below. An incomplete record returned status 2 with a diagnostic. The source also passed TypeScript strict type checking.

```json
{
  "top_error": "Timeout"
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

Practice designing local command-line workflows, validating records, applying log analysis rules, and testing baseline setup.

## Conclusion

The sample demonstrates log analysis in the single-host lab scenario with a repeatable baseline setup run.

[Back to the TypeScript project index](../README.md)
