# JavaScript: Backup Verification — Automation for Home Development (Project 020212)

## Project overview

A home development workspace provides the setting for this backup verification exercise. The code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

A home development workspace needs a repeatable way to handle backup verification from local records. Case `DEV-05` illustrates automation.

## Project objectives

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Process a batch in order and keep valid results when individual records fail.
- Verify the sample result, repeat the run, and observe how bad input is handled.

## Technologies used

JavaScript, Node.js, JSON, and built-in Node.js modules. The runtime requires no external packages.

## System architecture

`Local JSON → file reader → field validation → backup verification rules → automation response → terminal`

## Features

- Compare a synthetic backup payload against an expected SHA-256 digest.
- The CLI processes multiple rows in one call and returns status 1 if any row fails.

## Prerequisites

Install Node.js 24 or later. Use an editor and a writable practice directory with synthetic records or data you have permission to use.

## Complete JavaScript source code

```javascript
// Local reference implementation for project 020212.
import { createHash } from "node:crypto";
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

function errorMessage(error) {
  return error instanceof Error ? error.message : String(error);
}

function load(path) {
  const value = JSON.parse(readFileSync(path, "utf8"));
  return object(value, "input");
}

function evaluate(record) {
  const expected = text(record.sha256, "sha256").toLowerCase();
  if (!/^[a-f0-9]{64}$/.test(expected)) throw new Error("Invalid SHA-256 digest");
  const payload = text(record.payload, "payload", false);
  const sha256 = createHash("sha256").update(payload, "utf8").digest("hex");
  return { bytes: Buffer.byteLength(payload), sha256, verified: sha256 === expected };
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

`evaluate()` applies the backup verification rules to one record. The outer records array keeps each result or error associated with its original row number. The exported `run()` function handles arguments and returns the status and output used by the command-line entry point.

## Sample input data

Save the following synthetic data as `sample.json`.

```json
{
  "records": [
    {
      "asset": "DEV-05",
      "payload": "DEV-05 inventory backup, revision 1",
      "sha256": "cd6f823c4bb5acc34221cfe842b885475f9bc477844e40629a42b2b930669654"
    },
    {
      "asset": "DEV-05-B",
      "payload": "DEV-05 inventory backup, revision 1",
      "sha256": "cd6f823c4bb5acc34221cfe842b885475f9bc477844e40629a42b2b930669654"
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
  "verified": true
}
```

## Testing and validation

Actual checks used Node.js 24 and temporary fixtures. The CLI returned status 0 for valid input. A repeat run produced the same JSON and the suite asserted the observed field below. A mixed batch preserved its valid row and returned status 1 with a row-level error.

```json
{
  "verified": true
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

Practice designing local command-line workflows, validating records, applying backup verification rules, and testing automation.

## Conclusion

The sample demonstrates backup verification in the home development scenario with a repeatable automation run.

[Back to the JavaScript project index](../README.md)
