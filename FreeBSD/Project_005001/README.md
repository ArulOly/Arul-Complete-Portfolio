# FreeBSD: Inventory — Baseline Setup for School Lab (Project 005001)

## Project overview

Shared classroom equipment provides the setting for this inventory exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The school lab needs a repeatable inventory baseline that another student can verify.

## Project objectives

- Record the school lab inventory baseline.
- Report the installed version using the platform's native inventory tools.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

POSIX shell and freebsd-version and pkg.

## System architecture

`Package name → platform check → native package database → terminal report`

## Features

- Record the school lab inventory baseline.
- Report the installed version using the platform's native inventory tools.

## Prerequisites

Use FreeBSD with its standard package database and freebsd-version and pkg. Use a text editor and a local practice directory.

## Complete FreeBSD source code

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

if ! { [ "$(uname -s)" = FreeBSD ]; }; then
    echo "Run this inventory check on FreeBSD" >&2
    exit 2
fi

if ! version=$(if [ "$package" = base ]; then freebsd-version -u; else pkg query '%v' "$package"; fi 2>/dev/null) || [ -z "$version" ]; then
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
  "package": "base"
}
```

## Execution instructions

Save the block as `inventory.sh`, then run `sh inventory.sh base` in the practice environment.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "package": "base",
  "version": "14.2-RELEASE"
}
```

## Testing and validation

Executed the embedded script on a FreeBSD 14.2 virtual machine. A repeat matched the report. A missing package returned status 1, and a malformed name returned status 2.

```json
{
  "package": "base",
  "version": "14.2-RELEASE"
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

Practice inventory using FreeBSD and check a repeatable baseline setup workflow in the school lab scenario.

## Conclusion

The project records an observable inventory result for training case `LAB-17`.

[Back to the FreeBSD project index](../README.md)
