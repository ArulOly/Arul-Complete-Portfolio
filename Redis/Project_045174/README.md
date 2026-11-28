# Redis: Log Analysis — Reporting for Branch Office Simulator (Project 045174)

## Project overview

A simulated branch office provides the setting for this log analysis exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The branch office simulator needs a reproducible log analysis workflow that stores and reads results in Redis.

## Project objectives

- Count event levels in a synthetic log and identify the most frequent error.
- Build a JSON report from the database's ordered result set and row summary.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Redis hashes, multi-command transactions, and expiration, Python, JSON, and redis-py.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Count event levels in a synthetic log and identify the most frequent error.
- Build a JSON report from the database's ordered result set and row summary.

## Prerequisites

Install Python 3.10 or later and use a local Redis instance and a practice user allowed to read, write, and expire hashes and use transactions. Use a text editor and a local practice directory.

## Complete Redis source code

```python
"""Local Redis reference implementation for project 045174."""


from collections import Counter
from uuid import uuid4
import argparse
import json
import os
import redis
import sys


METRIC = 'top_error'
PROJECT_ID = '045174'


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
    """Count event levels in a synthetic log and identify the most frequent error."""
    levels = Counter()
    errors = Counter()
    for line in labels(record, "events"):
        level, delimiter, message = line.partition("|")
        level, message = level.strip(), message.strip()
        if not delimiter or level not in {"INFO", "WARN", "ERROR"} or not message:
            raise ValueError("expected INFO, WARN or ERROR followed by | and a message")
        levels[level] += 1
        if level == "ERROR":
            errors[message] += 1
    return {"counts": dict(sorted(levels.items())),
            "top_error": min(errors, key=lambda key: (-errors[key], key), default=None)}


def mapping(record, key):
    if not isinstance(record, dict):
        raise ValueError(f"{key} must be an object")
    return record


class Store:

    def __enter__(self):
        self.client = redis.Redis.from_url(os.environ["PORTFOLIO_REDIS_URL"],
                                          decode_responses=True, socket_timeout=5,
                                          socket_connect_timeout=5)
        self.key = "portfolio:demo:" + uuid4().hex
        return self

    def __exit__(self, *_):
        try:
            self.client.delete(self.key)
        finally:
            self.client.close()

    def put(self, position, source, response):
        with self.client.pipeline(transaction=True) as batch:
            batch.hset(self.key, mapping={
                f"source:{position}": dumps(source), f"response:{position}": dumps(response),
                f"rejected:{position}": int("error" in response),
            })
            if "result" in response:
                batch.hset(self.key, mapping={
                    f"metric:{position}:{key}": dumps(value)
                    for key, value in response["result"].items()
                })
            batch.expire(self.key, 300)
            batch.execute()

    def read(self, position):
        source, response = self.client.hmget(self.key, f"source:{position}", f"response:{position}")
        if source is None or response is None:
            raise ValueError("Stored row is missing")
        return json.loads(source), json.loads(response)

    def ordered(self):
        fields = self.client.hgetall(self.key)
        keys = sorted((key for key in fields if key.startswith("response:")),
                      key=lambda key: int(key.split(":")[1]))
        return [json.loads(fields[key]) for key in keys]

    def summary(self):
        entries = dict(self.client.hscan_iter(self.key, match="rejected:*"))
        return len(entries), sum(map(int, entries.values()))

    def metric(self, position, key):
        raw = self.client.hget(self.key, f"metric:{position}:{key}")
        return json.loads(raw) if raw is not None else None


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
    parser = argparse.ArgumentParser(description='Count event levels in a synthetic log and identify the most frequent error.')
    parser.add_argument("input", help="path to the synthetic JSON fixture")
    args = parser.parse_args()
    try:
        response = run(args.input)
        print(json.dumps(response, indent=2, ensure_ascii=False, sort_keys=True))
        return response["status"]
    except (redis.RedisError, OSError, ValueError, KeyError, OverflowError, RecursionError) as exc:
        print(f"Demo failed: {exc}", file=sys.stderr)
        return 2


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

`evaluate()` performs the log analysis calculation and validates its inputs. `Store` handles database writes and reads. The program uses a uniquely named Redis hash with a five-minute TTL and deletes that hash on normal exit. Build a JSON report from the database's ordered result set and row summary.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "asset": "BRANCH-08",
  "events": [
    "INFO | Started",
    "ERROR | Timeout",
    "ERROR | Timeout",
    "WARN | Retrying"
  ]
}
```

## Execution instructions

Copy the Python block to `project.py` and the sample to `sample.json` in the same directory.

Install the driver in your virtual environment with `python -m pip install redis`. Set `PORTFOLIO_REDIS_URL` to your local connection URI. Its format is `redis://127.0.0.1:6379/0`. Use your own practice credentials when authentication is required.

Run the example.

```sh
python project.py sample.json
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "top_error": "Timeout"
}
```

## Testing and validation

Actual tests used Redis and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. An incomplete record was rejected.

```json
{
  "top_error": "Timeout"
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

Practice log analysis using Redis and check a repeatable reporting workflow in the branch office simulator scenario.

## Conclusion

The project records an observable log analysis result for training case `BRANCH-08`.

[Back to the Redis project index](../README.md)
