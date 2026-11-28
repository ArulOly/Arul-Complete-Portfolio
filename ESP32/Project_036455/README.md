# ESP32: Scheduled Reporting — Usability Improvement for School Lab (Project 036455)

## Project overview

Shared classroom equipment provides the setting for this scheduled reporting exercise. This project includes source code, sample data, and instructions to run it. The observations are synthetic.

## Problem statement

The school lab setting needs a reproducible way to check days since last report.

## Project objectives

- Check days since last report against the sample threshold.
- Return a compact result that is easy to inspect.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Arduino-ESP32 sketch, C++, and Serial output at 115200 baud.

## System architecture

`Synthetic fixture → native tool → reported result`

## Features

- Check days since last report against the sample threshold.
- Return a compact result that is easy to inspect.

## Prerequisites

Use Arduino IDE with ESP32 board support for an optional board run. Use a text editor and a local practice directory.

## Complete ESP32 source code

```cpp
#include <Arduino.h>

// Fixed synthetic observations for days since last report.
constexpr unsigned values[] = {8};
constexpr unsigned threshold = 7;
constexpr size_t count = sizeof(values) / sizeof(values[0]);

void setup() {
  Serial.begin(115200);
  unsigned flagged = 0;
  for (size_t index = 0; index < count; ++index) {
    if (values[index] >= threshold) {
      ++flagged;
    }
  }
  Serial.printf("source=primary readings=%u flagged=%u\n",
                static_cast<unsigned>(count), flagged);
}

void loop() {
  // Run once at startup so the Serial Monitor keeps the result readable.
}
```

## Implementation explanation

Check days since last report against the sample threshold. Return a compact result that is easy to inspect.

## Sample input data

The code already includes these synthetic observations.

```json
{
  "project": "036455",
  "asset": "LAB-17",
  "metric": "days since last report",
  "limit": 7,
  "comparison": "ge",
  "finding": "report due",
  "mode": "summary",
  "primary": [
    8
  ],
  "fallback": []
}
```

## Execution instructions

Save the source as an Arduino sketch, choose your ESP32 board and port, and open Serial Monitor at 115200 baud after uploading. The sample is already in the array.

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

Compiled and ran the sketch as host C++ using a Serial test double. The calculated output matched the expected values. ESP32 toolchain compilation and hardware execution remain unverified.

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

Practice scheduled reporting using ESP32 and check a repeatable usability improvement workflow in the school lab scenario.

## Conclusion

The project records an observable scheduled reporting result for training case `LAB-17`.

[Back to the ESP32 project index](../README.md)
