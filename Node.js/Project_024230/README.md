# Node.js: Backup Verification — Usability Improvement for Offline Classroom (Project 024230)

## Project overview

A classroom operating offline provides the setting for this backup verification exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

A classroom operating offline needs a repeatable way to handle backup verification from local records. Case `CLASS-09` illustrates usability improvement.

## Project objectives

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Provide a discoverable help screen and a readable terminal summary.
- Verify the sample result, repeat the run, and observe how bad input is handled.

## Technologies used

Node.js, Node.js, JSON, and built-in Node.js modules. The runtime requires no external packages.

## System architecture

`Local JSON → file reader → field validation → backup verification rules → usability improvement response → terminal`

## Features

- Compare a synthetic backup payload against an expected SHA-256 digest.
- The operator can use --help to inspect arguments and --json for programmatic review.

## Prerequisites

Install Node.js 24 or later. Use an editor and a writable practice directory with synthetic records or data you have permission to use.

## Complete Node.js source code

```javascript
// Local reference implementation for project 024230.
import { createHash } from "node:crypto";
import { readFileSync } from "node:fs";
import { pathToFileURL } from "node:url";
import { parseArgs } from "node:util";

const SCENARIO = "Offline Classroom";

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
  const usage = "Usage: node project.mjs sample.json [--json]";
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

`evaluate()` applies the backup verification rules to one record. The CLI converts each result field into a labeled line or outputs JSON with a flag. The exported `run()` function handles arguments and returns the status and output used by the command-line entry point.

## Sample input data

Save the following synthetic data as `sample.json`.

```json
{
  "asset": "CLASS-09",
  "payload": "CLASS-09 inventory backup, revision 1",
  "sha256": "642e2ecb857141211b0b4db5863e9df6fb47236fe3704fc1bd0791cb1108b3e7"
}
```

## Execution instructions

1. Copy the source block into `project.mjs`.
2. Save the sample input alongside it.
3. Run this command in that directory.

Omit `--json` for labeled text. Pass `--help` to list the arguments.

```sh
node project.mjs sample.json --json
```

## Example output

Illustrative excerpt. The CLI response includes additional fields.

```json
{
  "verified": true
}
```

## Testing and validation

Actual checks used Node.js 24 and temporary fixtures. The CLI returned status 0 for valid input. A repeat run produced the same JSON and the suite asserted the observed field below. An incomplete record returned status 2. Help documented the JSON flag, and the default view labeled its fields.

```json
{
  "verified": true
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

Practice designing local command-line workflows, validating records, applying backup verification rules, and testing usability improvement.

## Conclusion

The sample demonstrates backup verification in the offline classroom scenario with a repeatable usability improvement run.

[Back to the Node.js project index](../README.md)
