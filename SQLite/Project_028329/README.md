# SQLite: Data Export — Reporting for Offline Classroom (Project 028329)

## Project overview

A classroom operating offline provides the setting for this data export exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The offline classroom needs a reproducible data export workflow that stores and reads results in SQLite.

## Project objectives

- Convert structured records into a predictable two-column CSV.
- Build a JSON report from the database's ordered result set and row summary.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

SQLite temporary tables, bound SQL parameters, and JSON functions, Python, JSON, and sqlite3 from the Python standard library.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Convert structured records into a predictable two-column CSV.
- Build a JSON report from the database's ordered result set and row summary.

## Prerequisites

Install Python 3.10 or later with the SQLite JSON functions available. Use a text editor and a local practice directory.

## Complete SQLite source code

```python
"""Local SQLite reference implementation for project 028329."""


import argparse
import csv
import io
import json
import sqlite3
import sys


METRIC = 'csv'
PROJECT_ID = '028329'


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


def objects(record, key):
    values = array(record, key)
    if any(not isinstance(value, dict) for value in values):
        raise ValueError(f"every item in {key} must be an object")
    return values


def evaluate(record):
    """Convert structured records into a predictable two-column CSV."""
    output = io.StringIO()
    writer = csv.writer(output, lineterminator="\n")
    writer.writerow(["code", "quantity"])
    entries = objects(record, "entries")
    for entry in entries:
        code = label(entry, "code")
        if code.startswith(("=", "+", "-", "@")):
            raise ValueError("code must begin with a letter or number")
        writer.writerow([code, integer(entry, "quantity")])
    return {"exported": len(entries), "csv": output.getvalue()}


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

    def ordered(self):
        self.cursor.execute('SELECT response FROM portfolio_rows ORDER BY position')
        return [json.loads(row[0]) for row in self.cursor.fetchall()]

    def summary(self):
        self.cursor.execute('SELECT COUNT(*), COALESCE(SUM(rejected), 0) FROM portfolio_rows')
        total, failed = self.cursor.fetchone()
        return int(total), int(failed)

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
    total, failed = store.summary()
    return {"project": PROJECT_ID, "report": store.ordered(), "total": total,
            "failed": failed, "metric": store.metric(1, METRIC), "status": 0}


def run(path):
    payload = load_input(path)
    with Store() as store:
        return process(payload, store)


def main():
    parser = argparse.ArgumentParser(description='Convert structured records into a predictable two-column CSV.')
    parser.add_argument("input", help="path to the synthetic JSON fixture")
    args = parser.parse_args()
    try:
        response = run(args.input)
        print(json.dumps(response, indent=2, ensure_ascii=False, sort_keys=True))
        return response["status"]
    except (sqlite3.Error, OSError, ValueError, KeyError, OverflowError, RecursionError) as exc:
        print(f"Demo failed: {exc}", file=sys.stderr)
        return 2


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

`evaluate()` performs the data export calculation and validates its inputs. `Store` handles database writes and reads. SQL rows live in a temporary in-memory table that closes with the connection. Build a JSON report from the database's ordered result set and row summary.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "asset": "CLASS-09",
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

Copy the Python block to `project.py` and the sample to `sample.json` in the same directory.



Run the example.

```sh
python project.py sample.json
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "csv": "code,quantity\nKIT-01,4\nKIT-02,2\n"
}
```

## Testing and validation

Actual tests used SQLite and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. An incomplete record was rejected.

```json
{
  "csv": "code,quantity\nKIT-01,4\nKIT-02,2\n"
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

Practice data export using SQLite and check a repeatable reporting workflow in the offline classroom scenario.

## Conclusion

The project records an observable data export result for training case `CLASS-09`.

[Back to the SQLite project index](../README.md)
