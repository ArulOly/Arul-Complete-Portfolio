# Arduino: Alerting — Usability Improvement for Home Development (Project 037415)

## Project overview

A home development workspace provides the setting for this alerting exercise. This project includes source code, sample data, and instructions to run it. The observations are synthetic.

## Problem statement

The home development setting needs a reproducible way to check consecutive breach readings.

## Project objectives

- Check consecutive breach readings against the sample threshold.
- Return a compact result that is easy to inspect.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

An Arduino-compatible C++ sketch and Serial Monitor at 9600 baud.

## System architecture

`Synthetic fixture → native tool → reported result`

## Features

- Check consecutive breach readings against the sample threshold.
- Return a compact result that is easy to inspect.

## Prerequisites

Use Arduino IDE and a supported Arduino board for an optional board run. Use a text editor and a local practice directory.

## Complete Arduino source code

```cpp
#include <Arduino.h>

// Fixed synthetic observations for consecutive breach readings.
constexpr unsigned values[] = {3};
constexpr unsigned threshold = 3;
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
}

void loop() {
  // Run once at startup so the Serial Monitor keeps the result readable.
}
```

## Implementation explanation

Check consecutive breach readings against the sample threshold. Return a compact result that is easy to inspect.

## Sample input data

The code already includes these synthetic observations.

```json
{
  "project": "037415",
  "asset": "DEV-05",
  "metric": "consecutive breach readings",
  "limit": 3,
  "comparison": "ge",
  "finding": "alert threshold reached",
  "mode": "summary",
  "primary": [
    3
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
  "readings": 1,
  "flagged": 1
}
```

## Testing and validation

Compiled and ran the sketch as host C++ using a small Serial test double. The calculated output matched the expected values. Arduino core compilation and hardware execution remain unverified.

```json
{
  "source": "primary",
  "readings": 1,
  "flagged": 1
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

Practice alerting using Arduino and check a repeatable usability improvement workflow in the home development scenario.

## Conclusion

The project records an observable alerting result for training case `DEV-05`.

[Back to the Arduino project index](../README.md)
