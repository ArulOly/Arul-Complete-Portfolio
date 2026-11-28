# Fedora: Inventory — Baseline Setup for School Lab (Project 004001)

## Project overview

Shared classroom equipment provides the setting for this inventory exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The school lab needs a repeatable inventory baseline that another student can verify.

## Project objectives

- Record the school lab inventory baseline.
- Report the installed version using the platform's native inventory tools.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

POSIX shell and rpm.

## System architecture

`Package name → platform check → native package database → terminal report`

## Features

- Record the school lab inventory baseline.
- Report the installed version using the platform's native inventory tools.

## Prerequisites

Use Fedora with its standard package database and rpm. Use a text editor and a local practice directory.

## Complete Fedora source code

```sh
#!/bin/sh
set -eu

if [ "$#" -ne 1 ]; then
    echo "Usage: sh inventory.sh PACKAGE" >&2
    exit 2
fi

package=$1
case "$package" in
    ""|-*|*[!A-Za-z0-9_.+-]*)
        echo "Use one unqualified package name" >&2
        exit 2 ;;
esac

if ! { [ -r /etc/os-release ] && ( . /etc/os-release; [ "$ID" = fedora ] ); }; then
    echo "Run this inventory check on Fedora" >&2
    exit 2
fi

if ! version=$(rpm -q --queryformat '%{VERSION}-%{RELEASE}' -- "$package" 2>/dev/null) || [ -z "$version" ]; then
    echo "Package not found: $package" >&2
    exit 1
fi

printf 'package=%s\nversion=%s\n' "$package" "$version"
```

## Implementation explanation

Report the installed version using the platform's native inventory tools.

## Sample input data

Use this example input.

```json
{
  "package": "rpm"
}
```

## Execution instructions

Save the block as `inventory.sh`, then run `sh inventory.sh rpm` in the practice environment.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "package": "rpm",
  "version": "6.0.2-1.fc43"
}
```

## Testing and validation

Executed the embedded script on a Fedora 43 container. A repeat matched the report. A missing package returned status 1, and a malformed name returned status 2.

```json
{
  "package": "rpm",
  "version": "6.0.2-1.fc43"
}
```

## Error handling

Invalid names return status 2. An unavailable package returns status 1. The package database is read only.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

It reads one installed item at a time. Installed versions vary by system and update level.

## Learning outcomes

Practice inventory using Fedora and check a repeatable baseline setup workflow in the school lab scenario.

## Conclusion

The project records an observable inventory result for training case `LAB-17`.

[Back to the Fedora project index](../README.md)
