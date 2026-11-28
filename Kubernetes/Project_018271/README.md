# Kubernetes: Data Import — Baseline Setup for Branch Office Simulator (Project 018271)

## Project overview

A simulated branch office provides the setting for this data import exercise. This project includes source code, sample data, and instructions to run it. The readings are synthetic.

## Problem statement

A simulated branch office needs a repeatable check for imported rows.

## Project objectives

- Review imported rows and flag readings that meet the cutoff rule.
- Record a reference observation and its threshold.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Kubernetes ConfigMap, Job, and Pod resource limits.

## System architecture

`Synthetic observations → Job manifest → native evaluation → report`

## Features

- Review imported rows and flag readings that meet the cutoff rule.
- Record a reference observation and its threshold.

## Prerequisites

Use kubectl, a local Kubernetes practice cluster, and access to the Alpine 3.22 image. Use a text editor and a local practice directory.

## Complete Kubernetes source code

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: portfolio-018271
data:
  report.sh: "#!/bin/sh\nset -eu\nprimary=${PRIMARY-48}\nfallback=${FALLBACK-''}\n\
    threshold=${THRESHOLD-50}\ncomparison=lt\nmode=baseline\nmetric='imported rows'\n\
    \nsource=primary\nvalues=$primary\nif [ -z \"$values\" ] && [ \"$mode\" = recovery\
    \ ]; then\n    source=fallback\n    values=$fallback\nfi\n\nprintf '%s\\n' \"\
    $values\" | awk -v limit=\"$threshold\" -v op=\"$comparison\" \\\n    -v source=\"\
    $source\" -v mode=\"$mode\" -v metric=\"$metric\" '\n    BEGIN { if (limit !~\
    \ /^[0-9]+$/ || limit > 1000000000) exit 2 }\n    {\n        if (NF == 0) exit\
    \ 2\n        for (i = 1; i <= NF; i++) {\n            if ($i !~ /^[0-9]+$/ ||\
    \ $i > 1000000000) exit 2\n            n = $i + 0\n            flagged += (op\
    \ == \"lt\" && n < limit) || (op == \"gt\" && n > limit) || (op == \"ge\" && n\
    \ >= limit)\n            detail = detail (i == 1 ? \"\" : \",\") n\n        }\n\
    \        printf \"{\\\"source\\\":\\\"%s\\\",\\\"readings\\\":%d,\\\"flagged\\\
    \":%d\", source, NF, flagged\n        if (mode == \"baseline\" || mode == \"report\"\
    ) printf \",\\\"values\\\":[%s]\", detail\n        printf \"}\\n\"\n    }\n' ||\
    \ { printf '%s\\n' \"Invalid $metric input\" >&2; exit 2; }\n"
---
apiVersion: batch/v1
kind: Job
metadata:
  name: portfolio-018271
spec:
  backoffLimit: 0
  activeDeadlineSeconds: 60
  template:
    metadata:
      labels:
        app: portfolio-018271
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
          name: portfolio-018271
```

## Implementation explanation

The comparison is lt with limit 50. A flagged row means rows missing after import. Record a reference observation and its threshold.

## Sample input data

The example embeds these synthetic inputs.

```json
{
  "project": "018271",
  "asset": "BRANCH-08",
  "metric": "imported rows",
  "limit": 50,
  "comparison": "lt",
  "finding": "rows missing after import",
  "mode": "baseline",
  "primary": [
    48
  ],
  "fallback": []
}
```

## Execution instructions

Save the block as `job.yaml`. In a dedicated practice namespace, run `kubectl apply -f job.yaml`, wait for Job `portfolio-018271` to finish, then run `kubectl logs job/portfolio-018271`. Clean up the practice objects with `kubectl delete -f job.yaml`.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "source": "primary",
  "readings": 1,
  "flagged": 1,
  "values": [
    48
  ]
}
```

## Testing and validation

Kubeconform validated both manifests against the Kubernetes 1.32 schemas. The mounted Job script ran in the configured Alpine image and rejected malformed input. A Kubernetes cluster run remains unverified.

```json
{
  "source": "primary",
  "readings": 1,
  "flagged": 1,
  "values": [
    48
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

Practice data import using Kubernetes and check a repeatable baseline setup workflow in the branch office simulator scenario.

## Conclusion

The project records an observable data import result for training case `BRANCH-08`.

[Back to the Kubernetes project index](../README.md)
