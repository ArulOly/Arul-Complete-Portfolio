# Kubernetes: Service Check — Usability Improvement for School Lab (Project 018605)

## Project overview

Shared classroom equipment provides the setting for this service check exercise. This project includes source code, sample data, and instructions to run it. The readings are synthetic.

## Problem statement

Shared classroom equipment needs a repeatable check for healthy service checks.

## Project objectives

- Review healthy service checks and flag readings that meet the cutoff rule.
- Show the input source and flagged count in a concise result.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Kubernetes ConfigMap, Job, and Pod resource limits.

## System architecture

`Synthetic observations → Job manifest → native evaluation → report`

## Features

- Review healthy service checks and flag readings that meet the cutoff rule.
- Show the input source and flagged count in a concise result.

## Prerequisites

Use kubectl, a local Kubernetes practice cluster, and access to the Alpine 3.22 image. Use a text editor and a local practice directory.

## Complete Kubernetes source code

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: portfolio-018605
data:
  report.sh: |
    #!/bin/sh
    set -eu
    primary=${PRIMARY-2}
    fallback=${FALLBACK-''}
    threshold=${THRESHOLD-3}
    comparison=lt
    mode=summary
    metric='healthy service checks'

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
  name: portfolio-018605
spec:
  backoffLimit: 0
  activeDeadlineSeconds: 60
  template:
    metadata:
      labels:
        app: portfolio-018605
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
          name: portfolio-018605
```

## Implementation explanation

The comparison is lt with limit 3. A flagged row means health checks failed. Show the input source and flagged count in a concise result.

## Sample input data

The example embeds these synthetic inputs.

```json
{
  "project": "018605",
  "asset": "LAB-17",
  "metric": "healthy service checks",
  "limit": 3,
  "comparison": "lt",
  "finding": "health checks failed",
  "mode": "summary",
  "primary": [
    2
  ],
  "fallback": []
}
```

## Execution instructions

Save the block as `job.yaml`. In a dedicated practice namespace, run `kubectl apply -f job.yaml`, wait for Job `portfolio-018605` to finish, then run `kubectl logs job/portfolio-018605`. Clean up the practice objects with `kubectl delete -f job.yaml`.

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

Kubeconform validated both manifests against the Kubernetes 1.32 schemas. The mounted Job script ran in the configured Alpine image and rejected malformed input. A Kubernetes cluster run remains unverified.

```json
{
  "source": "primary",
  "readings": 1,
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

Practice service check using Kubernetes and check a repeatable usability improvement workflow in the school lab scenario.

## Conclusion

The project records an observable service check result for training case `LAB-17`.

[Back to the Kubernetes project index](../README.md)
