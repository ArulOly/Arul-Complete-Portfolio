# TypeScript: Data Export — Usability Improvement for Small Business (Project 021310)

## Project overview

Local shop operations provides the setting for this data export exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

Local shop operations needs a repeatable way to handle data export from local records. Case `SHOP-22` illustrates usability improvement.

## Project objectives

- Convert structured records into a predictable two-column CSV.
- Provide a discoverable help screen and a readable terminal summary.
- Verify the sample result, repeat the run, and observe how bad input is handled.

## Technologies used

TypeScript, Node.js, JSON, and built-in Node.js modules. The runtime requires no external packages.

## System architecture

`Local JSON → file reader → field validation → data export rules → usability improvement response → terminal`

## Features

- Convert structured records into a predictable two-column CSV.
- The operator can use --help to inspect arguments and --json for programmatic review.

## Prerequisites

Install Node.js 24 or later. Use an editor and a writable practice directory with synthetic records or data you have permission to use.

## Complete TypeScript source code

```typescript
// Local reference implementation for project 021310.
import { readFileSync } from "node:fs";
import { pathToFileURL } from "node:url";
import { parseArgs } from "node:util";

type Data = Record<string, unknown>;
type Reply = { status: number; output: string; error?: string };

const SCENARIO = "Small Business";

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

function array(value: unknown, name: string, allowEmpty = false): unknown[] {
  if (!Array.isArray(value) || (!allowEmpty && !value.length)) {
    throw new Error(`${name} must be ${allowEmpty ? "an array" : "a non-empty array"}`);
  }
  return value;
}

function objects(value: unknown, name: string): Data[] {
  return array(value, name).map((row, i) => object(row, `${name}[${i}]`));
}

function errorMessage(error: unknown): string {
  return error instanceof Error ? error.message : String(error);
}

function load(path: string): Data {
  const value: unknown = JSON.parse(readFileSync(path, "utf8"));
  return object(value, "input");
}

function evaluate(record: Data): Data {
  const entries = objects(record.entries, "entries");
  const lines = entries.map(entry => {
    const code = text(entry.code, "code");
    if (!/^[A-Za-z0-9][A-Za-z0-9_-]*$/.test(code)) {
      throw new Error("Use an alphanumeric code with optional hyphens or underscores");
    }
    return `${code},${integer(entry.quantity, "quantity")}`;
  });
  return { exported: entries.length, csv: ["code,quantity", ...lines, ""].join("\n") };
}

function reply(value: Data, status = 0): Reply {
  return { status, output: JSON.stringify(value, null, 2) };
}

export function run(args: string[]): Reply {
  const usage = "Usage: node project.mts sample.json [--json]";
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

`evaluate()` applies the data export rules to one record. The CLI converts each result field into a labeled line or outputs JSON with a flag. The exported `run()` function handles arguments and returns the status and output used by the command-line entry point. TypeScript uses explicit types for the command-line interface and treats parsed input as unknown until validation.

## Sample input data

Save the following synthetic data as `sample.json`.

```json
{
  "asset": "SHOP-22",
  "entries": [
    {
      "code": "KIT-01",
      "quantity": 4
    },
    {
      "code": "KIT-02",
      "quantity": 2
    }
  ]
}
```

## Execution instructions

1. Copy the source block into `project.mts`.
2. Save the sample input alongside it.
3. Run this command in that directory.

Omit `--json` for labeled text. Pass `--help` to list the arguments.

```sh
node project.mts sample.json --json
```

## Example output

Illustrative excerpt. The CLI response includes additional fields.

```json
{
  "csv": "code,quantity\nKIT-01,4\nKIT-02,2\n"
}
```

## Testing and validation

Actual checks used Node.js 24 and temporary fixtures. The CLI returned status 0 for valid input. A repeat run produced the same JSON and the suite asserted the observed field below. An incomplete record returned status 2. Help documented the JSON flag, and the default view labeled its fields. The source also passed TypeScript strict type checking.

```json
{
  "csv": "code,quantity\nKIT-01,4\nKIT-02,2\n"
}
```

## Error handling

The CLI returns status 2 for a bad invocation or a file or input error. The operator can use --help to inspect arguments and --json for programmatic review.

## Troubleshooting

- Use `node --version` to check the runtime and confirm the program and sample names.
- Check the diagnostic and compare field types with the sample.
- Run the original sample again before testing a different setting.

## Limitations

The export accepts codes made of letters, digits, hyphens, and underscores.

## Learning outcomes

Practice designing local command-line workflows, validating records, applying data export rules, and testing usability improvement.

## Conclusion

The sample demonstrates data export in the small business scenario with a repeatable usability improvement run.

[Back to the TypeScript project index](../README.md)
