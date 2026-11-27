# Node.js: Service Check — Baseline Setup for Offline Classroom (Project 024626)

## Project overview

A classroom operating offline provides the setting for this service check exercise. The code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

A classroom operating offline needs a repeatable way to handle service check from local records. Case `CLASS-09` illustrates baseline setup.

## Project objectives

- Classify synthetic service probes by status and latency.
- Establish a repeatable reference run for one input record.
- Verify the sample result, repeat the run, and observe how bad input is handled.

## Technologies used

Node.js, Node.js, JSON, and built-in Node.js modules. The runtime requires no external packages.

## System architecture

`Local JSON → file reader → field validation → service check rules → baseline setup response → terminal`

## Features

- Classify synthetic service probes by status and latency.
- The terminal shows a reference result for a single snapshot.

## Prerequisites

Install Node.js 24 or later. Use an editor and a writable practice directory with synthetic records or data you have permission to use.

## Complete Node.js source code

```javascript
// Local reference implementation for project 024626.
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

function objects(value, name) {
  return array(value, name).map((row, i) => object(row, `${name}[${i}]`));
}

function errorMessage(error) {
  return error instanceof Error ? error.message : String(error);
}

function load(path) {
  const value = JSON.parse(readFileSync(path, "utf8"));
  return object(value, "input");
}

function evaluate(record) {
  const max = integer(record.max_ms, "max_ms", 1);
  const checks = objects(record.probes, "probes").map(probe => {
    const status = integer(probe.status, "status", 100, 599);
    const latency = integer(probe.ms, "ms");
    return { service: text(probe.service, "service"), healthy: status >= 200 && status < 300 && latency <= max };
  });
  return { checks, healthy: checks.filter(row => row.healthy).length };
}

function reply(value, status = 0) {
  return { status, output: JSON.stringify(value, null, 2) };
}

export function run(args) {
  const usage = "Usage: node project.mjs sample.json";
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

`evaluate()` applies the service check rules to one record. Input JSON supplies one record and the CLI identifies the case so later runs can be compared. The exported `run()` function handles arguments and returns the status and output used by the command-line entry point.

## Sample input data

Save the following synthetic data as `sample.json`.

```json
{
  "asset": "CLASS-09",
  "probes": [
    {
      "service": "catalog",
      "status": 200,
      "ms": 25
    },
    {
      "service": "print-queue",
      "status": 503,
      "ms": 60
    }
  ],
  "max_ms": 50
}
```

## Execution instructions

1. Copy the source block into `project.mjs`.
2. Save the sample input alongside it.
3. Run this command in that directory.

```sh
node project.mjs sample.json
```

## Example output

Illustrative excerpt. The CLI response includes additional fields.

```json
{
  "healthy": 1
}
```

## Testing and validation

Actual checks used Node.js 24 and temporary fixtures. The CLI returned status 0 for valid input. A repeat run produced the same JSON and the suite asserted the observed field below. An incomplete record returned status 2 with a diagnostic.

```json
{
  "healthy": 1
}
```

## Error handling

The CLI returns status 2 for a bad invocation or a file or input error. The terminal shows a reference result for a single snapshot.

## Troubleshooting

- Use `node --version` to check the runtime and confirm the program and sample names.
- Check the diagnostic and compare field types with the sample.
- Run the original sample again before testing a different setting.

## Limitations

The program classifies recorded probe results and runs fully offline.

## Learning outcomes

Practice designing local command-line workflows, validating records, applying service check rules, and testing baseline setup.

## Conclusion

The sample demonstrates service check in the offline classroom scenario with a repeatable baseline setup run.

[Back to the Node.js project index](../README.md)
