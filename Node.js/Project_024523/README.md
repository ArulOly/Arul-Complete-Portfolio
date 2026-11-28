# Node.js: Access Review — Recovery for Branch Office Simulator (Project 024523)

## Project overview

A simulated branch office provides the setting for this access review exercise. The code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

A simulated branch office needs a repeatable way to handle access review from local records. Case `BRANCH-08` illustrates recovery.

## Project objectives

- Compare synthetic active grants with an approved role policy.
- Try a fallback input snapshot when the primary input is missing or invalid.
- Verify the sample result, repeat the run, and observe how bad input is handled.

## Technologies used

Node.js, Node.js, JSON, and built-in Node.js modules. The runtime requires no external packages.

## System architecture

`Local JSON → file reader → field validation → access review rules → recovery response → terminal`

## Features

- Compare synthetic active grants with an approved role policy.
- The output identifies the input role and gives the reason when the fallback is used.

## Prerequisites

Install Node.js 24 or later. Use an editor and a writable practice directory with synthetic records or data you have permission to use.

## Complete Node.js source code

```javascript
// Local reference implementation for project 024523.
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
  const approved = new Set(strings(record.approved, "approved", true));
  const grants = new Set(strings(record.grants, "grants", true));
  const excess = [...grants].filter(grant => !approved.has(grant)).sort();
  const missing = [...approved].filter(grant => !grants.has(grant)).sort();
  return { excess_grants: excess, missing_grants: missing, matches_policy: !excess.length && !missing.length };
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

`evaluate()` applies the access review rules to one record. The primary file is evaluated first. An input error triggers a fresh read of the fallback. The exported `run()` function handles arguments and returns the status and output used by the command-line entry point.

## Sample input data

Save the following synthetic data as `sample.json`.

```json
{
  "asset": "BRANCH-08",
  "approved": [
    "read",
    "submit"
  ],
  "grants": [
    "read",
    "admin"
  ]
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
  "excess_grants": [
    "admin"
  ]
}
```

## Testing and validation

Actual checks used Node.js 24 and temporary fixtures. The CLI returned status 0 for valid input. A repeat run produced the same JSON and the suite asserted the observed field below. An incomplete primary input used the fallback. An unavailable fallback returned status 2.

```json
{
  "excess_grants": [
    "admin"
  ]
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

Practice designing local command-line workflows, validating records, applying access review rules, and testing recovery.

## Conclusion

The sample demonstrates access review in the branch office simulator scenario with a repeatable recovery run.

[Back to the Node.js project index](../README.md)
