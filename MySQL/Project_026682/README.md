# MySQL: Configuration Comparison — Automation for Single-Host Lab (Project 026682)

## Project overview

One simulated lab host provides the setting for this configuration comparison exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The single-host lab needs a reproducible configuration comparison workflow that stores and reads results in MySQL.

## Project objectives

- Identify added, removed, and changed settings between two configuration objects.
- Store valid results and row errors in order and summarize the persisted batch.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

MySQL temporary tables, JSON_EXTRACT, and bound SQL parameters, Python, JSON, and PyMySQL.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Identify added, removed, and changed settings between two configuration objects.
- Store valid results and row errors in order and summarize the persisted batch.

## Prerequisites

Install Python 3.10 or later and use a local MySQL database and a practice user allowed to create temporary tables. Use a text editor and a local practice directory.

## Complete MySQL source code

```python
"""Local MySQL reference implementation for project 026682."""


from urllib.parse import unquote, urlsplit
import argparse
import json
import os
import pymysql
import sys


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


def evaluate(record):
    """Identify added, removed, and changed settings between two configuration objects."""
    baseline = field_mapping(record, "baseline")
    current = field_mapping(record, "current")
    return {"added": sorted(current.keys() - baseline.keys()),
            "removed": sorted(baseline.keys() - current.keys()),
            "changed": sorted(key for key in baseline.keys() & current.keys()
                              if json.dumps(baseline[key], sort_keys=True)
                              != json.dumps(current[key], sort_keys=True))}


def field_mapping(record, key):
    value = record.get(key)
    if not isinstance(value, dict):
        raise ValueError(f"{key} must be an object")
    return value


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

    def ordered(self):
        self.cursor.execute('SELECT response FROM portfolio_rows ORDER BY position')
        return [json.loads(row[0]) for row in self.cursor.fetchall()]

    def summary(self):
        self.cursor.execute('SELECT COUNT(*), COALESCE(SUM(rejected), 0) FROM portfolio_rows')
        total, failed = self.cursor.fetchone()
        return int(total), int(failed)


def load_input(path):
    with open(path, encoding="utf-8") as handle:
        value = json.load(handle)
    dumps(value)  # Reject non-finite numbers before any database write.
    return value


def evaluate_record(record):
    return {"asset": label(record, "asset"), "result": evaluate(record)}


def process(payload, store):
    records = array(mapping(payload, "input"), "records")
    for position, candidate in enumerate(records, 1):
        try:
            response = evaluate_record(mapping(candidate, f"row {position}"))
        except (ValueError, OverflowError) as exc:
            response = {"error": str(exc)}
        store.put(position, candidate, response)
    total, failed = store.summary()
    return {"rows": store.ordered(), "total": total, "failed": failed,
            "status": int(failed > 0)}


def run(path):
    payload = load_input(path)
    with Store() as store:
        return process(payload, store)


def main():
    parser = argparse.ArgumentParser(description='Identify added, removed, and changed settings between two configuration objects.')
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

`evaluate()` performs the configuration comparison calculation and validates its inputs. `Store` handles database writes and reads. The session uses a temporary MySQL table with JSON columns that closes with the connection. Store valid results and row errors in order and summarize the persisted batch.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "records": [
    {
      "asset": "HOST-03",
      "baseline": {
        "timeout": 30,
        "theme": "light"
      },
      "current": {
        "timeout": 45,
        "retries": 2
      }
    },
    {
      "asset": "HOST-03-B",
      "baseline": {
        "timeout": 30,
        "theme": "light"
      },
      "current": {
        "timeout": 45,
        "retries": 2
      }
    }
  ]
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
  "changed": [
    "timeout"
  ]
}
```

## Testing and validation

Actual tests used MySQL and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. A mixed batch kept its valid row and the database summary reported one rejection.

```json
{
  "changed": [
    "timeout"
  ]
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

Practice configuration comparison using MySQL and check a repeatable automation workflow in the single-host lab scenario.

## Conclusion

The project records an observable configuration comparison result for training case `HOST-03`.

[Back to the MySQL project index](../README.md)
