# Node.js: Monitoring — Recovery for Two-Node Lab (Project 024088)

## Project overview

A two-node lab simulation provides the setting for this monitoring exercise. The code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

A two-node lab simulation needs a repeatable way to handle monitoring from local records. Case `NODE-B` illustrates recovery.

## Project objectives

- Summarize utilization readings and count readings above the limit.
- Try a fallback input snapshot when the primary input is missing or invalid.
- Verify the sample result, repeat the run, and observe how bad input is handled.

## Technologies used

Node.js, Node.js, JSON, and built-in Node.js modules. The runtime requires no external packages.

## System architecture

`Local JSON → file reader → field validation → monitoring rules → recovery response → terminal`

## Features

- Summarize utilization readings and count readings above the limit.
- The output identifies the input role and gives the reason when the fallback is used.

## Prerequisites

Install Node.js 24 or later. Use an editor and a writable practice directory with synthetic records or data you have permission to use.

## Complete Node.js source code

```javascript
// Local reference implementation for project 024088.
import { readFileSync } from "node:fs";
import { pathToFileURL } from "node:url";
import { parseArgs } from "node:util";

function object(value, name) {
  if (typeof value !== "object" || value === null || Array.isArray(value)) {
    throw new Error(`${name} must be an object`);
  }
  return value;
}

function text(value, name, trim = true) {
  if (typeof value !== "string" || !value.trim()) throw new Error(`${name} must be non-empty text`);
  return trim ? value.trim() : value;
}

function integer(value, name, min = 0, max = 1_000_000_000) {
  if (typeof value !== "number" || !Number.isSafeInteger(value) || value < min || value > max) {
    throw new Error(`${name} must be an integer from ${min} to ${max}`);
  }
  return value;
}

function array(value, name, allowEmpty = false) {
  if (!Array.isArray(value) || (!allowEmpty && !value.length)) {
    throw new Error(`${name} must be ${allowEmpty ? "an array" : "a non-empty array"}`);
  }
  return value;
}

function numbers(value, name, max = 1_000_000_000) {
  return array(value, name).map((n, i) => integer(n, `${name}[${i}]`, 0, max));
}

function round(value, places = 2) {
  return Number(value.toFixed(places));
}

function errorMessage(error) {
  return error instanceof Error ? error.message : String(error);
}

function load(path) {
  const value = JSON.parse(readFileSync(path, "utf8"));
  return object(value, "input");
}

function evaluate(record) {
  const readings = numbers(record.readings_pct, "readings_pct", 100);
  const limit = integer(record.limit_pct, "limit_pct", 0, 100);
  const total = readings.reduce((sum, n) => sum + n, 0);
  return { mean_pct: round(total / readings.length), peak_pct: readings.reduce((peak, n) => Math.max(peak, n), 0),
    above_limit: readings.filter(n => n > limit).length };
}

function reply(value, status = 0) {
  return { status, output: JSON.stringify(value, null, 2) };
}

export function run(args) {
  const usage = "Usage: node project.mjs sample.json --fallback fallback.json";
  try {
    const { values, positionals } = parseArgs({
      args, allowPositionals: true, options: { fallback: { type: "string" }, help: { type: "boolean" } },
    });
    if (values.help) return { status: 0, output: usage };
    if (positionals.length !== 1) throw new Error(usage);
    const [input] = positionals;

    const fallback = text(values.fallback, "--fallback");
    try {
      const record = load(input);
      return reply({ source: "primary", asset: text(record.asset, "asset"), result: evaluate(record) });
    } catch (primary) {
      const record = load(fallback);
      return reply({ source: "fallback", asset: text(record.asset, "asset"),
        primary_error: errorMessage(primary), result: evaluate(record) });
    }
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

`evaluate()` applies the monitoring rules to one record. The primary file is evaluated first. An input error triggers a fresh read of the fallback. The exported `run()` function handles arguments and returns the status and output used by the command-line entry point.

## Sample input data

Save the following synthetic data as `sample.json`.

```json
{
  "asset": "NODE-B",
  "readings_pct": [
    35,
    81,
    54
  ],
  "limit_pct": 80
}
```

## Execution instructions

1. Copy the source block into `project.mjs`.
2. Save the sample input alongside it.
3. Run this command in that directory.

Save a copy of the sample as `fallback.json`. Use a missing primary filename to exercise recovery.

```sh
node project.mjs sample.json --fallback fallback.json
```

## Example output

Illustrative excerpt. The CLI response includes additional fields.

```json
{
  "above_limit": 1
}
```

## Testing and validation

Actual checks used Node.js 24 and temporary fixtures. The CLI returned status 0 for valid input. A repeat run produced the same JSON and the suite asserted the observed field below. An incomplete primary input used the fallback. An unavailable fallback returned status 2.

```json
{
  "above_limit": 1
}
```

## Error handling

The CLI returns status 2 for a bad invocation or a file or input error. The output identifies the input role and gives the reason when the fallback is used.

## Troubleshooting

- Use `node --version` to check the runtime and confirm the program and sample names.
- Check the diagnostic and compare field types with the sample.
- Run the original sample again before testing a different setting.

## Limitations

The exercise uses local synthetic records and covers the workflow described in the sample.

## Learning outcomes

Practice designing local command-line workflows, validating records, applying monitoring rules, and testing recovery.

## Conclusion

The sample demonstrates monitoring in the two-node lab scenario with a repeatable recovery run.

[Back to the Node.js project index](../README.md)
