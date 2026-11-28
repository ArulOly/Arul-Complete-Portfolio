# TypeScript: Documentation Generation — Reporting for Home Development (Project 021814)

## Project overview

A home development workspace provides the setting for this documentation generation exercise. The code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

A home development workspace needs a repeatable way to handle documentation generation from local records. Case `DEV-05` illustrates reporting.

## Project objectives

- Generate a short Markdown handover from a synthetic operating procedure.
- Create a portable JSON report without replacing an existing file.
- Verify the sample result, repeat the run, and observe how bad input is handled.

## Technologies used

TypeScript, Node.js, JSON, and built-in Node.js modules. The runtime requires no external packages.

## System architecture

`Local JSON → file reader → field validation → documentation generation rules → reporting response → terminal`

## Features

- Generate a short Markdown handover from a synthetic operating procedure.
- The terminal and optional UTF-8 report file carry the same structured result.

## Prerequisites

Install Node.js 24 or later. Use an editor and a writable practice directory with synthetic records or data you have permission to use.

## Complete TypeScript source code

```typescript
// Local reference implementation for project 021814.
import { readFileSync, writeFileSync } from "node:fs";
import { pathToFileURL } from "node:url";
import { parseArgs } from "node:util";

type Data = Record<string, unknown>;
type Reply = { status: number; output: string; error?: string };

const PROJECT_ID = "021814";

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
  const steps = strings(record.steps, "steps");
  const asset = text(record.asset, "asset");
  const summary = text(record.summary, "summary");
  const lines = [`# ${asset}`, "", summary, "", "## Procedure", "",
    ...steps.map((step, index) => `${index + 1}. ${step}`), ""];
  return { markdown: lines.join("\n"), steps: steps.length };
}

function reply(value: Data, status = 0): Reply {
  return { status, output: JSON.stringify(value, null, 2) };
}

export function run(args: string[]): Reply {
  const usage = "Usage: node project.mts sample.json [--output report.json]";
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

`evaluate()` applies the documentation generation rules to one record. The workflow builds its report before opening the destination in exclusive creation mode. The exported `run()` function handles arguments and returns the status and output used by the command-line entry point. TypeScript uses explicit types for the command-line interface and treats parsed input as unknown until validation.

## Sample input data

Save the following synthetic data as `sample.json`.

```json
{
  "asset": "DEV-05",
  "summary": "Review training asset DEV-05 before handover.",
  "steps": [
    "Confirm the asset label",
    "Record the handover status"
  ]
}
```

## Execution instructions

1. Copy the source block into `project.mts`.
2. Save the sample input alongside it.
3. Run this command in that directory.

Use a new report filename on each write. Omit `--output` to print the report only.

```sh
node project.mts sample.json --output report.json
```

## Example output

Illustrative excerpt. The CLI response includes additional fields.

```json
{
  "steps": 2
}
```

## Testing and validation

Actual checks used Node.js 24 and temporary fixtures. The CLI returned status 0 for valid input. A repeat run produced the same JSON and the suite asserted the observed field below. An incomplete record returned status 2. The report matched terminal JSON, and an attempt to overwrite it was rejected. The source also passed TypeScript strict type checking.

```json
{
  "steps": 2
}
```

## Error handling

The CLI returns status 2 for a bad invocation or a file or input error. The terminal and optional UTF-8 report file carry the same structured result.

## Troubleshooting

- Use `node --version` to check the runtime and confirm the program and sample names.
- Check the diagnostic and compare field types with the sample.
- Run the original sample again before testing a different setting.

## Limitations

The exercise uses local synthetic records and covers the workflow described in the sample.

## Learning outcomes

Practice designing local command-line workflows, validating records, applying documentation generation rules, and testing reporting.

## Conclusion

The sample demonstrates documentation generation in the home development scenario with a repeatable reporting run.

[Back to the TypeScript project index](../README.md)
