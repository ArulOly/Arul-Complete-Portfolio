# Wazuh: Monitoring — Baseline Setup for Help Desk (Project 030091)

## Project overview

A help desk training queue provides the setting for this monitoring exercise. This project includes source code, sample data, and instructions to run it. The data is synthetic.

## Problem statement

The help desk setting needs a repeatable way to review monitoring observations.

## Project objectives

- Match a simulated JSON event with a local custom rule.
- Demonstrate baseline setup with an explicit local sample.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Wazuh local rule XML, JSON event fields, and synthetic controls.

## System architecture

`Synthetic fixture → tool configuration → focused check or report`

## Features

- Match a simulated JSON event with a local custom rule.
- Demonstrate baseline setup with an explicit local sample.

## Prerequisites

Use a disposable Wazuh manager for native rule checks. Use a text editor and a local practice directory.

## Complete Wazuh source code

```xml
<group name="portfolio_lab,">
  <rule id="100190" level="5">
    <decoded_as>json</decoded_as>
    <field name="portfolio_project" type="pcre2">^030091$</field>
    <field name="portfolio_topic" type="pcre2">^monitoring$</field>
    <field name="portfolio_status" type="pcre2">^baseline-flagged$</field>
    <description>peak utilization percent | utilization above budget</description>
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
    "portfolio_project": "030091",
    "portfolio_topic": "monitoring",
    "portfolio_status": "baseline-flagged",
    "asset": "DESK-14"
  },
  "nonmatching_event": {
    "portfolio_project": "030091",
    "portfolio_topic": "monitoring",
    "portfolio_status": "normal",
    "asset": "DESK-14"
  }
}
```

## Execution instructions

In a disposable Wazuh manager, store the XML as a local custom rule in the manager's rules directory and ensure the displayed rule ID is available. In `wazuh-logtest`, submit each event object as one JSON line and inspect the rule ID.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "custom_rule_id": 100190,
  "offline_fixture_match": true,
  "offline_control_match": false
}
```

## Testing and validation

Parsed the rule XML and independently tested every anchored field condition with matching and nonmatching JSON fixtures. Wazuh decoder and manager execution remain unverified.

```json
{
  "custom_rule_id": 100190,
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

Practice monitoring using Wazuh and check a repeatable baseline setup workflow in the help desk scenario.

## Conclusion

The project records an observable monitoring result for training case `DESK-14`.

[Back to the Wazuh project index](../README.md)
