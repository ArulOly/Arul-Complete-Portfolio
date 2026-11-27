# JavaScript: Audit — Automation for Single-Host Lab (Project 020132)

## Project overview

One simulated lab host provides the setting for this audit exercise. The code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

One simulated lab host needs a repeatable way to handle audit from local records. Case `HOST-03` illustrates automation.

## Project objectives

- Check whether recorded asset reviews meet a cutoff and list departures.
- Process a batch in order and keep valid results when individual records fail.
- Verify the sample result, repeat the run, and observe how bad input is handled.

## Technologies used

JavaScript, Node.js, JSON, and built-in Node.js modules. The runtime requires no external packages.

## System architecture

`Local JSON → file reader → field validation → audit rules → automation response → terminal`

## Features

- Check whether recorded asset reviews meet a cutoff and list departures.
- The CLI processes multiple rows in one call and returns status 1 if any row fails.

## Prerequisites

Install Node.js 24 or later. Use an editor and a writable practice directory with synthetic records or data you have permission to use.

## Complete JavaScript source code

```javascript
// Local reference implementation for project 020132.
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

function array(value, name, allowEmpty = false) {
  if (!Array.isArray(value) || (!allowEmpty && !value.length)) {
    throw new Error(`${name} must be ${allowEmpty ? "an array" : "a non-empty array"}`);
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
  const reviewed = day(record.reviewed_on, "reviewed_on");
  const cutoff = day(record.cutoff, "cutoff");
  return { owner: text(record.owner, "owner"), reviewed_on: record.reviewed_on,
    follow_up: reviewed < cutoff };
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

`evaluate()` applies the audit rules to one record. The outer records array keeps each result or error associated with its original row number. The exported `run()` function handles arguments and returns the status and output used by the command-line entry point.

## Sample input data

Save the following synthetic data as `sample.json`.

```json
{
  "records": [
    {
      "asset": "HOST-03",
      "reviewed_on": "2026-05-03",
      "cutoff": "2026-06-01",
      "owner": "Lab Operations"
    },
    {
      "asset": "HOST-03-B",
      "reviewed_on": "2026-05-03",
      "cutoff": "2026-06-01",
      "owner": "Lab Operations"
    }
  ]
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
  "follow_up": true
}
```

## Testing and validation

Actual checks used Node.js 24 and temporary fixtures. The CLI returned status 0 for valid input. A repeat run produced the same JSON and the suite asserted the observed field below. A mixed batch preserved its valid row and returned status 1 with a row-level error.

```json
{
  "follow_up": true
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

Practice designing local command-line workflows, validating records, applying audit rules, and testing automation.

## Conclusion

The sample demonstrates audit in the single-host lab scenario with a repeatable automation run.

[Back to the JavaScript project index](../README.md)
