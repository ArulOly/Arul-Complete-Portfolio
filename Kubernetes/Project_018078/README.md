# Kubernetes: Monitoring — Recovery for Offline Classroom (Project 018078)

## Project overview

A classroom operating offline provides the setting for this monitoring exercise. This project includes source code, sample data, and instructions to run it. The readings are synthetic.

## Problem statement

A classroom operating offline needs a repeatable check for peak utilization percent.

## Project objectives

- Review peak utilization percent and flag readings that meet the cutoff rule.
- Use the fallback observations when the primary set is empty.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Kubernetes ConfigMap, Job, and Pod resource limits.

## System architecture

`Synthetic observations → Job manifest → native evaluation → report`

## Features

- Review peak utilization percent and flag readings that meet the cutoff rule.
- Use the fallback observations when the primary set is empty.

## Prerequisites

Use kubectl, a local Kubernetes practice cluster, and access to the Alpine 3.22 image. Use a text editor and a local practice directory.

## Complete Kubernetes source code

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: portfolio-018078
data:
  report.sh: "#!/bin/sh\nset -eu\nprimary=${PRIMARY-''}\nfallback=${FALLBACK-'83 80'}\n\
    threshold=${THRESHOLD-80}\ncomparison=gt\nmode=recovery\nmetric='peak utilization\
    \ percent'\n\nsource=primary\nvalues=$primary\nif [ -z \"$values\" ] && [ \"$mode\"\
    \ = recovery ]; then\n    source=fallback\n    values=$fallback\nfi\n\nprintf\
    \ '%s\\n' \"$values\" | awk -v limit=\"$threshold\" -v op=\"$comparison\" \\\n\
    \    -v source=\"$source\" -v mode=\"$mode\" -v metric=\"$metric\" '\n    BEGIN\
    \ { if (limit !~ /^[0-9]+$/ || limit > 1000000000) exit 2 }\n    {\n        if\
    \ (NF == 0) exit 2\n        for (i = 1; i <= NF; i++) {\n            if ($i !~\
    \ /^[0-9]+$/ || $i > 1000000000) exit 2\n            n = $i + 0\n            flagged\
    \ += (op == \"lt\" && n < limit) || (op == \"gt\" && n > limit) || (op == \"ge\"\
    \ && n >= limit)\n            detail = detail (i == 1 ? \"\" : \",\") n\n    \
    \    }\n        printf \"{\\\"source\\\":\\\"%s\\\",\\\"readings\\\":%d,\\\"flagged\\\
    \":%d\", source, NF, flagged\n        if (mode == \"baseline\" || mode == \"report\"\
    ) printf \",\\\"values\\\":[%s]\", detail\n        printf \"}\\n\"\n    }\n' ||\
    \ { printf '%s\\n' \"Invalid $metric input\" >&2; exit 2; }\n"
---
apiVersion: batch/v1
kind: Job
metadata:
  name: portfolio-018078
spec:
  backoffLimit: 0
  activeDeadlineSeconds: 60
  template:
    metadata:
      labels:
        app: portfolio-018078
    spec:
      restartPolicy: Never
      automountServiceAccountToken: false
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
      containers:
      - name: report
        image: alpine:3.22
        command:
        - /bin/sh
        - /sample/report.sh
        resources:
          requests:
            cpu: 10m
            memory: 16Mi
          limits:
            cpu: 100m
            memory: 64Mi
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
            - ALL
        volumeMounts:
        - name: sample
          mountPath: /sample
          readOnly: true
      volumes:
      - name: sample
        configMap:
          name: portfolio-018078
```

## Implementation explanation

The comparison is gt with limit 80. A flagged row means utilization above budget. Use the fallback observations when the primary set is empty.

## Sample input data

The example embeds these synthetic inputs.

```json
{
  "project": "018078",
  "asset": "CLASS-09",
  "metric": "peak utilization percent",
  "limit": 80,
  "comparison": "gt",
  "finding": "utilization above budget",
  "mode": "recovery",
  "primary": [],
  "fallback": [
    83,
    80
  ]
}
```

## Execution instructions

Save the block as `job.yaml`. In a dedicated practice namespace, run `kubectl apply -f job.yaml`, wait for Job `portfolio-018078` to finish, then run `kubectl logs job/portfolio-018078`. Clean up the practice objects with `kubectl delete -f job.yaml`.

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

Kubeconform validated both manifests against the Kubernetes 1.32 schemas. The mounted Job script ran in the configured Alpine image and rejected malformed input. A Kubernetes cluster run remains unverified.

```json
{
  "source": "fallback",
  "readings": 2,
  "flagged": 1
}
```

## Error handling

The implementation rejects invalid numeric inputs. Recovery uses the fallback only when the primary list is empty.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The Job uses synthetic readings and a temporary ConfigMap in a practice namespace. Cluster scheduling and admission behavior remain unverified.

## Learning outcomes

Practice monitoring using Kubernetes and check a repeatable recovery workflow in the offline classroom scenario.

## Conclusion

The project records an observable monitoring result for training case `CLASS-09`.

[Back to the Kubernetes project index](../README.md)
