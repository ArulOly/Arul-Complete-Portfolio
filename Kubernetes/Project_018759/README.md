# Kubernetes: Data Validation — Reporting for Small Business (Project 018759)

## Project overview

Local shop operations provides the setting for this data validation exercise. This project includes source code, sample data, and instructions to run it. The readings are synthetic.

## Problem statement

Local shop operations needs a repeatable check for invalid input rows.

## Project objectives

- Review invalid input rows and flag readings that meet the cutoff rule.
- Return the detailed readings and flag count for review.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Kubernetes ConfigMap, Job, and Pod resource limits.

## System architecture

`Synthetic observations → Job manifest → native evaluation → report`

## Features

- Review invalid input rows and flag readings that meet the cutoff rule.
- Return the detailed readings and flag count for review.

## Prerequisites

Use kubectl, a local Kubernetes practice cluster, and access to the Alpine 3.22 image. Use a text editor and a local practice directory.

## Complete Kubernetes source code

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: portfolio-018759
data:
  report.sh: |
    #!/bin/sh
    set -eu
    primary=${PRIMARY-'1 0'}
    fallback=${FALLBACK-''}
    threshold=${THRESHOLD-0}
    comparison=gt
    mode=report
    metric='invalid input rows'

    source=primary
    values=$primary
    if [ -z "$values" ] && [ "$mode" = recovery ]; then
        source=fallback
        values=$fallback
    fi

    printf '%s\n' "$values" | awk -v limit="$threshold" -v op="$comparison" \
        -v source="$source" -v mode="$mode" -v metric="$metric" '
        BEGIN { if (limit !~ /^[0-9]+$/ || limit > 1000000000) exit 2 }
        {
            if (NF == 0) exit 2
            for (i = 1; i <= NF; i++) {
                if ($i !~ /^[0-9]+$/ || $i > 1000000000) exit 2
                n = $i + 0
                flagged += (op == "lt" && n < limit) || (op == "gt" && n > limit) || (op == "ge" && n >= limit)
                detail = detail (i == 1 ? "" : ",") n
            }
            printf "{\"source\":\"%s\",\"readings\":%d,\"flagged\":%d", source, NF, flagged
            if (mode == "baseline" || mode == "report") printf ",\"values\":[%s]", detail
            printf "}\n"
        }
    ' || { printf '%s\n' "Invalid $metric input" >&2; exit 2; }
---
apiVersion: batch/v1
kind: Job
metadata:
  name: portfolio-018759
spec:
  backoffLimit: 0
  activeDeadlineSeconds: 60
  template:
    metadata:
      labels:
        app: portfolio-018759
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
          name: portfolio-018759
```

## Implementation explanation

The comparison is gt with limit 0. A flagged row means validation failed. Return the detailed readings and flag count for review.

## Sample input data

The example embeds these synthetic inputs.

```json
{
  "project": "018759",
  "asset": "SHOP-22",
  "metric": "invalid input rows",
  "limit": 0,
  "comparison": "gt",
  "finding": "validation failed",
  "mode": "report",
  "primary": [
    1,
    0
  ],
  "fallback": []
}
```

## Execution instructions

Save the block as `job.yaml`. In a dedicated practice namespace, run `kubectl apply -f job.yaml`, wait for Job `portfolio-018759` to finish, then run `kubectl logs job/portfolio-018759`. Clean up the practice objects with `kubectl delete -f job.yaml`.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "source": "primary",
  "readings": 2,
  "flagged": 1,
  "values": [
    1,
    0
  ]
}
```

## Testing and validation

Kubeconform validated both manifests against the Kubernetes 1.32 schemas. The mounted Job script ran in the configured Alpine image and rejected malformed input. A Kubernetes cluster run remains unverified.

```json
{
  "source": "primary",
  "readings": 2,
  "flagged": 1,
  "values": [
    1,
    0
  ]
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

Practice data validation using Kubernetes and check a repeatable reporting workflow in the small business scenario.

## Conclusion

The project records an observable data validation result for training case `SHOP-22`.

[Back to the Kubernetes project index](../README.md)
