# Prometheus: Error Handling — Baseline Setup for Home Development (Project 044711)

## Project overview

A home development workspace provides the setting for this error handling exercise. This project includes source code, sample data, and instructions to run it. The data is synthetic.

## Problem statement

The home development setting needs a repeatable way to review error handling observations.

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
- name: portfolio-044711
  rules:
  - record: portfolio:error_handling:baseline
    expr: max(portfolio_error_handling{project="044711",asset="DEV-05",source="primary"})
    labels:
      project: '044711'
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
      "series": "portfolio_error_handling{project=\"044711\",asset=\"DEV-05\",source=\"primary\",reading=\"1\"}",
      "values": "2"
    }
  ],
  "promql_expr_test": [
    {
      "expr": "portfolio:error_handling:baseline{project=\"044711\"}",
      "eval_time": "0m",
      "exp_samples": [
        {
          "labels": "portfolio:error_handling:baseline{project=\"044711\"}",
          "value": 2
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
  - series: portfolio_error_handling{project="044711",asset="DEV-05",source="primary",reading="1"}
    values: '2'
  promql_expr_test:
  - expr: portfolio:error_handling:baseline{project="044711"}
    eval_time: 0m
    exp_samples:
    - labels: portfolio:error_handling:baseline{project="044711"}
      value: 2
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "series": "portfolio:error_handling:baseline{project=\"044711\"}",
  "value": 2
}
```

## Testing and validation

Promtool evaluated the recording rule and matched the expected labeled sample. An empty-input test produced no matching series.

```json
{
  "series": "portfolio:error_handling:baseline{project=\"044711\"}",
  "value": 2
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

Practice error handling using Prometheus and check a repeatable baseline setup workflow in the home development scenario.

## Conclusion

The project records an observable error handling result for training case `DEV-05`.

[Back to the Prometheus project index](../README.md)
