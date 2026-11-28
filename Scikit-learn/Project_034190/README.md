# Scikit-learn: Log Analysis — Usability Improvement for Two-Node Lab (Project 034190)

## Project overview

A two-node lab simulation provides the setting for this log analysis exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The two-node lab needs to compare a simulated log analysis outcome with known reference profiles.

## Project objectives

- Count event levels in a synthetic log and identify the most frequent error.
- Provide help, labeled terminal output, and a JSON output mode.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Python, scikit-learn, JSON, and the Python standard library.

## System architecture

`Structured input → workflow rules → feature dictionary → fitted scaler → nearest-neighbor comparison`

## Features

- Count event levels in a synthetic log and identify the most frequent error.
- Provide help, labeled terminal output, and a JSON output mode.

## Prerequisites

Install Python 3.10 or later and a compatible release of scikit-learn. Use a text editor and a local practice directory.

## Complete Scikit-learn source code

```python
"""Local Scikit-learn reference implementation for project 034190."""


from collections import Counter
from pathlib import Path
from sklearn.feature_extraction import DictVectorizer
from sklearn.neighbors import NearestNeighbors
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler
import argparse
import json
import math
import sys


SCENARIO = "Two-Node Lab"


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


def features(result):
    return {"errors": result["counts"].get("ERROR", 0), "warnings": result["counts"].get("WARN", 0)}


def as_object(value, name):
    if not isinstance(value, dict):
        raise ValueError(f"{name} must be an object")
    return value


def vector(data, names):
    row = as_object(data, "features")
    if set(row) != set(names):
        raise ValueError("Feature names must match the workflow feature schema")
    clean = {}
    for key, value in row.items():
        if type(value) not in (float, int) or not math.isfinite(value) or abs(value) > 1e12:
            raise ValueError(f"{key} must be a finite number within ±1e12")
        clean[key] = value
    return clean


def fit_reference(rows):
    if len(rows) < 2:
        raise ValueError("At least two reference profiles are required")
    labels, records = [], []
    names = None
    for value in rows:
        row = as_object(value, "reference profile")
        name = label(row, "label")
        if name in labels:
            raise ValueError(f"Duplicate reference label: {name}")
        data = as_object(row.get("features"), "features")
        if names is None:
            names = frozenset(data)
            if not names:
                raise ValueError("Reference profiles need at least one feature")
        labels.append(name)
        records.append(vector(data, names))
    transform = make_pipeline(DictVectorizer(sparse=False), StandardScaler())
    training = transform.fit_transform(records)
    nearest = NearestNeighbors(algorithm="ball_tree").fit(training)
    return labels, transform, nearest, names


def candidate(value, model, full=False):
    record = as_object(value, "candidate")
    result = evaluate(record)
    labels, transform, nearest, names = model
    extracted = vector(features(result), names)
    point = transform.transform([extracted])
    distances, indices = nearest.kneighbors(point, n_neighbors=len(labels))
    ranked = sorted(
        ((float(distance), labels[int(index)]) for distance, index in zip(distances[0], indices[0])),
        key=lambda item: (item[0], item[1]),
    )
    distance, name = ranked[0]
    response = {"asset": label(record, "asset"), "result": result, "features": extracted,
                "nearest_profile": name, "distance": round(distance, 6)}
    if full:
        response["profile_distances"] = [
            {"label": profile, "distance": round(gap, 6)} for gap, profile in ranked
        ]
    return response


def analyze(payload):
    payload = as_object(payload, "input")
    model = fit_reference(array(payload, "reference"))
    return {"scenario": SCENARIO, "summary": candidate(payload.get("candidate"), model)}


def run(path):
    data = json.loads(Path(path).read_text(encoding="utf-8"))
    json.dumps(data, allow_nan=False)
    return analyze(data)


def main():
    parser = argparse.ArgumentParser(description="Compare a workflow record with synthetic reference profiles")
    parser.add_argument("input", type=Path, help="JSON fixture containing references and candidates")
    parser.add_argument("--json", action="store_true", help="print structured JSON")
    args = parser.parse_args()
    try:
        result = run(args.input)
        if args.json:
            print(json.dumps(result, ensure_ascii=False, indent=2))
        else:
            summary = result["summary"]
            print(f"{SCENARIO} | {summary['asset']}")
            print(f"Nearest profile: {summary['nearest_profile']} (distance {summary['distance']})")
        return int(result.get("failed", 0) > 0)
    except (OSError, ValueError, OverflowError, RecursionError) as exc:
        print(f"Analysis error: {exc}", file=sys.stderr)
        return 2


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

`evaluate()` applies the log analysis rules before training-profile comparison. DictVectorizer fixes feature ordering, StandardScaler fits on the reference rows, and NearestNeighbors ranks the transformed candidate by Euclidean distance.

## Sample input data

The reference profiles are synthetic feature vectors. The matching profile is included deliberately to verify zero distance.

```json
{
  "reference": [
    {
      "label": "reference-low",
      "features": {
        "errors": 1,
        "warnings": 0
      }
    },
    {
      "label": "reference-match",
      "features": {
        "errors": 2,
        "warnings": 1
      }
    },
    {
      "label": "reference-high",
      "features": {
        "errors": 3,
        "warnings": 2
      }
    }
  ],
  "candidate": {
    "asset": "NODE-B",
    "events": [
      "INFO | Started",
      "ERROR | Timeout",
      "ERROR | Timeout",
      "WARN | Retrying"
    ]
  }
}
```

## Execution instructions

Save the code as `project.py` and the fixture as `sample.json`. Run these commands in a Python virtual environment.

```sh
python -m pip install scikit-learn
python project.py sample.json --json
```

Pass `--help` for usage. Omit `--json` for a labeled summary.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "top_error": "Timeout",
  "nearest_profile": "reference-match",
  "distance": 0.0
}
```

## Testing and validation

Actual tests used scikit-learn 1.8.0 and temporary JSON files. The matching synthetic reference returned the workflow field and zero distance twice. An incomplete candidate was rejected. Removing the matching reference produced a positive distance.

```json
{
  "top_error": "Timeout",
  "nearest_profile": "reference-match",
  "distance": 0.0
}
```

## Error handling

Field or model input errors produce diagnostics. Batch mode keeps valid rows, and recovery mode can compare a fallback.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The reference dataset is a tiny synthetic exercise. Distance measures similarity under these features and has no calibrated probability or operational decision meaning.

## Learning outcomes

Practice log analysis using Scikit-learn and check a repeatable usability improvement workflow in the two-node lab scenario.

## Conclusion

The project records an observable log analysis result for training case `NODE-B`.

[Back to the Scikit-learn project index](../README.md)
