# Node.js: Log Analysis — Usability Improvement for Home Development (Project 024165)

## Project overview

A home development workspace provides the setting for this log analysis exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

A home development workspace needs a repeatable way to handle log analysis from local records. Case `DEV-05` illustrates usability improvement.

## Project objectives

- Count event levels in a synthetic log and identify the most frequent error.
- Provide a discoverable help screen and a readable terminal summary.
- Verify the sample result, repeat the run, and observe how bad input is handled.

## Technologies used

Node.js, Node.js, JSON, and built-in Node.js modules. The runtime requires no external packages.

## System architecture

`Local JSON → file reader → field validation → log analysis rules → usability improvement response → terminal`

## Features

- Count event levels in a synthetic log and identify the most frequent error.
- The operator can use --help to inspect arguments and --json for programmatic review.

## Prerequisites

Install Node.js 24 or later. Use an editor and a writable practice directory with synthetic records or data you have permission to use.

## Complete Node.js source code

```javascript
// Local reference implementation for project 024165.
import { readFileSync } from "node:fs";
import { pathToFileURL } from "node:url";
import { parseArgs } from "node:util";

const SCENARIO = "Home Development";

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

function strings(value, name, allowEmpty = false) {
  return array(value, name, allowEmpty).map((item, i) => text(item, `${name}[${i}]`));
}

function errorMessage(error) {
  return error instanceof Error ? error.message : String(error);
}

function load(path) {
  const value = JSON.parse(readFileSync(path, "utf8"));
  return object(value, "input");
}

function evaluate(record) {
  const levels = new Map();
  const errors = new Map();
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

`evaluate()` applies the log analysis rules to one record. The CLI converts each result field into a labeled line or outputs JSON with a flag. The exported `run()` function handles arguments and returns the status and output used by the command-line entry point.

## Sample input data

Save the following synthetic data as `sample.json`.

```json
{
  "asset": "DEV-05",
  "events": [
    "INFO | Started",
    "ERROR | Timeout",
    "ERROR | Timeout",
    "WARN | Retrying"
  ]
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
  "top_error": "Timeout"
}
```

## Testing and validation

Actual checks used Node.js 24 and temporary fixtures. The CLI returned status 0 for valid input. A repeat run produced the same JSON and the suite asserted the observed field below. An incomplete record returned status 2. Help documented the JSON flag, and the default view labeled its fields.

```json
{
  "top_error": "Timeout"
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

Practice designing local command-line workflows, validating records, applying log analysis rules, and testing usability improvement.

## Conclusion

The sample demonstrates log analysis in the home development scenario with a repeatable usability improvement run.

[Back to the Node.js project index](../README.md)
