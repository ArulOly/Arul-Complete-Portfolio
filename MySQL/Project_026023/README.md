# MySQL: Inventory — Recovery for Branch Office Simulator (Project 026023)

## Project overview

A simulated branch office provides the setting for this inventory exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The branch office simulator needs a reproducible inventory workflow that stores and reads results in MySQL.

## Project objectives

- Count items and identify stock below the reorder level.
- Save a fallback snapshot and read it back if the primary record fails validation.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

MySQL temporary tables, JSON_EXTRACT, and bound SQL parameters, Python, JSON, and PyMySQL.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Count items and identify stock below the reorder level.
- Save a fallback snapshot and read it back if the primary record fails validation.

## Prerequisites

Install Python 3.10 or later and use a local MySQL database and a practice user allowed to create temporary tables. Use a text editor and a local practice directory.

## Complete MySQL source code

```python
"""Local MySQL reference implementation for project 026023."""


from urllib.parse import unquote, urlsplit
import argparse
import json
import os
import pymysql
import sys


METRIC = 'needs_reorder'


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


def evaluate(record):
    """Count items and identify stock below the reorder level."""
    stock = integer(record, "stock")
    reorder = integer(record, "reorder_at", minimum=1)
    return {"stock": stock, "reorder_at": reorder, "needs_reorder": stock < reorder}


def mapping(record, key):
    if not isinstance(record, dict):
        raise ValueError(f"{key} must be an object")
    return record


class Store:

    def __enter__(self):
        uri = urlsplit(os.environ["PORTFOLIO_MYSQL_URI"])
        if uri.scheme != "mysql" or not uri.hostname or not uri.username or not uri.path.strip("/"):
            raise ValueError("Use mysql://USER:PASSWORD@HOST:PORT/DATABASE")
        self.database = pymysql.connect(
            host=uri.hostname, port=uri.port or 3306, user=unquote(uri.username),
            password=unquote(uri.password or ""), database=unquote(uri.path.strip("/")),
            connect_timeout=5, charset="utf8mb4", autocommit=True,
        )
        self.cursor = self.database.cursor()
        try:
            self.cursor.execute('CREATE TEMPORARY TABLE portfolio_rows (\n                    position INTEGER PRIMARY KEY,\n                    source JSON NOT NULL,\n                    response JSON NOT NULL,\n                    rejected INTEGER NOT NULL CHECK (rejected IN (0, 1))\n                )')
        except pymysql.MySQLError:
            self.database.close()
            raise
        return self

    def __exit__(self, *_):
        self.database.close()

    def put(self, position, source, response):
        self.cursor.execute('INSERT INTO portfolio_rows VALUES (%s, %s, %s, %s)',
                            (position, dumps(source), dumps(response), int("error" in response)))

    def read(self, position):
        self.cursor.execute('SELECT source, response FROM portfolio_rows WHERE position = %s', (position,))
        row = self.cursor.fetchone()
        if row is None:
            raise ValueError("Stored row is missing")
        return json.loads(row[0]), json.loads(row[1])

    def metric(self, position, key):
        self.cursor.execute("SELECT JSON_EXTRACT(response, CONCAT('$.result.', %s)) FROM portfolio_rows WHERE position = %s", (key, position))
        value = self.cursor.fetchone()[0]
        return json.loads(value) if True and value is not None else value


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
    parser = argparse.ArgumentParser(description='Count items and identify stock below the reorder level.')
    parser.add_argument("input", help="path to the synthetic JSON fixture")
    args = parser.parse_args()
    try:
        response = run(args.input)
        print(json.dumps(response, indent=2, ensure_ascii=False, sort_keys=True))
        return response["status"]
    except (pymysql.MySQLError, OSError, ValueError, KeyError, OverflowError, RecursionError) as exc:
        print(f"Demo failed: {exc}", file=sys.stderr)
        return 2


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

`evaluate()` performs the inventory calculation and validates its inputs. `Store` handles database writes and reads. The session uses a temporary MySQL table with JSON columns that closes with the connection. Save a fallback snapshot and read it back if the primary record fails validation.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "primary": {
    "asset": "BRANCH-08",
    "stock": 2,
    "reorder_at": 4
  },
  "fallback": {
    "asset": "BRANCH-08",
    "stock": 2,
    "reorder_at": 4
  }
}
```

## Execution instructions

Copy the Python block to `project.py` and the sample to `sample.json` in the same directory.

Install the driver in your virtual environment with `python -m pip install PyMySQL cryptography`. Set `PORTFOLIO_MYSQL_URI` to your local connection URI. Its format is `mysql://USER:PASSWORD@127.0.0.1:3306/portfolio_lab`. Use your own practice credentials when authentication is required.

Run the example.

```sh
python project.py sample.json
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "needs_reorder": true
}
```

## Testing and validation

Actual tests used MySQL and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. An invalid primary record used the saved fallback and returned the expected field.

```json
{
  "needs_reorder": true
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

Practice inventory using MySQL and check a repeatable recovery workflow in the branch office simulator scenario.

## Conclusion

The project records an observable inventory result for training case `BRANCH-08`.

[Back to the MySQL project index](../README.md)
