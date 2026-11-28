# Ansible: Data Import — Baseline Setup for Branch Office Simulator (Project 047271)

## Project overview

A simulated branch office provides the setting for this data import exercise. This project includes source code, sample data, and instructions to run it. The readings are synthetic.

## Problem statement

A simulated branch office needs a repeatable check for imported rows.

## Project objectives

- Review imported rows and flag readings that meet the cutoff rule.
- Record a reference observation and its threshold.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Ansible YAML, assertions, and Jinja filters.

## System architecture

`Synthetic observations → playbook → native evaluation → report`

## Features

- Review imported rows and flag readings that meet the cutoff rule.
- Record a reference observation and its threshold.

## Prerequisites

Use ansible-core and its supported Python runtime. Use a text editor and a local practice directory.

## Complete Ansible source code

```yaml
- name: Read imported rows for project 047271
  hosts: localhost
  connection: local
  gather_facts: false
  vars:
    fixture:
      project: '047271'
      asset: BRANCH-08
      metric: imported rows
      limit: 50
      comparison: lt
      finding: rows missing after import
      mode: baseline
      primary:
      - 48
      fallback: []
  tasks:
  - name: Select the active readings
    ansible.builtin.set_fact:
      selected: '{{ fixture.primary if fixture.primary | length > 0 else (fixture.fallback
        if fixture.mode == ''recovery'' else []) }}'
      source: '{{ ''primary'' if fixture.primary | length > 0 else ''fallback'' }}'
  - name: Validate configured inputs
    ansible.builtin.assert:
      quiet: true
      that:
      - fixture.comparison in ['lt', 'gt', 'ge']
      - fixture.limit is integer
      - fixture.limit >= 0
      - fixture.limit <= 1000000000
      - selected | length > 0
      - selected | reject('integer') | list | length == 0
      - selected | min >= 0
      - selected | max <= 1000000000
  - name: Count readings meeting the comparison
    ansible.builtin.set_fact:
      report: '{{ {''source'': source, ''readings'': selected | length, ''flagged'':
        selected | select(fixture.comparison, fixture.limit) | list | length} | combine({''values'':
        selected} if fixture.mode in [''baseline'', ''report''] else {}) }}'
  - name: Print findings
    ansible.builtin.debug:
      var: report
```

## Implementation explanation

The comparison is lt with limit 50. A flagged row means rows missing after import. Record a reference observation and its threshold.

## Sample input data

The example embeds these synthetic inputs.

```json
{
  "project": "047271",
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

Save the block as `playbook.yaml`, then run `ansible-playbook -i localhost, playbook.yaml`.

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

Executed the play with ansible-core on localhost and asserted its report. A negative-reading variant in the batch failed the input assertion.

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

The play operates on synthetic observations on localhost and changes no remote hosts.

## Learning outcomes

Practice data import using Ansible and check a repeatable baseline setup workflow in the branch office simulator scenario.

## Conclusion

The project records an observable data import result for training case `BRANCH-08`.

[Back to the Ansible project index](../README.md)
