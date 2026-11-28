# Nginx: Dashboard — Usability Improvement for Two-Node Lab (Project 041390)

## Project overview

A two-node lab simulation provides the setting for this dashboard exercise. This project includes source code, sample data, and instructions to run it. The observations are synthetic.

## Problem statement

The two-node lab setting needs a reproducible way to check active tickets.

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
        location = /check-041390.json {
            return 200 '{"project":"041390","asset":"NODE-B","metric":"active tickets","source":"primary","readings":1,"flagged":1}';
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
  "project": "041390",
  "asset": "NODE-B",
  "metric": "active tickets",
  "limit": 12,
  "comparison": "gt",
  "finding": "queue above capacity",
  "mode": "summary",
  "primary": [
    18
  ],
  "fallback": []
}
```

## Execution instructions

Save the source in a fresh practice directory as `serve.sh`. Run `sh serve.sh` and keep it in the foreground. From a second terminal in the same environment run `curl -fsS http://127.0.0.1:18080/check-041390.json`. Stop the local server with Ctrl+C.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "project": "041390",
  "asset": "NODE-B",
  "metric": "active tickets",
  "source": "primary",
  "readings": 1,
  "flagged": 1
}
```

## Testing and validation

Ran the native Nginx configuration check, started the isolated loopback server, and verified the exact JSON response and the unknown-path HTTP 404.

```json
{
  "project": "041390",
  "asset": "NODE-B",
  "metric": "active tickets",
  "source": "primary",
  "readings": 1,
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

Practice dashboard using Nginx and check a repeatable usability improvement workflow in the two-node lab scenario.

## Conclusion

The project records an observable dashboard result for training case `NODE-B`.

[Back to the Nginx project index](../README.md)
