# Arduino: Data Validation — Reporting for Portfolio Demo (Project 037799)

## Project overview

A repeatable portfolio demonstration provides the setting for this data validation exercise. This project includes source code, sample data, and instructions to run it. The observations are synthetic.

## Problem statement

The portfolio demo setting needs a reproducible way to check invalid input rows.

## Project objectives

- Check invalid input rows against the sample threshold.
- Produce a report for the supplied observations.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

An Arduino-compatible C++ sketch and Serial Monitor at 9600 baud.

## System architecture

`Synthetic fixture → native tool → reported result`

## Features

- Check invalid input rows against the sample threshold.
- Produce a report for the supplied observations.

## Prerequisites

Use Arduino IDE and a supported Arduino board for an optional board run. Use a text editor and a local practice directory.

## Complete Arduino source code

```cpp
#include <Arduino.h>

// Fixed synthetic observations for invalid input rows.
constexpr unsigned values[] = {1, 0};
constexpr unsigned threshold = 0;
constexpr size_t count = sizeof(values) / sizeof(values[0]);

void setup() {
  Serial.begin(9600);
  unsigned flagged = 0;
  for (size_t index = 0; index < count; ++index) {
    if (values[index] > threshold) {
      ++flagged;
    }
  }
  Serial.print("source=primary readings=");
  Serial.print(static_cast<unsigned>(count));
  Serial.print(" flagged=");
  Serial.println(flagged);
  for (size_t i = 0; i < count; ++i) {
    Serial.print("reading=");
    Serial.println(values[i]);
  }
}

void loop() {
  // Run once at startup so the Serial Monitor keeps the result readable.
}
```

## Implementation explanation

Check invalid input rows against the sample threshold. Produce a report for the supplied observations.

## Sample input data

The code already includes these synthetic observations.

```json
{
  "project": "037799",
  "asset": "DEMO-02",
  "metric": "invalid input rows",
  "limit": 0,
  "comparison": "gt",
  "finding": "validation failed",
  "mode": "report",
  "primary": [
    1,
    0
  ],
  "fallback": []
}
```

## Execution instructions

Save the source as an Arduino sketch, choose the matching board and port, and open Serial Monitor at 9600 baud after uploading. The sample is already in the array.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "source": "primary",
  "readings": 2,
  "flagged": 1,
  "values": [
    1,
    0
  ]
}
```

## Testing and validation

Compiled and ran the sketch as host C++ using a small Serial test double. The calculated output matched the expected values. Arduino core compilation and hardware execution remain unverified.

```json
{
  "source": "primary",
  "readings": 2,
  "flagged": 1,
  "values": [
    1,
    0
  ]
}
```

## Error handling

The fixture contains only unsigned integers known at compile time.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The sketch uses synthetic readings and produces a one-time serial report. No external wiring is required.

## Learning outcomes

Practice data validation using Arduino and check a repeatable reporting workflow in the portfolio demo scenario.

## Conclusion

The project records an observable data validation result for training case `DEMO-02`.

[Back to the Arduino project index](../README.md)
