# Node.js: Automation — Reporting for Small Business (Project 024859)

## Project overview

Local shop operations provides the setting for this automation exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

Local shop operations needs a repeatable way to handle automation from local records. Case `SHOP-22` illustrates reporting.

## Project objectives

- Resolve a local task plan and propagate prerequisite failures to dependent work.
- Create a portable JSON report without replacing an existing file.
- Verify the sample result, repeat the run, and observe how bad input is handled.

## Technologies used

Node.js, Node.js, JSON, and built-in Node.js modules. The runtime requires no external packages.

## System architecture

`Local JSON → file reader → field validation → automation rules → reporting response → terminal`

## Features

- Resolve a local task plan and propagate prerequisite failures to dependent work.
- The terminal and optional UTF-8 report file carry the same structured result.

## Prerequisites

Install Node.js 24 or later. Use an editor and a writable practice directory with synthetic records or data you have permission to use.

## Complete Node.js source code

```javascript
// Local reference implementation for project 024859.
import { readFileSync, writeFileSync } from "node:fs";
import { pathToFileURL } from "node:url";
import { parseArgs } from "node:util";

const PROJECT_ID = "024859";

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

function objects(value, name) {
  return array(value, name).map((row, i) => object(row, `${name}[${i}]`));
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
  const pending = new Map();
  for (const task of objects(record.tasks, "tasks")) {
    const name = text(task.name, "name");
    if (pending.has(name) || typeof task.ready !== "boolean") {
      throw new Error("Each task needs a unique name and Boolean ready flag");
    }
    pending.set(name, { needs: strings(task.needs, "needs", true), ready: task.ready });
  }
  for (const task of pending.values()) {
    if (task.needs.some(name => !pending.has(name))) throw new Error("Missing prerequisite");
  }
  const states = new Map();
  while (pending.size) {
    const eligible = [...pending].filter(([, task]) => task.needs.every(name => states.has(name)));
    if (!eligible.length) throw new Error("Task dependencies contain a cycle");
    for (const [name, task] of eligible) {
      const blocked = task.needs.some(need => states.get(need) !== "done");
      states.set(name, blocked ? "blocked" : task.ready ? "done" : "failed");
      pending.delete(name);
    }
  }
  return { task_states: Object.fromEntries(states) };
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

`evaluate()` applies the automation rules to one record. The workflow builds its report before opening the destination in exclusive creation mode. The exported `run()` function handles arguments and returns the status and output used by the command-line entry point.

## Sample input data

Save the following synthetic data as `sample.json`.

```json
{
  "asset": "SHOP-22",
  "tasks": [
    {
      "name": "collect",
      "ready": true,
      "needs": []
    },
    {
      "name": "verify",
      "ready": false,
      "needs": [
        "collect"
      ]
    },
    {
      "name": "publish",
      "ready": true,
      "needs": [
        "verify"
      ]
    }
  ]
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
  "task_states": {
    "collect": "done",
    "verify": "failed",
    "publish": "blocked"
  }
}
```

## Testing and validation

Actual checks used Node.js 24 and temporary fixtures. The CLI returned status 0 for valid input. A repeat run produced the same JSON and the suite asserted the observed field below. An incomplete record returned status 2. The report matched terminal JSON, and an attempt to overwrite it was rejected.

```json
{
  "task_states": {
    "collect": "done",
    "verify": "failed",
    "publish": "blocked"
  }
}
```

## Error handling

The CLI returns status 2 for a bad invocation or a file or input error. The terminal and optional UTF-8 report file carry the same structured result.

## Troubleshooting

- Use `node --version` to check the runtime and confirm the program and sample names.
- Check the diagnostic and compare field types with the sample.
- Run the original sample again before testing a different setting.

## Limitations

The task workflow simulates prerequisite outcomes using Boolean readiness values.

## Learning outcomes

Practice designing local command-line workflows, validating records, applying automation rules, and testing reporting.

## Conclusion

The sample demonstrates automation in the small business scenario with a repeatable reporting run.

[Back to the Node.js project index](../README.md)
