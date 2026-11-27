# SQLite: Backup Verification — Recovery for Help Desk (Project 028243)

## Project overview

A help desk training queue provides the setting for this backup verification exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The help desk needs a reproducible backup verification workflow that stores and reads results in SQLite.

## Project objectives

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Save a fallback snapshot and read it back if the primary record fails validation.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

SQLite temporary tables, bound SQL parameters, and JSON functions, Python, JSON, and sqlite3 from the Python standard library.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Save a fallback snapshot and read it back if the primary record fails validation.

## Prerequisites

Install Python 3.10 or later with the SQLite JSON functions available. Use a text editor and a local practice directory.

## Complete SQLite source code

```python
"""Local SQLite reference implementation for project 028243."""


import argparse
import hashlib
import json
import sqlite3
import sys


METRIC = 'verified'


def dumps(value):
    return json.dumps(value, ensure_ascii=False, allow_nan=False, sort_keys=True)


def label(record, key):
    value = record.get(key)
    if not isinstance(value, str) or not value.strip():
        raise ValueError(f"{key} must be non-empty text")
    return value.strip()


def evaluate(record):
    """Compare a synthetic backup payload against an expected SHA-256 digest."""
    expected = label(record, "sha256").lower()
    if len(expected) != 64 or any(char not in "0123456789abcdef" for char in expected):
        raise ValueError("sha256 must contain exactly 64 hexadecimal characters")
    actual = hashlib.sha256(label(record, "payload").encode("utf-8")).hexdigest()
    return {"bytes": len(label(record, "payload").encode("utf-8")),
            "sha256": actual, "verified": actual == expected}


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
    envelope = mapping(payload, "input")
    fallback = mapping(envelope.get("fallback"), "fallback")
    store.put(1, fallback, {"snapshot": True})
    try:
        record = mapping(envelope.get("primary"), "primary")
        result = evaluate_record(record)
        source = "primary"
        diagnostic = None
    except (ValueError, OverflowError) as exc:
        record, _ = store.read(1)
        result = evaluate_record(record)
        source = "fallback"
        diagnostic = str(exc)
    store.put(2, record, result)
    return {"source": source, "primary_error": diagnostic,
            "row": store.read(2)[1], "metric": store.metric(2, METRIC), "status": 0}


def run(path):
    payload = load_input(path)
    with Store() as store:
        return process(payload, store)


def main():
    parser = argparse.ArgumentParser(description='Compare a synthetic backup payload against an expected SHA-256 digest.')
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

`evaluate()` performs the backup verification calculation and validates its inputs. `Store` handles database writes and reads. SQL rows live in a temporary in-memory table that closes with the connection. Save a fallback snapshot and read it back if the primary record fails validation.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "primary": {
    "asset": "DESK-14",
    "payload": "Backup manifest for DESK-14, revision 1",
    "sha256": "5a412a8cdd093bbb75132ff411e2e6e9805d0e55d34aebe4a03e58f107a2a789"
  },
  "fallback": {
    "asset": "DESK-14",
    "payload": "Backup manifest for DESK-14, revision 1",
    "sha256": "5a412a8cdd093bbb75132ff411e2e6e9805d0e55d34aebe4a03e58f107a2a789"
  }
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
  "verified": true
}
```

## Testing and validation

Actual tests used SQLite and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. An invalid primary record used the saved fallback and returned the expected field.

```json
{
  "verified": true
}
```

## Error handling

The CLI reports invalid input, database errors, and file errors with status 2. A batch with rejected records preserves its valid rows and returns status 1.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The sample verifies a synthetic payload digest. It does not use server backup facilities.

## Learning outcomes

Practice backup verification using SQLite and check a repeatable recovery workflow in the help desk scenario.

## Conclusion

The project records an observable backup verification result for training case `DESK-14`.

[Back to the SQLite project index](../README.md)
