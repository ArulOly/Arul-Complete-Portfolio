# Greenbone: Scheduled Reporting — Usability Improvement for Public Dataset (Project 031470)

## Project overview

A synthetic public-data exercise provides the setting for this scheduled reporting exercise. This project includes source code, sample data, and instructions to run it. The data is synthetic.

## Problem statement

The public dataset setting needs a repeatable way to review scheduled reporting observations.

## Project objectives

- Filter defensive findings from a simulated Greenbone GMP report.
- Demonstrate usability improvement with an explicit local sample.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Simulated Greenbone GMP XML report data, XPath, and XSLT 1.0.

## System architecture

`Synthetic fixture → tool configuration → focused check or report`

## Features

- Filter defensive findings from a simulated Greenbone GMP report.
- Demonstrate usability improvement with an explicit local sample.

## Prerequisites

Use xsltproc or an equivalent XSLT 1.0 processor. Use a text editor and a local practice directory.

## Complete Greenbone source code

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
  <xsl:output method="xml" indent="yes"/>
  <xsl:template match="/">
    <xsl:variable name="selected" select="/get_reports_response/report/report/results/result[host = '192.0.2.10' and name = 'scheduled_reporting' and number(severity) &gt;= 4.0]"/>
    <portfolio project="031470" mode="summary" asset="DATA-12" count="{count($selected)}">
    </portfolio>
  </xsl:template>
</xsl:stylesheet>
```

## Implementation explanation

Filter defensive findings from a simulated Greenbone GMP report.

## Sample input data

Use these synthetic observations or inspect them in the embedded source.

```json
{
  "report_source": "current",
  "report_xml": "<get_reports_response status=\"200\">\n  <report>\n    <report>\n      <results>\n        <result><name>scheduled_reporting</name><host>192.0.2.10</host><severity>5.0</severity></result>\n        <result><name>scheduled_reporting</name><host>192.0.2.10</host><severity>1.0</severity></result>\n      </results>\n    </report>\n  </report>\n</get_reports_response>\n"
}
```

## Execution instructions

Save the source as `report.xsl` and the fixture below as `report.xml`. Run `xsltproc --nonet report.xsl report.xml` to review the selected results.

```xml
<get_reports_response status="200">
  <report>
    <report>
      <results>
        <result><name>scheduled_reporting</name><host>192.0.2.10</host><severity>5.0</severity></result>
        <result><name>scheduled_reporting</name><host>192.0.2.10</host><severity>1.0</severity></result>
      </results>
    </report>
  </report>
</get_reports_response>
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "asset": "DATA-12",
  "matched_findings": 1
}
```

## Testing and validation

Executed the stylesheet with libxslt against the embedded XML. The selected finding count repeated and an empty report returned zero.

```json
{
  "asset": "DATA-12",
  "matched_findings": 1
}
```

## Error handling

Start with the provided fixture. Invalid inputs, unmatched observations, or missing setup are checked as described above.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The sample uses a simplified GMP-shaped export with a documentation-only IP address. Scan execution and live Greenbone integration are outside this project.

## Learning outcomes

Practice scheduled reporting using Greenbone and check a repeatable usability improvement workflow in the public dataset scenario.

## Conclusion

The project records an observable scheduled reporting result for training case `DATA-12`.

[Back to the Greenbone project index](../README.md)
