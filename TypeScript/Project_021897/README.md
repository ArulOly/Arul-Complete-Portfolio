# TypeScript: Automation — Automation for Portfolio Demo (Project 021897)

## Project overview

A repeatable portfolio demonstration provides the setting for this automation exercise. The code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

A repeatable portfolio demonstration needs a repeatable way to handle automation from local records. Case `DEMO-02` illustrates automation.

## Project objectives

- Resolve a local task plan and propagate prerequisite failures to dependent work.
- Process a batch in order and keep valid results when individual records fail.
- Verify the sample result, repeat the run, and observe how bad input is handled.

## Technologies used

TypeScript, Node.js, JSON, and built-in Node.js modules. The runtime requires no external packages.

## System architecture

`Local JSON → file reader → field validation → automation rules → automation response → terminal`

## Features

- Resolve a local task plan and propagate prerequisite failures to dependent work.
- The CLI processes multiple rows in one call and returns status 1 if any row fails.

## Prerequisites

Install Node.js 24 or later. Use an editor and a writable practice directory with synthetic records or data you have permission to use.

## Complete TypeScript source code

```typescript
// Local reference implementation for project 021897.
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

function objects(value: unknown, name: string): Data[] {
  return array(value, name).map((row, i) => object(row, `${name}[${i}]`));
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
  const pending = new Map<string, { needs: string[]; ready: boolean }>();
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
  const states = new Map<string, "done" | "failed" | "blocked">();
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

`evaluate()` applies the automation rules to one record. The outer records array keeps each result or error associated with its original row number. The exported `run()` function handles arguments and returns the status and output used by the command-line entry point. TypeScript uses explicit types for the command-line interface and treats parsed input as unknown until validation.

## Sample input data

Save the following synthetic data as `sample.json`.

```json
{
  "records": [
    {
      "asset": "DEMO-02",
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
    },
    {
      "asset": "DEMO-02-B",
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
  "task_states": {
    "collect": "done",
    "verify": "failed",
    "publish": "blocked"
  }
}
```

## Testing and validation

Actual checks used Node.js 24 and temporary fixtures. The CLI returned status 0 for valid input. A repeat run produced the same JSON and the suite asserted the observed field below. A mixed batch preserved its valid row and returned status 1 with a row-level error. The source also passed TypeScript strict type checking.

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

The CLI returns status 2 for a bad invocation or a file or input error. The CLI processes multiple rows in one call and returns status 1 if any row fails.

## Troubleshooting

- Use `node --version` to check the runtime and confirm the program and sample names.
- Check the diagnostic and compare field types with the sample.
- Run the original sample again before testing a different setting.

## Limitations

The task workflow simulates prerequisite outcomes using Boolean readiness values.

## Learning outcomes

Practice designing local command-line workflows, validating records, applying automation rules, and testing automation.

## Conclusion

The sample demonstrates automation in the portfolio demo scenario with a repeatable automation run.

[Back to the TypeScript project index](../README.md)
