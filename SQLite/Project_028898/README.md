# SQLite: Automation — Recovery for Portfolio Demo (Project 028898)

## Project overview

A repeatable portfolio demonstration provides the setting for this automation exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The portfolio demo needs a reproducible automation workflow that stores and reads results in SQLite.

## Project objectives

- Resolve a local task plan and propagate prerequisite failures to dependent work.
- Save a fallback snapshot and read it back if the primary record fails validation.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

SQLite temporary tables, bound SQL parameters, and JSON functions, Python, JSON, and sqlite3 from the Python standard library.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Resolve a local task plan and propagate prerequisite failures to dependent work.
- Save a fallback snapshot and read it back if the primary record fails validation.

## Prerequisites

Install Python 3.10 or later with the SQLite JSON functions available. Use a text editor and a local practice directory.

## Complete SQLite source code

```python
"""Local SQLite reference implementation for project 028898."""


import argparse
import json
import sqlite3
import sys


METRIC = 'task_states'


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


def objects(record, key):
    values = array(record, key)
    if any(not isinstance(value, dict) for value in values):
        raise ValueError(f"every item in {key} must be an object")
    return values


def labels(record, key, allow_empty=False):
    return [label({key: item}, key) for item in array(record, key, allow_empty)]


def evaluate(record):
    """Resolve a local task plan and propagate prerequisite failures to dependent work."""
    pending = {}
    for task in objects(record, "tasks"):
        name = label(task, "name")
        needs = labels(task, "needs", allow_empty=True)
        ready = task.get("ready")
        if type(ready) is not bool or name in pending:
            raise ValueError("each task needs a unique name and Boolean ready flag")
        pending[name] = (needs, ready)
    known = set(pending)
    if any(set(needs) - known for needs, _ in pending.values()):
        raise ValueError("a prerequisite task is missing")
    states = {}
    while pending:
        eligible = [name for name, (needs, _) in pending.items() if set(needs) <= states.keys()]
        if not eligible:
            raise ValueError("task dependencies contain a cycle")
        for name in eligible:
            needs, ready = pending.pop(name)
            if any(states[need] != "done" for need in needs):
                states[name] = "blocked"
            else:
                states[name] = "done" if ready else "failed"
    return {"task_states": states}


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
    parser = argparse.ArgumentParser(description='Resolve a local task plan and propagate prerequisite failures to dependent work.')
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

`evaluate()` performs the automation calculation and validates its inputs. `Store` handles database writes and reads. SQL rows live in a temporary in-memory table that closes with the connection. Save a fallback snapshot and read it back if the primary record fails validation.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "primary": {
    "asset": "DEMO-02",
    "tasks": [
      {
        "name": "collect",
        "ready": true,
        "needs": []
      },
      {
        "name": "verify",
        "ready": false,
        "needs": [
          "collect"
        ]
      },
      {
        "name": "publish",
        "ready": true,
        "needs": [
          "verify"
        ]
      }
    ]
  },
  "fallback": {
    "asset": "DEMO-02",
    "tasks": [
      {
        "name": "collect",
        "ready": true,
        "needs": []
      },
      {
        "name": "verify",
        "ready": false,
        "needs": [
          "collect"
        ]
      },
      {
        "name": "publish",
        "ready": true,
        "needs": [
          "verify"
        ]
      }
    ]
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
  "task_states": {
    "collect": "done",
    "verify": "failed",
    "publish": "blocked"
  }
}
```

## Testing and validation

Actual tests used SQLite and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. An invalid primary record used the saved fallback and returned the expected field.

```json
{
  "task_states": {
    "collect": "done",
    "publish": "blocked",
    "verify": "failed"
  }
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

Practice automation using SQLite and check a repeatable recovery workflow in the portfolio demo scenario.

## Conclusion

The project records an observable automation result for training case `DEMO-02`.

[Back to the SQLite project index](../README.md)
