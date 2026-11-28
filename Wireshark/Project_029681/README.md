# Wireshark: Configuration Comparison — Baseline Setup for Single-Host Lab (Project 029681)

## Project overview

One simulated lab host provides the setting for this configuration comparison exercise. This project includes source code, sample data, and instructions to run it. The data is synthetic.

## Problem statement

The single-host lab setting needs a repeatable way to review configuration comparison observations.

## Project objectives

- Use a display filter to isolate marked packets in an offline capture.
- Demonstrate baseline setup with an explicit local sample.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Wireshark display filters, TShark, text2pcap, and synthetic UDP capture fixtures.

## System architecture

`Synthetic fixture → tool configuration → focused check or report`

## Features

- Use a display filter to isolate marked packets in an offline capture.
- Demonstrate baseline setup with an explicit local sample.

## Prerequisites

Use a Wireshark CLI install that includes text2pcap and TShark. Use a text editor and a local practice directory.

## Complete Wireshark source code

```sh
#!/bin/sh
set -eu
work=$(mktemp -d)
trap 'rm -rf "$work"' EXIT
cat >"$work/packets.txt" <<'PACKETS'
000000 70 6f 72 74 66 6f 6c 69 6f 3a 63 6f 6e 66 69 67 75 72 61 74 69 6f 6e 5f 63 6f 6d 70 61 72 69 73 6f 6e 3a 62 61 73 65 6c 69 6e 65

000000 75 6e 72 65 6c 61 74 65 64 3a 63 6f 6e 74 72 6f 6c
PACKETS
text2pcap -q -u 12345,24680 "$work/packets.txt" "$work/practice.pcapng"
tshark -n -r "$work/practice.pcapng" -Y 'udp.dstport == 24680 && udp contains "portfolio:configuration_comparison:baseline"' -T fields -e frame.number
```

## Implementation explanation

Use a display filter to isolate marked packets in an offline capture.

## Sample input data

Use these synthetic observations or inspect them in the embedded source.

```json
{
  "destination_port": 24680,
  "payloads": [
    "portfolio:configuration_comparison:baseline",
    "unrelated:control"
  ],
  "display_filter": "udp.dstport == 24680 && udp contains \"portfolio:configuration_comparison:baseline\""
}
```

## Execution instructions

Save the source as `practice.sh` and run `sh practice.sh`. The script generates the temporary capture from embedded text and prints the selected frame numbers.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "matched_frames": [
    1
  ]
}
```

## Testing and validation

Generated and read the offline capture with Wireshark CLI tools. The display filter returned the expected frame numbers and excluded the control frame.

```json
{
  "matched_frames": [
    1
  ]
}
```

## Error handling

Start with the provided fixture. Invalid inputs, unmatched observations, or missing setup are checked as described above.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The packets are made from text inside a temporary directory. No packets are sent or captured from a live interface.

## Learning outcomes

Practice configuration comparison using Wireshark and check a repeatable baseline setup workflow in the single-host lab scenario.

## Conclusion

The project records an observable configuration comparison result for training case `HOST-03`.

[Back to the Wireshark project index](../README.md)
