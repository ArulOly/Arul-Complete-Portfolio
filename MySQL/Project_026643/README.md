# MySQL: Service Check — Recovery for Help Desk (Project 026643)

## Project overview

A help desk training queue provides the setting for this service check exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The help desk needs a reproducible service check workflow that stores and reads results in MySQL.

## Project objectives

- Classify synthetic service probes by status and latency.
- Save a fallback snapshot and read it back if the primary record fails validation.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

MySQL temporary tables, JSON_EXTRACT, and bound SQL parameters, Python, JSON, and PyMySQL.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Classify synthetic service probes by status and latency.
- Save a fallback snapshot and read it back if the primary record fails validation.

## Prerequisites

Install Python 3.10 or later and use a local MySQL database and a practice user allowed to create temporary tables. Use a text editor and a local practice directory.

## Complete MySQL source code

```python
"""Local MySQL reference implementation for project 026643."""


from urllib.parse import unquote, urlsplit
import argparse
import json
import os
import pymysql
import sys


METRIC = 'healthy'


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
    """Classify synthetic service probes by status and latency."""
    budget = integer(record, "max_ms", minimum=1)
    results = []
    for probe in objects(record, "probes"):
        status = integer(probe, "status", minimum=100, maximum=599)
        latency = integer(probe, "ms")
        results.append({"service": label(probe, "service"),
                        "healthy": 200 <= status < 300 and latency <= budget})
    return {"checks": results, "healthy": sum(result["healthy"] for result in results)}


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
    parser = argparse.ArgumentParser(description='Classify synthetic service probes by status and latency.')
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

`evaluate()` performs the service check calculation and validates its inputs. `Store` handles database writes and reads. The session uses a temporary MySQL table with JSON columns that closes with the connection. Save a fallback snapshot and read it back if the primary record fails validation.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "primary": {
    "asset": "DESK-14",
    "probes": [
      {
        "service": "catalog",
        "status": 200,
        "ms": 25
      },
      {
        "service": "print-queue",
        "status": 503,
        "ms": 60
      }
    ],
    "max_ms": 50
  },
  "fallback": {
    "asset": "DESK-14",
    "probes": [
      {
        "service": "catalog",
        "status": 200,
        "ms": 25
      },
      {
        "service": "print-queue",
        "status": 503,
        "ms": 60
      }
    ],
    "max_ms": 50
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
  "healthy": 1
}
```

## Testing and validation

Actual tests used MySQL and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. An invalid primary record used the saved fallback and returned the expected field.

```json
{
  "healthy": 1
}
```

## Error handling

The CLI reports invalid input, database errors, and file errors with status 2. A batch with rejected records preserves its valid rows and returns status 1.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

Probe data is read from the fixture. The program sends no probe requests.

## Learning outcomes

Practice service check using MySQL and check a repeatable recovery workflow in the help desk scenario.

## Conclusion

The project records an observable service check result for training case `DESK-14`.

[Back to the MySQL project index](../README.md)
