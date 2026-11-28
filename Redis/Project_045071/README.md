# Redis: Monitoring — Baseline Setup for Branch Office Simulator (Project 045071)

## Project overview

A simulated branch office provides the setting for this monitoring exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The branch office simulator needs a reproducible monitoring workflow that stores and reads results in Redis.

## Project objectives

- Summarize utilization readings and count readings above the limit.
- Create a session-local store, save a validated baseline, and read it back by row.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Redis hashes, multi-command transactions, and expiration, Python, JSON, and redis-py.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Summarize utilization readings and count readings above the limit.
- Create a session-local store, save a validated baseline, and read it back by row.

## Prerequisites

Install Python 3.10 or later and use a local Redis instance and a practice user allowed to read, write, and expire hashes and use transactions. Use a text editor and a local practice directory.

## Complete Redis source code

```python
"""Local Redis reference implementation for project 045071."""


from uuid import uuid4
import argparse
import json
import os
import redis
import statistics
import sys


METRIC = 'above_limit'


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


def series(record, key, maximum=1_000_000_000):
    return [integer({key: item}, key, maximum=maximum) for item in array(record, key)]


def evaluate(record):
    """Summarize utilization readings and count readings above the limit."""
    readings = series(record, "readings_pct", maximum=100)
    limit = integer(record, "limit_pct", maximum=100)
    return {"mean_pct": round(statistics.mean(readings), 2),
            "peak_pct": max(readings), "above_limit": sum(n > limit for n in readings)}


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
    record = mapping(payload, "input")
    result = evaluate_record(record)
    store.put(1, record, result)
    original, restored = store.read(1)
    if original != record:
        raise ValueError("Stored baseline changed")
    return {"baseline": restored, "metric": store.metric(1, METRIC), "status": 0}


def run(path):
    payload = load_input(path)
    with Store() as store:
        return process(payload, store)


def main():
    parser = argparse.ArgumentParser(description='Summarize utilization readings and count readings above the limit.')
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

`evaluate()` performs the monitoring calculation and validates its inputs. `Store` handles database writes and reads. The program uses a uniquely named Redis hash with a five-minute TTL and deletes that hash on normal exit. Create a session-local store, save a validated baseline, and read it back by row.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "asset": "BRANCH-08",
  "readings_pct": [
    35,
    81,
    54
  ],
  "limit_pct": 80
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
  "above_limit": 1
}
```

## Testing and validation

Actual tests used Redis and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. An incomplete record was rejected.

```json
{
  "above_limit": 1
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

Practice monitoring using Redis and check a repeatable baseline setup workflow in the branch office simulator scenario.

## Conclusion

The project records an observable monitoring result for training case `BRANCH-08`.

[Back to the Redis project index](../README.md)
