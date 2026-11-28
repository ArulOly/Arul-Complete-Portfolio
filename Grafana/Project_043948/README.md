# Grafana: Visual Reporting — Recovery for Portfolio Demo (Project 043948)

## Project overview

A repeatable portfolio demonstration provides the setting for this visual reporting exercise. This project includes source code, sample data, and instructions to run it. The data is synthetic.

## Problem statement

The portfolio demo setting needs a repeatable way to review visual reporting observations.

## Project objectives

- Visualize synthetic readings with the built-in TestData source.
- Demonstrate recovery with an explicit local sample.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Grafana dashboard JSON, TestData CSV Content, and dashboard field thresholds.

## System architecture

`Synthetic fixture → tool configuration → focused check or report`

## Features

- Visualize synthetic readings with the built-in TestData source.
- Demonstrate recovery with an explicit local sample.

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
  "uid": "portfolio-043948",
  "title": "Summary categories | DEMO-02 (043948)",
  "schemaVersion": 39,
  "version": 1,
  "tags": [
    "portfolio",
    "visual_reporting",
    "recovery"
  ],
  "panels": [
    {
      "id": 1,
      "title": "Summary categories",
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
          "csvContent": "asset,metric,source,reading,threshold\nDEMO-02,summary categories,fallback,6,5\nDEMO-02,summary categories,fallback,5,5\n"
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
                      "value": 6
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
  "embedded_csv": "asset,metric,source,reading,threshold\nDEMO-02,summary categories,fallback,6,5\nDEMO-02,summary categories,fallback,5,5\n"
}
```

## Execution instructions

Add a **TestData** data source to your local Grafana instance. Save the source as `dashboard.json`, import it as a dashboard, and select your TestData data source when prompted.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "rows": 2,
  "source": "fallback",
  "readings": [
    6,
    5
  ],
  "threshold_step": 6
}
```

## Testing and validation

Parsed the dashboard JSON and independently checked the CSV values, source labels, and threshold configuration. Grafana import and rendering remain unverified.

```json
{
  "rows": 2,
  "source": "fallback",
  "readings": [
    6,
    5
  ],
  "threshold_step": 6
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

Practice visual reporting using Grafana and check a repeatable recovery workflow in the portfolio demo scenario.

## Conclusion

The project records an observable visual reporting result for training case `DEMO-02`.

[Back to the Grafana project index](../README.md)
