# Wireshark: Scheduled Reporting — Recovery for Public Dataset (Project 029468)

## Project overview

A synthetic public-data exercise provides the setting for this scheduled reporting exercise. This project includes source code, sample data, and instructions to run it. The data is synthetic.

## Problem statement

The public dataset setting needs a repeatable way to review scheduled reporting observations.

## Project objectives

- Use a display filter to isolate marked packets in an offline capture.
- Demonstrate recovery with an explicit local sample.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Wireshark display filters, TShark, text2pcap, and synthetic UDP capture fixtures.

## System architecture

`Synthetic fixture → tool configuration → focused check or report`

## Features

- Use a display filter to isolate marked packets in an offline capture.
- Demonstrate recovery with an explicit local sample.

## Prerequisites

Use a Wireshark CLI install that includes text2pcap and TShark. Use a text editor and a local practice directory.

## Complete Wireshark source code

```sh
#!/bin/sh
set -eu
work=$(mktemp -d)
trap 'rm -rf "$work"' EXIT
cat >"$work/packets.txt" <<'PACKETS'
000000 70 6f 72 74 66 6f 6c 69 6f 3a 73 63 68 65 64 75 6c 65 64 5f 72 65 70 6f 72 74 69 6e 67 3a 72 65 63 6f 76 65 72 79

000000 75 6e 72 65 6c 61 74 65 64 3a 63 6f 6e 74 72 6f 6c

000000 70 6f 72 74 66 6f 6c 69 6f 3a 73 63 68 65 64 75 6c 65 64 5f 72 65 70 6f 72 74 69 6e 67 3a 72 65 63 6f 76 65 72 79
PACKETS
text2pcap -q -u 12345,24467 "$work/packets.txt" "$work/practice.pcapng"
tshark -n -r "$work/practice.pcapng" -Y 'udp.dstport == 24467 && udp contains "portfolio:scheduled_reporting:recovery"' -T fields -e frame.number
```

## Implementation explanation

Use a display filter to isolate marked packets in an offline capture.

## Sample input data

Use these synthetic observations or inspect them in the embedded source.

```json
{
  "destination_port": 24467,
  "payloads": [
    "portfolio:scheduled_reporting:recovery",
    "unrelated:control",
    "portfolio:scheduled_reporting:recovery"
  ],
  "display_filter": "udp.dstport == 24467 && udp contains \"portfolio:scheduled_reporting:recovery\""
}
```

## Execution instructions

Save the source as `practice.sh` and run `sh practice.sh`. The script generates the temporary capture from embedded text and prints the selected frame numbers.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "matched_frames": [
    1,
    3
  ]
}
```

## Testing and validation

Generated and read the offline capture with Wireshark CLI tools. The display filter returned the expected frame numbers and excluded the control frame.

```json
{
  "matched_frames": [
    1,
    3
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

Practice scheduled reporting using Wireshark and check a repeatable recovery workflow in the public dataset scenario.

## Conclusion

The project records an observable scheduled reporting result for training case `DATA-12`.

[Back to the Wireshark project index](../README.md)
