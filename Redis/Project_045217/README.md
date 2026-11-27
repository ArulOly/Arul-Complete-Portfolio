# Redis: Backup Verification — Automation for Public Dataset (Project 045217)

## Project overview

A synthetic public-data exercise provides the setting for this backup verification exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The public dataset needs a reproducible backup verification workflow that stores and reads results in Redis.

## Project objectives

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Store valid results and row errors in order and summarize the persisted batch.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Redis hashes, multi-command transactions, and expiration, Python, JSON, and redis-py.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Store valid results and row errors in order and summarize the persisted batch.

## Prerequisites

Install Python 3.10 or later and use a local Redis instance and a practice user allowed to read, write, and expire hashes and use transactions. Use a text editor and a local practice directory.

## Complete Redis source code

```python
"""Local Redis reference implementation for project 045217."""


from uuid import uuid4
import argparse
import hashlib
import json
import os
import redis
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

    def ordered(self):
        fields = self.client.hgetall(self.key)
        keys = sorted((key for key in fields if key.startswith("response:")),
                      key=lambda key: int(key.split(":")[1]))
        return [json.loads(fields[key]) for key in keys]

    def summary(self):
        entries = dict(self.client.hscan_iter(self.key, match="rejected:*"))
        return len(entries), sum(map(int, entries.values()))


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
    parser = argparse.ArgumentParser(description='Compare a synthetic backup payload against an expected SHA-256 digest.')
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

`evaluate()` performs the backup verification calculation and validates its inputs. `Store` handles database writes and reads. The program uses a uniquely named Redis hash with a five-minute TTL and deletes that hash on normal exit. Store valid results and row errors in order and summarize the persisted batch.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "records": [
    {
      "asset": "DATA-12",
      "payload": "Backup manifest for DATA-12, revision 1",
      "sha256": "0ee3b0100003244fc89d5bf2cce165e11955665f0e41740386d16036be1243cf"
    },
    {
      "asset": "DATA-12-B",
      "payload": "Backup manifest for DATA-12, revision 1",
      "sha256": "0ee3b0100003244fc89d5bf2cce165e11955665f0e41740386d16036be1243cf"
    }
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
  "verified": true
}
```

## Testing and validation

Actual tests used Redis and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. A mixed batch kept its valid row and the database summary reported one rejection.

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

Practice backup verification using Redis and check a repeatable automation workflow in the public dataset scenario.

## Conclusion

The project records an observable backup verification result for training case `DATA-12`.

[Back to the Redis project index](../README.md)
