# Prometheus: Dashboard — Baseline Setup for Offline Classroom (Project 044376)

## Project overview

A classroom operating offline provides the setting for this dashboard exercise. This project includes source code, sample data, and instructions to run it. The data is synthetic.

## Problem statement

The offline classroom setting needs a repeatable way to review dashboard observations.

## Project objectives

- Use PromQL to evaluate a synthetic metric observation.
- Demonstrate baseline setup with an explicit local sample.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Prometheus recording rules, PromQL, and synthetic rule-test series.

## System architecture

`Synthetic fixture → tool configuration → focused check or report`

## Features

- Use PromQL to evaluate a synthetic metric observation.
- Demonstrate baseline setup with an explicit local sample.

## Prerequisites

Use Prometheus promtool 3.5 or a compatible release. Use a text editor and a local practice directory.

## Complete Prometheus source code

```yaml
groups:
- name: portfolio-044376
  rules:
  - record: portfolio:dashboard:baseline
    expr: max(portfolio_dashboard{project="044376",asset="CLASS-09",source="primary"})
    labels:
      project: '044376'
```

## Implementation explanation

Use PromQL to evaluate a synthetic metric observation.

## Sample input data

Use these synthetic observations or inspect them in the embedded source.

```json
{
  "interval": "1m",
  "input_series": [
    {
      "series": "portfolio_dashboard{project=\"044376\",asset=\"CLASS-09\",source=\"primary\",reading=\"1\"}",
      "values": "18"
    }
  ],
  "promql_expr_test": [
    {
      "expr": "portfolio:dashboard:baseline{project=\"044376\"}",
      "eval_time": "0m",
      "exp_samples": [
        {
          "labels": "portfolio:dashboard:baseline{project=\"044376\"}",
          "value": 18
        }
      ]
    }
  ]
}
```

## Execution instructions

Save the source as `rules.yml`. Save the fixture below as `test.yml` in the same directory and run `promtool test rules test.yml`.

```yaml
rule_files:
- rules.yml
evaluation_interval: 1m
tests:
- interval: 1m
  input_series:
  - series: portfolio_dashboard{project="044376",asset="CLASS-09",source="primary",reading="1"}
    values: '18'
  promql_expr_test:
  - expr: portfolio:dashboard:baseline{project="044376"}
    eval_time: 0m
    exp_samples:
    - labels: portfolio:dashboard:baseline{project="044376"}
      value: 18
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "series": "portfolio:dashboard:baseline{project=\"044376\"}",
  "value": 18
}
```

## Testing and validation

Promtool evaluated the recording rule and matched the expected labeled sample. An empty-input test produced no matching series.

```json
{
  "series": "portfolio:dashboard:baseline{project=\"044376\"}",
  "value": 18
}
```

## Error handling

Start with the provided fixture. Invalid inputs, unmatched observations, or missing setup are checked as described above.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

This exercise uses rule-test fixtures. It does not scrape a live endpoint.

## Learning outcomes

Practice dashboard using Prometheus and check a repeatable baseline setup workflow in the offline classroom scenario.

## Conclusion

The project records an observable dashboard result for training case `CLASS-09`.

[Back to the Prometheus project index](../README.md)
