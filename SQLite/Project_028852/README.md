# SQLite: Automation — Automation for School Lab (Project 028852)

## Project overview

Shared classroom equipment provides the setting for this automation exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The school lab needs a reproducible automation workflow that stores and reads results in SQLite.

## Project objectives

- Resolve a local task plan and propagate prerequisite failures to dependent work.
- Store valid results and row errors in order and summarize the persisted batch.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

SQLite temporary tables, bound SQL parameters, and JSON functions, Python, JSON, and sqlite3 from the Python standard library.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Resolve a local task plan and propagate prerequisite failures to dependent work.
- Store valid results and row errors in order and summarize the persisted batch.

## Prerequisites

Install Python 3.10 or later with the SQLite JSON functions available. Use a text editor and a local practice directory.

## Complete SQLite source code

```python
"""Local SQLite reference implementation for project 028852."""


import argparse
import json
import sqlite3
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

`evaluate()` performs the automation calculation and validates its inputs. `Store` handles database writes and reads. SQL rows live in a temporary in-memory table that closes with the connection. Store valid results and row errors in order and summarize the persisted batch.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "records": [
    {
      "asset": "LAB-17",
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
    {
      "asset": "LAB-17-B",
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
  "task_states": {
    "collect": "done",
    "verify": "failed",
    "publish": "blocked"
  }
}
```

## Testing and validation

Actual tests used SQLite and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. A mixed batch kept its valid row and the database summary reported one rejection.

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

Practice automation using SQLite and check a repeatable automation workflow in the school lab scenario.

## Conclusion

The project records an observable automation result for training case `LAB-17`.

[Back to the SQLite project index](../README.md)
