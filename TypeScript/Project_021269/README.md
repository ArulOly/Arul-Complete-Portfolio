# TypeScript: Data Import — Reporting for Public Dataset (Project 021269)

## Project overview

A synthetic public-data exercise provides the setting for this data import exercise. The code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

A synthetic public-data exercise needs a repeatable way to handle data import from local records. Case `DATA-12` illustrates reporting.

## Project objectives

- Parse a small CSV with a fixed header and turn rows into normalized records.
- Create a portable JSON report without replacing an existing file.
- Verify the sample result, repeat the run, and observe how bad input is handled.

## Technologies used

TypeScript, Node.js, JSON, and built-in Node.js modules. The runtime requires no external packages.

## System architecture

`Local JSON → file reader → field validation → data import rules → reporting response → terminal`

## Features

- Parse a small CSV with a fixed header and turn rows into normalized records.
- The terminal and optional UTF-8 report file carry the same structured result.

## Prerequisites

Install Node.js 24 or later. Use an editor and a writable practice directory with synthetic records or data you have permission to use.

## Complete TypeScript source code

```typescript
// Local reference implementation for project 021269.
import { readFileSync, writeFileSync } from "node:fs";
import { pathToFileURL } from "node:url";
import { parseArgs } from "node:util";

type Data = Record<string, unknown>;
type Reply = { status: number; output: string; error?: string };

const PROJECT_ID = "021269";

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

function integer(value: unknown, name: string, min = 0, max = 1_000_000_000): number {
  if (typeof value !== "number" || !Number.isSafeInteger(value) || value < min || value > max) {
    throw new Error(`${name} must be an integer from ${min} to ${max}`);
  }
  return value;
}

function errorMessage(error: unknown): string {
  return error instanceof Error ? error.message : String(error);
}

function load(path: string): Data {
  const value: unknown = JSON.parse(readFileSync(path, "utf8"));
  return object(value, "input");
}

function evaluate(record: Data): Data {
  const [header, ...lines] = text(record.csv_data, "csv_data").split(/\r?\n/);
  if (header !== "code,quantity" || !lines.length) throw new Error("Expected code,quantity and data rows");
  const codes = new Set();
  const rows = lines.map((line, index) => {
    const match = /^([A-Za-z0-9_-]+),(\d+)$/.exec(line);
    if (!match) throw new Error(`Invalid simple CSV row ${index + 2}`);
    const [, code, quantity] = match;
    if (codes.has(code)) throw new Error(`Duplicate code ${code}`);
    codes.add(code);
    return { code, quantity: integer(Number(quantity), "quantity") };
  });
  return { imported: rows.length, rows };
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

`evaluate()` applies the data import rules to one record. The workflow builds its report before opening the destination in exclusive creation mode. The exported `run()` function handles arguments and returns the status and output used by the command-line entry point. TypeScript uses explicit types for the command-line interface and treats parsed input as unknown until validation.

## Sample input data

Save the following synthetic data as `sample.json`.

```json
{
  "asset": "DATA-12",
  "csv_data": "code,quantity\nKIT-01,4\nKIT-02,2\n"
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
  "imported": 2
}
```

## Testing and validation

Actual checks used Node.js 24 and temporary fixtures. The CLI returned status 0 for valid input. A repeat run produced the same JSON and the suite asserted the observed field below. An incomplete record returned status 2. The report matched terminal JSON, and an attempt to overwrite it was rejected. The source also passed TypeScript strict type checking.

```json
{
  "imported": 2
}
```

## Error handling

The CLI returns status 2 for a bad invocation or a file or input error. The terminal and optional UTF-8 report file carry the same structured result.

## Troubleshooting

- Use `node --version` to check the runtime and confirm the program and sample names.
- Check the diagnostic and compare field types with the sample.
- Run the original sample again before testing a different setting.

## Limitations

The CSV example supports a fixed header and unquoted codes. It expects one record per line.

## Learning outcomes

Practice designing local command-line workflows, validating records, applying data import rules, and testing reporting.

## Conclusion

The sample demonstrates data import in the public dataset scenario with a repeatable reporting run.

[Back to the TypeScript project index](../README.md)
