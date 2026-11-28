# Nginx: Configuration Comparison — Recovery for School Lab (Project 041653)

## Project overview

Shared classroom equipment provides the setting for this configuration comparison exercise. This project includes source code, sample data, and instructions to run it. The observations are synthetic.

## Problem statement

The school lab setting needs a reproducible way to check changed settings.

## Project objectives

- Expose a prepared sample report at an exact loopback-only Nginx route.
- Use the guide focus to label the contents of the static report.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Native Nginx configuration and a loopback HTTP endpoint.

## System architecture

`Synthetic fixture → native tool → reported result`

## Features

- Expose a prepared sample report at an exact loopback-only Nginx route.
- Use the guide focus to label the contents of the static report.

## Prerequisites

Use a disposable Alpine Linux environment with nginx and curl. Use a text editor and a local practice directory.

## Complete Nginx source code

```sh
#!/bin/sh
set -eu
mkdir -p work site
cat >work/practice.conf <<'CONFIG'
pid work/nginx.pid;
error_log stderr warn;
events {}
http {
    access_log off;
    default_type application/json;
    server {
        listen 127.0.0.1:18080;
        server_name localhost;
        location = /check-041653.json {
            return 200 '{"project":"041653","asset":"LAB-17","metric":"changed settings","source":"fallback","readings":2,"flagged":1}';
        }
        location / {
            return 404;
        }
    }
}
CONFIG
nginx -e stderr -t -p "$PWD/" -c "$PWD/work/practice.conf"
exec nginx -e stderr -p "$PWD/" -c "$PWD/work/practice.conf" -g "daemon off;"
```

## Implementation explanation

Expose a prepared sample report at an exact loopback-only Nginx route. Use the guide focus to label the contents of the static report.

## Sample input data

The code already includes these synthetic observations.

```json
{
  "project": "041653",
  "asset": "LAB-17",
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

Save the source in a fresh practice directory as `serve.sh`. Run `sh serve.sh` and keep it in the foreground. From a second terminal in the same environment run `curl -fsS http://127.0.0.1:18080/check-041653.json`. Stop the local server with Ctrl+C.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "project": "041653",
  "asset": "LAB-17",
  "metric": "changed settings",
  "source": "fallback",
  "readings": 2,
  "flagged": 1
}
```

## Testing and validation

Ran the native Nginx configuration check, started the isolated loopback server, and verified the exact JSON response and the unknown-path HTTP 404.

```json
{
  "project": "041653",
  "asset": "LAB-17",
  "metric": "changed settings",
  "source": "fallback",
  "readings": 2,
  "flagged": 1
}
```

## Error handling

The syntax check reports configuration errors. Requests for unknown paths return HTTP 404.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The server publishes a static sample report on loopback. Network exposure and production deployment are outside the exercise.

## Learning outcomes

Practice configuration comparison using Nginx and check a repeatable recovery workflow in the school lab scenario.

## Conclusion

The project records an observable configuration comparison result for training case `LAB-17`.

[Back to the Nginx project index](../README.md)
