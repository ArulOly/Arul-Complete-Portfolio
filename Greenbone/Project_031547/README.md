# Greenbone: Access Review — Automation for Portfolio Demo (Project 031547)

## Project overview

A repeatable portfolio demonstration provides the setting for this access review exercise. This project includes source code, sample data, and instructions to run it. The data is synthetic.

## Problem statement

The portfolio demo setting needs a repeatable way to review access review observations.

## Project objectives

- Filter defensive findings from a simulated Greenbone GMP report.
- Demonstrate automation with an explicit local sample.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Simulated Greenbone GMP XML report data, XPath, and XSLT 1.0.

## System architecture

`Synthetic fixture → tool configuration → focused check or report`

## Features

- Filter defensive findings from a simulated Greenbone GMP report.
- Demonstrate automation with an explicit local sample.

## Prerequisites

Use xsltproc or an equivalent XSLT 1.0 processor. Use a text editor and a local practice directory.

## Complete Greenbone source code

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
  <xsl:output method="xml" indent="yes"/>
  <xsl:template match="/">
    <xsl:variable name="selected" select="/get_reports_response/report/report/results/result[host = '192.0.2.17' and name = 'access_review' and number(severity) &gt;= 4.0]"/>
    <portfolio project="031547" mode="batch" asset="DEMO-02" count="{count($selected)}">
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
  "report_xml": "<get_reports_response status=\"200\">\n  <report>\n    <report>\n      <results>\n        <result><name>access_review</name><host>192.0.2.17</host><severity>5.0</severity></result><result><name>access_review</name><host>192.0.2.17</host><severity>5.0</severity></result>\n        <result><name>access_review</name><host>192.0.2.17</host><severity>1.0</severity></result>\n      </results>\n    </report>\n  </report>\n</get_reports_response>\n"
}
```

## Execution instructions

Save the source as `report.xsl` and the fixture below as `report.xml`. Run `xsltproc --nonet report.xsl report.xml` to review the selected results.

```xml
<get_reports_response status="200">
  <report>
    <report>
      <results>
        <result><name>access_review</name><host>192.0.2.17</host><severity>5.0</severity></result><result><name>access_review</name><host>192.0.2.17</host><severity>5.0</severity></result>
        <result><name>access_review</name><host>192.0.2.17</host><severity>1.0</severity></result>
      </results>
    </report>
  </report>
</get_reports_response>
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "asset": "DEMO-02",
  "matched_findings": 2
}
```

## Testing and validation

Executed the stylesheet with libxslt against the embedded XML. The selected finding count repeated and an empty report returned zero.

```json
{
  "asset": "DEMO-02",
  "matched_findings": 2
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

Practice access review using Greenbone and check a repeatable automation workflow in the portfolio demo scenario.

## Conclusion

The project records an observable access review result for training case `DEMO-02`.

[Back to the Greenbone project index](../README.md)
