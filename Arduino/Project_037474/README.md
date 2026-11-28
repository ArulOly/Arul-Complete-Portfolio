# Arduino: Scheduled Reporting — Reporting for Branch Office Simulator (Project 037474)

## Project overview

A simulated branch office provides the setting for this scheduled reporting exercise. This project includes source code, sample data, and instructions to run it. The observations are synthetic.

## Problem statement

The branch office simulator setting needs a reproducible way to check days since last report.

## Project objectives

- Check days since last report against the sample threshold.
- Produce a report for the supplied observations.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

An Arduino-compatible C++ sketch and Serial Monitor at 9600 baud.

## System architecture

`Synthetic fixture → native tool → reported result`

## Features

- Check days since last report against the sample threshold.
- Produce a report for the supplied observations.

## Prerequisites

Use Arduino IDE and a supported Arduino board for an optional board run. Use a text editor and a local practice directory.

## Complete Arduino source code

```cpp
#include <Arduino.h>

// Fixed synthetic observations for days since last report.
constexpr unsigned values[] = {8, 6};
constexpr unsigned threshold = 7;
constexpr size_t count = sizeof(values) / sizeof(values[0]);

void setup() {
  Serial.begin(9600);
  unsigned flagged = 0;
  for (size_t index = 0; index < count; ++index) {
    if (values[index] >= threshold) {
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

Check days since last report against the sample threshold. Produce a report for the supplied observations.

## Sample input data

The code already includes these synthetic observations.

```json
{
  "project": "037474",
  "asset": "BRANCH-08",
  "metric": "days since last report",
  "limit": 7,
  "comparison": "ge",
  "finding": "report due",
  "mode": "report",
  "primary": [
    8,
    6
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
    8,
    6
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
    8,
    6
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

Practice scheduled reporting using Arduino and check a repeatable reporting workflow in the branch office simulator scenario.

## Conclusion

The project records an observable scheduled reporting result for training case `BRANCH-08`.

[Back to the Arduino project index](../README.md)
