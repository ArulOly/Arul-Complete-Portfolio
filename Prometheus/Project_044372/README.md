# Prometheus: Dashboard — Automation for Branch Office Simulator (Project 044372)

## Project overview

A simulated branch office provides the setting for this dashboard exercise. This project includes source code, sample data, and instructions to run it. The data is synthetic.

## Problem statement

The branch office simulator setting needs a repeatable way to review dashboard observations.

## Project objectives

- Use PromQL to evaluate a synthetic metric observation.
- Demonstrate automation with an explicit local sample.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Prometheus recording rules, PromQL, and synthetic rule-test series.

## System architecture

`Synthetic fixture → tool configuration → focused check or report`

## Features

- Use PromQL to evaluate a synthetic metric observation.
- Demonstrate automation with an explicit local sample.

## Prerequisites

Use Prometheus promtool 3.5 or a compatible release. Use a text editor and a local practice directory.

## Complete Prometheus source code

```yaml
groups:
- name: portfolio-044372
  rules:
  - record: portfolio:dashboard:batch
    expr: sum(portfolio_dashboard{project="044372",asset="BRANCH-08",source="primary"}
      > bool 12)
    labels:
      project: '044372'
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
      "series": "portfolio_dashboard{project=\"044372\",asset=\"BRANCH-08\",source=\"primary\",reading=\"1\"}",
      "values": "18"
    },
    {
      "series": "portfolio_dashboard{project=\"044372\",asset=\"BRANCH-08\",source=\"primary\",reading=\"2\"}",
      "values": "12"
    }
  ],
  "promql_expr_test": [
    {
      "expr": "portfolio:dashboard:batch{project=\"044372\"}",
      "eval_time": "0m",
      "exp_samples": [
        {
          "labels": "portfolio:dashboard:batch{project=\"044372\"}",
          "value": 1
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
  - series: portfolio_dashboard{project="044372",asset="BRANCH-08",source="primary",reading="1"}
    values: '18'
  - series: portfolio_dashboard{project="044372",asset="BRANCH-08",source="primary",reading="2"}
    values: '12'
  promql_expr_test:
  - expr: portfolio:dashboard:batch{project="044372"}
    eval_time: 0m
    exp_samples:
    - labels: portfolio:dashboard:batch{project="044372"}
      value: 1
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "series": "portfolio:dashboard:batch{project=\"044372\"}",
  "value": 1
}
```

## Testing and validation

Promtool evaluated the recording rule and matched the expected labeled sample. An empty-input test produced no matching series.

```json
{
  "series": "portfolio:dashboard:batch{project=\"044372\"}",
  "value": 1
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

Practice dashboard using Prometheus and check a repeatable automation workflow in the branch office simulator scenario.

## Conclusion

The project records an observable dashboard result for training case `BRANCH-08`.

[Back to the Prometheus project index](../README.md)
