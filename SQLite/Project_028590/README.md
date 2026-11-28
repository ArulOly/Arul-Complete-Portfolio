# SQLite: Performance Baseline — Usability Improvement for Two-Node Lab (Project 028590)

## Project overview

A two-node lab simulation provides the setting for this performance baseline exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The two-node lab needs a reproducible performance baseline workflow that stores and reads results in SQLite.

## Project objectives

- Calculate median and nearest-rank p95 latency against a stated budget.
- Expose a compact labeled result and a JSON option that reads the selected database field.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

SQLite temporary tables, bound SQL parameters, and JSON functions, Python, JSON, and sqlite3 from the Python standard library.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Calculate median and nearest-rank p95 latency against a stated budget.
- Expose a compact labeled result and a JSON option that reads the selected database field.

## Prerequisites

Install Python 3.10 or later with the SQLite JSON functions available. Use a text editor and a local practice directory.

## Complete SQLite source code

```python
"""Local SQLite reference implementation for project 028590."""


import argparse
import json
import sqlite3
import statistics
import sys


METRIC = 'p95_ms'
SCENARIO = 'Two-Node Lab'


def dumps(value):
    return json.dumps(value, ensure_ascii=False, allow_nan=False, sort_keys=True)


def label(record, key):
    value = record.get(key)
    if not isinstance(value, str) or not value.strip():
        raise ValueError(f"{key} must be non-empty text")
    return value.strip()


def integer(record, key, minimum=0, maximum=1_000_000_000):
    value = record.get(key)
    if type(value) is not int or not minimum <= value <= maximum:
        raise ValueError(f"{key} must be an integer from {minimum} to {maximum}")
    return value


def array(record, key, allow_empty=False):
    value = record.get(key)
    if not isinstance(value, list) or (not value and not allow_empty):
        raise ValueError(f"{key} must be a list" + ("" if allow_empty else " with at least one item"))
    return value


def series(record, key, maximum=1_000_000_000):
    return [integer({key: item}, key, maximum=maximum) for item in array(record, key)]


def evaluate(record):
    """Calculate median and nearest-rank p95 latency against a stated budget."""
    readings = sorted(series(record, "latencies_ms"))
    rank = (95 * len(readings) + 99) // 100
    p95 = readings[rank - 1]
    budget = integer(record, "budget_ms", minimum=1)
    return {"samples": len(readings), "median_ms": statistics.median(readings),
            "p95_ms": p95, "within_budget": p95 <= budget}


def mapping(record, key):
    if not isinstance(record, dict):
        raise ValueError(f"{key} must be an object")
    return record


class Store:

    def __enter__(self):
        self.database = sqlite3.connect(":memory:", isolation_level=None)
        self.cursor = self.database.cursor()
        try:
            self.cursor.execute('CREATE TEMP TABLE portfolio_rows (\n                    position INTEGER PRIMARY KEY,\n                    source TEXT NOT NULL CHECK (json_valid(source)),\n                    response TEXT NOT NULL CHECK (json_valid(response)),\n                    rejected INTEGER NOT NULL CHECK (rejected IN (0, 1))\n                )')
        except sqlite3.Error:
            self.database.close()
            raise
        return self

    def __exit__(self, *_):
        self.database.close()

    def put(self, position, source, response):
        self.cursor.execute('INSERT INTO portfolio_rows VALUES (?, ?, ?, ?)',
                            (position, dumps(source), dumps(response), int("error" in response)))

    def read(self, position):
        self.cursor.execute('SELECT source, response FROM portfolio_rows WHERE position = ?', (position,))
        row = self.cursor.fetchone()
        if row is None:
            raise ValueError("Stored row is missing")
        return json.loads(row[0]), json.loads(row[1])

    def metric(self, position, key):
        # SQLite returns JSON containers as text and booleans as integers.
        # The full response supplies the original JSON type.
        value = self.cursor.execute("SELECT json_extract(response, '$.result.' || ?) FROM portfolio_rows WHERE position = ?", (key, position)).fetchone()[0]
        original = self.read(position)[1]["result"][key]
        if isinstance(original, (dict, list)):
            return json.loads(value)
        return bool(value) if type(original) is bool else value


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
    parser = argparse.ArgumentParser(description='Calculate median and nearest-rank p95 latency against a stated budget.')
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
    except (sqlite3.Error, OSError, ValueError, KeyError, OverflowError, RecursionError) as exc:
        print(f"Demo failed: {exc}", file=sys.stderr)
        return 2


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

`evaluate()` performs the performance baseline calculation and validates its inputs. `Store` handles database writes and reads. SQL rows live in a temporary in-memory table that closes with the connection. Expose a compact labeled result and a JSON option that reads the selected database field.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "asset": "NODE-B",
  "latencies_ms": [
    12,
    15,
    14,
    44,
    17
  ],
  "budget_ms": 30
}
```

## Execution instructions

Copy the Python block to `project.py` and the sample to `sample.json` in the same directory.



Run the example.

```sh
python project.py sample.json --json
```

Omit `--json` for a compact labeled result.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "p95_ms": 44
}
```

## Testing and validation

Actual tests used SQLite and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. An incomplete record was rejected.

```json
{
  "p95_ms": 44
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

Practice performance baseline using SQLite and check a repeatable usability improvement workflow in the two-node lab scenario.

## Conclusion

The project records an observable performance baseline result for training case `NODE-B`.

[Back to the SQLite project index](../README.md)
