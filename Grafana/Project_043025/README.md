# Grafana: Inventory — Usability Improvement for Branch Office Simulator (Project 043025)

## Project overview

A simulated branch office provides the setting for this inventory exercise. This project includes source code, sample data, and instructions to run it. The data is synthetic.

## Problem statement

The branch office simulator setting needs a repeatable way to review inventory observations.

## Project objectives

- Visualize synthetic readings with the built-in TestData source.
- Demonstrate usability improvement with an explicit local sample.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Grafana dashboard JSON, TestData CSV Content, and dashboard field thresholds.

## System architecture

`Synthetic fixture → tool configuration → focused check or report`

## Features

- Visualize synthetic readings with the built-in TestData source.
- Demonstrate usability improvement with an explicit local sample.

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
  "uid": "portfolio-043025",
  "title": "Units in stock | BRANCH-08 (043025)",
  "schemaVersion": 39,
  "version": 1,
  "tags": [
    "portfolio",
    "inventory",
    "summary"
  ],
  "panels": [
    {
      "id": 1,
      "title": "Units in stock",
      "type": "stat",
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
          "csvContent": "asset,metric,source,reading,threshold\nBRANCH-08,units in stock,primary,2,4\n"
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
                      "color": "red",
                      "value": null
                    },
                    {
                      "color": "green",
                      "value": 4
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
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "/^reading$/",
          "values": false
        },
        "colorMode": "value",
        "graphMode": "none"
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
  "embedded_csv": "asset,metric,source,reading,threshold\nBRANCH-08,units in stock,primary,2,4\n"
}
```

## Execution instructions

Add a **TestData** data source to your local Grafana instance. Save the source as `dashboard.json`, import it as a dashboard, and select your TestData data source when prompted.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "rows": 1,
  "source": "primary",
  "readings": [
    2
  ],
  "threshold_step": 4
}
```

## Testing and validation

Parsed the dashboard JSON and independently checked the CSV values, source labels, and threshold configuration. Grafana import and rendering remain unverified.

```json
{
  "rows": 1,
  "source": "primary",
  "readings": [
    2
  ],
  "threshold_step": 4
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

Practice inventory using Grafana and check a repeatable usability improvement workflow in the branch office simulator scenario.

## Conclusion

The project records an observable inventory result for training case `BRANCH-08`.

[Back to the Grafana project index](../README.md)
