# Wazuh: Performance Baseline — Reporting for Single-Host Lab (Project 030584)

## Project overview

One simulated lab host provides the setting for this performance baseline exercise. This project includes source code, sample data, and instructions to run it. The data is synthetic.

## Problem statement

The single-host lab setting needs a repeatable way to review performance baseline observations.

## Project objectives

- Match a simulated JSON event with a local custom rule.
- Demonstrate reporting with an explicit local sample.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Wazuh local rule XML, JSON event fields, and synthetic controls.

## System architecture

`Synthetic fixture → tool configuration → focused check or report`

## Features

- Match a simulated JSON event with a local custom rule.
- Demonstrate reporting with an explicit local sample.

## Prerequisites

Use a disposable Wazuh manager for native rule checks. Use a text editor and a local practice directory.

## Complete Wazuh source code

```xml
<group name="portfolio_lab,">
  <rule id="100683" level="5">
    <decoded_as>json</decoded_as>
    <field name="portfolio_project" type="pcre2">^030584$</field>
    <field name="portfolio_topic" type="pcre2">^performance_baseline$</field>
    <field name="portfolio_status" type="pcre2">^report-ready$</field>
    <description>p95 latency in milliseconds | latency above baseline</description>
  </rule>
</group>
```

## Implementation explanation

Match a simulated JSON event with a local custom rule.

## Sample input data

Use these synthetic observations or inspect them in the embedded source.

```json
{
  "matching_event": {
    "portfolio_project": "030584",
    "portfolio_topic": "performance_baseline",
    "portfolio_status": "report-ready",
    "asset": "HOST-03"
  },
  "nonmatching_event": {
    "portfolio_project": "030584",
    "portfolio_topic": "performance_baseline",
    "portfolio_status": "normal",
    "asset": "HOST-03"
  }
}
```

## Execution instructions

In a disposable Wazuh manager, store the XML as a local custom rule in the manager's rules directory and ensure the displayed rule ID is available. In `wazuh-logtest`, submit each event object as one JSON line and inspect the rule ID.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "custom_rule_id": 100683,
  "offline_fixture_match": true,
  "offline_control_match": false
}
```

## Testing and validation

Parsed the rule XML and independently tested every anchored field condition with matching and nonmatching JSON fixtures. Wazuh decoder and manager execution remain unverified.

```json
{
  "custom_rule_id": 100683,
  "offline_fixture_match": true,
  "offline_control_match": false
}
```

## Error handling

Start with the provided fixture. Invalid inputs, unmatched observations, or missing setup are checked as described above.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

This is a defensive rule exercise using fictional log entries. It neither collects live logs nor changes monitoring on another machine.

## Learning outcomes

Practice performance baseline using Wazuh and check a repeatable reporting workflow in the single-host lab scenario.

## Conclusion

The project records an observable performance baseline result for training case `HOST-03`.

[Back to the Wazuh project index](../README.md)
