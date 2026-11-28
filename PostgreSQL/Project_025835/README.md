# PostgreSQL: Documentation Generation — Usability Improvement for Single-Host Lab (Project 025835)

## Project overview

One simulated lab host provides the setting for this documentation generation exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The single-host lab needs a reproducible documentation generation workflow that stores and reads results in PostgreSQL.

## Project objectives

- Generate a short Markdown handover from a synthetic operating procedure.
- Expose a compact labeled result and a JSON option that reads the selected database field.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

PostgreSQL temporary tables, JSONB operators, and bound SQL parameters, Python, JSON, and Psycopg 3.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Generate a short Markdown handover from a synthetic operating procedure.
- Expose a compact labeled result and a JSON option that reads the selected database field.

## Prerequisites

Install Python 3.10 or later and use a local PostgreSQL database and a practice role allowed to create temporary tables. Use a text editor and a local practice directory.

## Complete PostgreSQL source code

```python
"""Local PostgreSQL reference implementation for project 025835."""


import argparse
import json
import os
import psycopg
import sys


METRIC = 'steps'
SCENARIO = 'Single-Host Lab'


def dumps(value):
    return json.dumps(value, ensure_ascii=False, allow_nan=False, sort_keys=True)


def label(record, key):
    value = record.get(key)
    if not isinstance(value, str) or not value.strip():
        raise ValueError(f"{key} must be non-empty text")
    return value.strip()


def array(record, key, allow_empty=False):
    value = record.get(key)
    if not isinstance(value, list) or (not value and not allow_empty):
        raise ValueError(f"{key} must be a list" + ("" if allow_empty else " with at least one item"))
    return value


def labels(record, key, allow_empty=False):
    return [label({key: item}, key) for item in array(record, key, allow_empty)]


def evaluate(record):
    """Generate a short Markdown handover from a synthetic operating procedure."""
    lines = [f"# {label(record, 'asset')}", "", label(record, "summary"), "", "## Procedure", ""]
    lines.extend(f"{index}. {step}" for index, step in enumerate(labels(record, "steps"), 1))
    return {"markdown": "\n".join(lines) + "\n", "steps": len(lines) - 6}


def mapping(record, key):
    if not isinstance(record, dict):
        raise ValueError(f"{key} must be an object")
    return record


class Store:

    def __enter__(self):
        self.database = psycopg.connect(os.environ["PORTFOLIO_POSTGRES_DSN"], connect_timeout=5, autocommit=True)
        self.cursor = self.database.cursor()
        try:
            self.cursor.execute('CREATE TEMP TABLE portfolio_rows (\n                    position INTEGER PRIMARY KEY,\n                    source JSONB NOT NULL,\n                    response JSONB NOT NULL,\n                    rejected INTEGER NOT NULL CHECK (rejected IN (0, 1))\n                )')
        except psycopg.Error:
            self.database.close()
            raise
        return self

    def __exit__(self, *_):
        self.database.close()

    def put(self, position, source, response):
        self.cursor.execute('INSERT INTO portfolio_rows VALUES (%s, %s::jsonb, %s::jsonb, %s)',
                            (position, dumps(source), dumps(response), int("error" in response)))

    def read(self, position):
        self.cursor.execute('SELECT source::text, response::text FROM portfolio_rows WHERE position = %s', (position,))
        row = self.cursor.fetchone()
        if row is None:
            raise ValueError("Stored row is missing")
        return json.loads(row[0]), json.loads(row[1])

    def metric(self, position, key):
        self.cursor.execute("SELECT response -> 'result' -> %s::text FROM portfolio_rows WHERE position = %s", (key, position))
        value = self.cursor.fetchone()[0]
        return json.loads(value) if False and value is not None else value


def load_input(path):
    with open(path, encoding="utf-8") as handle:
        value = json.load(handle)
    dumps(value)  # Reject non-finite numbers before any database write.
    return value


def evaluate_record(record):
    return {"asset": label(record, "asset"), "result": evaluate(record)}


def process(payload, store):
    record = mapping(payload, "input")
    store.put(1, record, evaluate_record(record))
    return {"scenario": SCENARIO, "asset": store.read(1)[1]["asset"],
            "field": METRIC, "metric": store.metric(1, METRIC), "status": 0}


def run(path):
    payload = load_input(path)
    with Store() as store:
        return process(payload, store)


def main():
    parser = argparse.ArgumentParser(description='Generate a short Markdown handover from a synthetic operating procedure.')
    parser.add_argument("input", help="path to the synthetic JSON fixture")
    parser.add_argument("--json", action="store_true", help="print structured output")
    args = parser.parse_args()
    try:
        response = run(args.input)
        if args.json:
            print(json.dumps(response, indent=2, ensure_ascii=False, sort_keys=True))
        else:
            print(f"{SCENARIO} | {response['asset']} | {METRIC}: {dumps(response['metric'])}")
        return response["status"]
    except (psycopg.Error, OSError, ValueError, KeyError, OverflowError, RecursionError) as exc:
        print(f"Demo failed: {exc}", file=sys.stderr)
        return 2


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

`evaluate()` performs the documentation generation calculation and validates its inputs. `Store` handles database writes and reads. The session keeps its data in a temporary PostgreSQL table with JSONB columns that closes with the connection. Expose a compact labeled result and a JSON option that reads the selected database field.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "asset": "HOST-03",
  "summary": "Review the training asset HOST-03 before handover.",
  "steps": [
    "Confirm the asset label",
    "Record the handover status"
  ]
}
```

## Execution instructions

Copy the Python block to `project.py` and the sample to `sample.json` in the same directory.

Install the driver in your virtual environment with `python -m pip install psycopg[binary]`. Set `PORTFOLIO_POSTGRES_DSN` to your local connection URI. Its format is `postgresql://USER:PASSWORD@127.0.0.1:5432/portfolio_lab`. Use your own practice credentials when authentication is required.

Run the example.

```sh
python project.py sample.json --json
```

Omit `--json` for a compact labeled result.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "steps": 2
}
```

## Testing and validation

Actual tests used PostgreSQL and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. An incomplete record was rejected.

```json
{
  "steps": 2
}
```

## Error handling

The CLI reports invalid input, database errors, and file errors with status 2. A batch with rejected records preserves its valid rows and returns status 1.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

This small example uses a temporary database store and synthetic records.

## Learning outcomes

Practice documentation generation using PostgreSQL and check a repeatable usability improvement workflow in the single-host lab scenario.

## Conclusion

The project records an observable documentation generation result for training case `HOST-03`.

[Back to the PostgreSQL project index](../README.md)
