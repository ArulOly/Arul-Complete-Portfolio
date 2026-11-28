# Apache: Recovery Test — Recovery for Branch Office Simulator (Project 042973)

## Project overview

A simulated branch office provides the setting for this recovery test exercise. This project includes source code, sample data, and instructions to run it. The observations are synthetic.

## Problem statement

The branch office simulator setting needs a reproducible way to check unrestored records.

## Project objectives

- Serve a prepared sample report through a loopback-only Apache configuration.
- Use the guide focus to label the contents of the static report.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Native Apache httpd configuration, static files, and a loopback HTTP endpoint.

## System architecture

`Synthetic fixture → native tool → reported result`

## Features

- Serve a prepared sample report through a loopback-only Apache configuration.
- Use the guide focus to label the contents of the static report.

## Prerequisites

Use a disposable Alpine Linux container with apache2 and curl. Use a text editor and a local practice directory.

## Complete Apache source code

```sh
#!/bin/sh
set -eu
mkdir -p work site
printf '%s\n' '{"project":"042973","asset":"BRANCH-08","metric":"unrestored records","source":"fallback","readings":2,"flagged":1}' > site/check-042973.json
chmod 755 . work site
chmod 644 site/check-042973.json
cat >work/practice.conf <<'CONFIG'
Listen 127.0.0.1:18080
ServerName localhost
PidFile work/httpd.pid
ErrorLog /dev/stderr
LoadModule mpm_event_module /usr/lib/apache2/mod_mpm_event.so
LoadModule authz_core_module /usr/lib/apache2/mod_authz_core.so
LoadModule unixd_module /usr/lib/apache2/mod_unixd.so
User apache
Group apache
DocumentRoot site
<Directory "site">
    AllowOverride None
    Require all granted
    <FilesMatch "\.json$">
        ForceType application/json
    </FilesMatch>
</Directory>
CONFIG
httpd -t -d "$PWD" -f "$PWD/work/practice.conf"
exec httpd -DFOREGROUND -d "$PWD" -f "$PWD/work/practice.conf"
```

## Implementation explanation

Serve a prepared sample report through a loopback-only Apache configuration. Use the guide focus to label the contents of the static report.

## Sample input data

The code already includes these synthetic observations.

```json
{
  "project": "042973",
  "asset": "BRANCH-08",
  "metric": "unrestored records",
  "limit": 0,
  "comparison": "gt",
  "finding": "recovery remains incomplete",
  "mode": "recovery",
  "primary": [],
  "fallback": [
    1,
    0
  ]
}
```

## Execution instructions

Save the source in a fresh practice directory as `serve.sh`. Run `sh serve.sh` inside your disposable container so httpd can start workers as the packaged apache user. From a second terminal in the same container, run `curl -fsS http://127.0.0.1:18080/check-042973.json`. Stop the local server with Ctrl+C.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "project": "042973",
  "asset": "BRANCH-08",
  "metric": "unrestored records",
  "source": "fallback",
  "readings": 2,
  "flagged": 1
}
```

## Testing and validation

Ran the native Apache configuration check, started the isolated loopback server, and verified the exact static JSON response and the unknown-path HTTP 404.

```json
{
  "project": "042973",
  "asset": "BRANCH-08",
  "metric": "unrestored records",
  "source": "fallback",
  "readings": 2,
  "flagged": 1
}
```

## Error handling

The syntax check reports configuration errors. Requests for unknown files return HTTP 404.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The Alpine module paths and apache account are platform-specific. The server publishes a static report on loopback.

## Learning outcomes

Practice recovery test using Apache and check a repeatable recovery workflow in the branch office simulator scenario.

## Conclusion

The project records an observable recovery test result for training case `BRANCH-08`.

[Back to the Apache project index](../README.md)
