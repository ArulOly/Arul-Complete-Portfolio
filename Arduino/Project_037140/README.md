# Arduino: Audit — Usability Improvement for Two-Node Lab (Project 037140)

## Project overview

A two-node lab simulation provides the setting for this audit exercise. This project includes source code, sample data, and instructions to run it. The observations are synthetic.

## Problem statement

The two-node lab setting needs a reproducible way to check days since last review.

## Project objectives

- Check days since last review against the sample threshold.
- Return a compact result that is easy to inspect.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

An Arduino-compatible C++ sketch and Serial Monitor at 9600 baud.

## System architecture

`Synthetic fixture → native tool → reported result`

## Features

- Check days since last review against the sample threshold.
- Return a compact result that is easy to inspect.

## Prerequisites

Use Arduino IDE and a supported Arduino board for an optional board run. Use a text editor and a local practice directory.

## Complete Arduino source code

```cpp
#include <Arduino.h>

// Fixed synthetic observations for days since last review.
constexpr unsigned values[] = {45};
constexpr unsigned threshold = 30;
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
}

void loop() {
  // Run once at startup so the Serial Monitor keeps the result readable.
}
```

## Implementation explanation

Check days since last review against the sample threshold. Return a compact result that is easy to inspect.

## Sample input data

The code already includes these synthetic observations.

```json
{
  "project": "037140",
  "asset": "NODE-B",
  "metric": "days since last review",
  "limit": 30,
  "comparison": "gt",
  "finding": "review overdue",
  "mode": "summary",
  "primary": [
    45
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

Practice audit using Arduino and check a repeatable usability improvement workflow in the two-node lab scenario.

## Conclusion

The project records an observable audit result for training case `NODE-B`.

[Back to the Arduino project index](../README.md)
