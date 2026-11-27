# Node.js: Scheduled Reporting — Reporting for Branch Office Simulator (Project 024474)

## Project overview

A simulated branch office provides the setting for this scheduled reporting exercise. The code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

A simulated branch office needs a repeatable way to handle scheduled reporting from local records. Case `BRANCH-08` illustrates reporting.

## Project objectives

- Calculate the next due date from an explicit reporting interval and reference date.
- Create a portable JSON report without replacing an existing file.
- Verify the sample result, repeat the run, and observe how bad input is handled.

## Technologies used

Node.js, Node.js, JSON, and built-in Node.js modules. The runtime requires no external packages.

## System architecture

`Local JSON → file reader → field validation → scheduled reporting rules → reporting response → terminal`

## Features

- Calculate the next due date from an explicit reporting interval and reference date.
- The terminal and optional UTF-8 report file carry the same structured result.

## Prerequisites

Install Node.js 24 or later. Use an editor and a writable practice directory with synthetic records or data you have permission to use.

## Complete Node.js source code

```javascript
// Local reference implementation for project 024474.
import { readFileSync, writeFileSync } from "node:fs";
import { pathToFileURL } from "node:url";
import { parseArgs } from "node:util";

const PROJECT_ID = "024474";

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

function day(value, name) {
  const date = text(value, name);
  const epoch = Date.parse(date + "T00:00:00.000Z");
  if (!/^\d{4}-\d{2}-\d{2}$/.test(date) || !Number.isFinite(epoch)
    || new Date(epoch).toISOString().slice(0, 10) !== date) {
    throw new Error(`${name} must be a valid YYYY-MM-DD date`);
  }
  return epoch;
}

function errorMessage(error) {
  return error instanceof Error ? error.message : String(error);
}

function load(path) {
  const value = JSON.parse(readFileSync(path, "utf8"));
  return object(value, "input");
}

function evaluate(record) {
  const previous = day(record.previous_report, "previous_report");
  const asOf = day(record.as_of, "as_of");
  const interval = integer(record.interval_days, "interval_days", 1, 365);
  if (asOf < previous) throw new Error("Reference date precedes previous report");
  const dayMs = 86_400_000;
  const due = previous + interval * dayMs;
  return { due_on: new Date(due).toISOString().split("T")[0],
    due: asOf >= due, days_overdue: Math.max(0, (asOf - due) / dayMs) };
}

function reply(value, status = 0) {
  return { status, output: JSON.stringify(value, null, 2) };
}

export function run(args) {
  const usage = "Usage: node project.mjs sample.json [--output report.json]";
  try {
    const { values, positionals } = parseArgs({
      args, allowPositionals: true, options: { output: { type: "string" }, help: { type: "boolean" } },
    });
    if (values.help) return { status: 0, output: usage };
    if (positionals.length !== 1) throw new Error(usage);
    const [input] = positionals;

    const record = load(input);
    const report = reply({ project: PROJECT_ID, asset: text(record.asset, "asset"), result: evaluate(record) });
    if (values.output !== undefined) {
      writeFileSync(values.output, report.output + "\n", { encoding: "utf8", flag: "wx" });
    }
    return report;
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

`evaluate()` applies the scheduled reporting rules to one record. The workflow builds its report before opening the destination in exclusive creation mode. The exported `run()` function handles arguments and returns the status and output used by the command-line entry point.

## Sample input data

Save the following synthetic data as `sample.json`.

```json
{
  "asset": "BRANCH-08",
  "previous_report": "2026-06-01",
  "as_of": "2026-06-16",
  "interval_days": 7
}
```

## Execution instructions

1. Copy the source block into `project.mjs`.
2. Save the sample input alongside it.
3. Run this command in that directory.

Use a new report filename on each write. Omit `--output` to print the report only.

```sh
node project.mjs sample.json --output report.json
```

## Example output

Illustrative excerpt. The CLI response includes additional fields.

```json
{
  "days_overdue": 8
}
```

## Testing and validation

Actual checks used Node.js 24 and temporary fixtures. The CLI returned status 0 for valid input. A repeat run produced the same JSON and the suite asserted the observed field below. An incomplete record returned status 2. The report matched terminal JSON, and an attempt to overwrite it was rejected.

```json
{
  "days_overdue": 8
}
```

## Error handling

The CLI returns status 2 for a bad invocation or a file or input error. The terminal and optional UTF-8 report file carry the same structured result.

## Troubleshooting

- Use `node --version` to check the runtime and confirm the program and sample names.
- Check the diagnostic and compare field types with the sample.
- Run the original sample again before testing a different setting.

## Limitations

The program calculates due dates from supplied UTC calendar dates. Invoke it manually with a reference date.

## Learning outcomes

Practice designing local command-line workflows, validating records, applying scheduled reporting rules, and testing reporting.

## Conclusion

The sample demonstrates scheduled reporting in the branch office simulator scenario with a repeatable reporting run.

[Back to the Node.js project index](../README.md)
