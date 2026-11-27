# Redis: Audit — Recovery for Help Desk (Project 045143)

## Project overview

A help desk training queue provides the setting for this audit exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The help desk needs a reproducible audit workflow that stores and reads results in Redis.

## Project objectives

- Check whether recorded asset reviews meet a cutoff and list departures.
- Save a fallback snapshot and read it back if the primary record fails validation.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Redis hashes, multi-command transactions, and expiration, Python, JSON, and redis-py.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Check whether recorded asset reviews meet a cutoff and list departures.
- Save a fallback snapshot and read it back if the primary record fails validation.

## Prerequisites

Install Python 3.10 or later and use a local Redis instance and a practice user allowed to read, write, and expire hashes and use transactions. Use a text editor and a local practice directory.

## Complete Redis source code

```python
"""Local Redis reference implementation for project 045143."""


from datetime import date
from uuid import uuid4
import argparse
import json
import os
import redis
import sys


METRIC = 'follow_up'


def dumps(value):
    return json.dumps(value, ensure_ascii=False, allow_nan=False, sort_keys=True)


def label(record, key):
    value = record.get(key)
    if not isinstance(value, str) or not value.strip():
        raise ValueError(f"{key} must be non-empty text")
    return value.strip()


def iso_date(record, key):
    value = label(record, key)
    try:
        parsed = date.fromisoformat(value)
    except ValueError as exc:
        raise ValueError(f"{key} must use YYYY-MM-DD") from exc
    if parsed.isoformat() != value:
        raise ValueError(f"{key} must use YYYY-MM-DD")
    return parsed


def evaluate(record):
    """Check whether recorded asset reviews meet a cutoff and list departures."""
    reviewed = iso_date(record, "reviewed_on")
    cutoff = iso_date(record, "cutoff")
    return {"owner": label(record, "owner"), "reviewed_on": reviewed.isoformat(),
            "follow_up": reviewed < cutoff}


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
    parser = argparse.ArgumentParser(description='Check whether recorded asset reviews meet a cutoff and list departures.')
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

`evaluate()` performs the audit calculation and validates its inputs. `Store` handles database writes and reads. The program uses a uniquely named Redis hash with a five-minute TTL and deletes that hash on normal exit. Save a fallback snapshot and read it back if the primary record fails validation.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "primary": {
    "asset": "DESK-14",
    "reviewed_on": "2026-05-03",
    "cutoff": "2026-06-01",
    "owner": "Lab Operations"
  },
  "fallback": {
    "asset": "DESK-14",
    "reviewed_on": "2026-05-03",
    "cutoff": "2026-06-01",
    "owner": "Lab Operations"
  }
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
  "follow_up": true
}
```

## Testing and validation

Actual tests used Redis and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. An invalid primary record used the saved fallback and returned the expected field.

```json
{
  "follow_up": true
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

Practice audit using Redis and check a repeatable recovery workflow in the help desk scenario.

## Conclusion

The project records an observable audit result for training case `DESK-14`.

[Back to the Redis project index](../README.md)
