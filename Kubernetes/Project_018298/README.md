# Kubernetes: Data Import — Recovery for Portfolio Demo (Project 018298)

## Project overview

A repeatable portfolio demonstration provides the setting for this data import exercise. This project includes source code, sample data, and instructions to run it. The readings are synthetic.

## Problem statement

A repeatable portfolio demonstration needs a repeatable check for imported rows.

## Project objectives

- Review imported rows and flag readings that meet the cutoff rule.
- Use the fallback observations when the primary set is empty.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Kubernetes ConfigMap, Job, and Pod resource limits.

## System architecture

`Synthetic observations → Job manifest → native evaluation → report`

## Features

- Review imported rows and flag readings that meet the cutoff rule.
- Use the fallback observations when the primary set is empty.

## Prerequisites

Use kubectl, a local Kubernetes practice cluster, and access to the Alpine 3.22 image. Use a text editor and a local practice directory.

## Complete Kubernetes source code

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: portfolio-018298
data:
  report.sh: |
    #!/bin/sh
    set -eu
    primary=${PRIMARY-''}
    fallback=${FALLBACK-'48 51'}
    threshold=${THRESHOLD-50}
    comparison=lt
    mode=recovery
    metric='imported rows'

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
  name: portfolio-018298
spec:
  backoffLimit: 0
  activeDeadlineSeconds: 60
  template:
    metadata:
      labels:
        app: portfolio-018298
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
          name: portfolio-018298
```

## Implementation explanation

The comparison is lt with limit 50. A flagged row means rows missing after import. Use the fallback observations when the primary set is empty.

## Sample input data

The example embeds these synthetic inputs.

```json
{
  "project": "018298",
  "asset": "DEMO-02",
  "metric": "imported rows",
  "limit": 50,
  "comparison": "lt",
  "finding": "rows missing after import",
  "mode": "recovery",
  "primary": [],
  "fallback": [
    48,
    51
  ]
}
```

## Execution instructions

Save the block as `job.yaml`. In a dedicated practice namespace, run `kubectl apply -f job.yaml`, wait for Job `portfolio-018298` to finish, then run `kubectl logs job/portfolio-018298`. Clean up the practice objects with `kubectl delete -f job.yaml`.

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

Practice data import using Kubernetes and check a repeatable recovery workflow in the portfolio demo scenario.

## Conclusion

The project records an observable data import result for training case `DEMO-02`.

[Back to the Kubernetes project index](../README.md)
