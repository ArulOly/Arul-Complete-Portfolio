# Grafana: Audit — Automation for Two-Node Lab (Project 043137)

## Project overview

A two-node lab simulation provides the setting for this audit exercise. This project includes source code, sample data, and instructions to run it. The data is synthetic.

## Problem statement

The two-node lab setting needs a repeatable way to review audit observations.

## Project objectives

- Visualize synthetic readings with the built-in TestData source.
- Demonstrate automation with an explicit local sample.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Grafana dashboard JSON, TestData CSV Content, and dashboard field thresholds.

## System architecture

`Synthetic fixture → tool configuration → focused check or report`

## Features

- Visualize synthetic readings with the built-in TestData source.
- Demonstrate automation with an explicit local sample.

## Prerequisites

Use Grafana with the built-in TestData data source enabled. Use a text editor and a local practice directory.

## Complete Grafana source code

```json
{
  "__inputs": [
    {
      "name": "TESTDATA_UID",
      "label": "TestData",
      "type": "datasource",
      "pluginId": "grafana-testdata-datasource",
      "pluginName": "TestData"
    }
  ],
  "uid": "portfolio-043137",
  "title": "Days since last review | NODE-B (043137)",
  "schemaVersion": 39,
  "version": 1,
  "tags": [
    "portfolio",
    "audit",
    "batch"
  ],
  "panels": [
    {
      "id": 1,
      "title": "Days since last review",
      "type": "table",
      "gridPos": {
        "h": 9,
        "w": 24,
        "x": 0,
        "y": 0
      },
      "datasource": {
        "type": "grafana-testdata-datasource",
        "uid": "${TESTDATA_UID}"
      },
      "targets": [
        {
          "refId": "A",
          "scenarioId": "csv_content",
          "csvContent": "asset,metric,source,reading,threshold\nNODE-B,days since last review,primary,45,30\nNODE-B,days since last review,primary,30,30\n"
        }
      ],
      "fieldConfig": {
        "defaults": {},
        "overrides": [
          {
            "matcher": {
              "id": "byName",
              "options": "reading"
            },
            "properties": [
              {
                "id": "thresholds",
                "value": {
                  "mode": "absolute",
                  "steps": [
                    {
                      "color": "green",
                      "value": null
                    },
                    {
                      "color": "red",
                      "value": 31
                    }
                  ]
                }
              },
              {
                "id": "color",
                "value": {
                  "mode": "thresholds"
                }
              }
            ]
          }
        ]
      },
      "options": {
        "showHeader": true
      }
    }
  ]
}
```

## Implementation explanation

Visualize synthetic readings with the built-in TestData source.

## Sample input data

Use these synthetic observations or inspect them in the embedded source.

```json
{
  "embedded_csv": "asset,metric,source,reading,threshold\nNODE-B,days since last review,primary,45,30\nNODE-B,days since last review,primary,30,30\n"
}
```

## Execution instructions

Add a **TestData** data source to your local Grafana instance. Save the source as `dashboard.json`, import it as a dashboard, and select your TestData data source when prompted.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "rows": 2,
  "source": "primary",
  "readings": [
    45,
    30
  ],
  "threshold_step": 31
}
```

## Testing and validation

Parsed the dashboard JSON and independently checked the CSV values, source labels, and threshold configuration. Grafana import and rendering remain unverified.

```json
{
  "rows": 2,
  "source": "primary",
  "readings": [
    45,
    30
  ],
  "threshold_step": 31
}
```

## Error handling

Start with the provided fixture. Invalid inputs, unmatched observations, or missing setup are checked as described above.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The dashboard uses embedded data. Layout and visual behavior in Grafana remain unverified.

## Learning outcomes

Practice audit using Grafana and check a repeatable automation workflow in the two-node lab scenario.

## Conclusion

The project records an observable audit result for training case `NODE-B`.

[Back to the Grafana project index](../README.md)
