# Arduino: Configuration Comparison — Recovery for Single-Host Lab (Project 037683)

## Project overview

One simulated lab host provides the setting for this configuration comparison exercise. This project includes source code, sample data, and instructions to run it. The observations are synthetic.

## Problem statement

The single-host lab setting needs a reproducible way to check changed settings.

## Project objectives

- Check changed settings against the sample threshold.
- Use the sample fallback or inspect the recovered file.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

An Arduino-compatible C++ sketch and Serial Monitor at 9600 baud.

## System architecture

`Synthetic fixture → native tool → reported result`

## Features

- Check changed settings against the sample threshold.
- Use the sample fallback or inspect the recovered file.

## Prerequisites

Use Arduino IDE and a supported Arduino board for an optional board run. Use a text editor and a local practice directory.

## Complete Arduino source code

```cpp
#include <Arduino.h>

// Fixed synthetic observations for changed settings.
constexpr unsigned values[] = {2, 0};
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
  Serial.print("source=fallback readings=");
  Serial.print(static_cast<unsigned>(count));
  Serial.print(" flagged=");
  Serial.println(flagged);
}

void loop() {
  // Run once at startup so the Serial Monitor keeps the result readable.
}
```

## Implementation explanation

Check changed settings against the sample threshold. Use the sample fallback or inspect the recovered file.

## Sample input data

The code already includes these synthetic observations.

```json
{
  "project": "037683",
  "asset": "HOST-03",
  "metric": "changed settings",
  "limit": 0,
  "comparison": "gt",
  "finding": "configuration drift found",
  "mode": "recovery",
  "primary": [],
  "fallback": [
    2,
    0
  ]
}
```

## Execution instructions

Save the source as an Arduino sketch, choose the matching board and port, and open Serial Monitor at 9600 baud after uploading. The sample is already in the array.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "source": "fallback",
  "readings": 2,
  "flagged": 1
}
```

## Testing and validation

Compiled and ran the sketch as host C++ using a small Serial test double. The calculated output matched the expected values. Arduino core compilation and hardware execution remain unverified.

```json
{
  "source": "fallback",
  "readings": 2,
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

Practice configuration comparison using Arduino and check a repeatable recovery workflow in the single-host lab scenario.

## Conclusion

The project records an observable configuration comparison result for training case `HOST-03`.

[Back to the Arduino project index](../README.md)
