# MySQL: Data Export — Automation for Portfolio Demo (Project 026347)

## Project overview

A repeatable portfolio demonstration provides the setting for this data export exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The portfolio demo needs a reproducible data export workflow that stores and reads results in MySQL.

## Project objectives

- Convert structured records into a predictable two-column CSV.
- Store valid results and row errors in order and summarize the persisted batch.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

MySQL temporary tables, JSON_EXTRACT, and bound SQL parameters, Python, JSON, and PyMySQL.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Convert structured records into a predictable two-column CSV.
- Store valid results and row errors in order and summarize the persisted batch.

## Prerequisites

Install Python 3.10 or later and use a local MySQL database and a practice user allowed to create temporary tables. Use a text editor and a local practice directory.

## Complete MySQL source code

```python
"""Local MySQL reference implementation for project 026347."""


from urllib.parse import unquote, urlsplit
import argparse
import csv
import io
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
    parser = argparse.ArgumentParser(description='Convert structured records into a predictable two-column CSV.')
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

`evaluate()` performs the data export calculation and validates its inputs. `Store` handles database writes and reads. The session uses a temporary MySQL table with JSON columns that closes with the connection. Store valid results and row errors in order and summarize the persisted batch.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "records": [
    {
      "asset": "DEMO-02",
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
    },
    {
      "asset": "DEMO-02-B",
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
  "csv": "code,quantity\nKIT-01,4\nKIT-02,2\n"
}
```

## Testing and validation

Actual tests used MySQL and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. A mixed batch kept its valid row and the database summary reported one rejection.

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

Practice data export using MySQL and check a repeatable automation workflow in the portfolio demo scenario.

## Conclusion

The project records an observable data export result for training case `DEMO-02`.

[Back to the MySQL project index](../README.md)
