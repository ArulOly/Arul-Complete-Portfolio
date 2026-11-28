# Ansible: Service Check — Recovery for Help Desk (Project 047643)

## Project overview

A help desk training queue provides the setting for this service check exercise. This project includes source code, sample data, and instructions to run it. The readings are synthetic.

## Problem statement

A help desk training queue needs a repeatable check for healthy service checks.

## Project objectives

- Review healthy service checks and flag readings that meet the cutoff rule.
- Use the fallback observations when the primary set is empty.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Ansible YAML, assertions, and Jinja filters.

## System architecture

`Synthetic observations → playbook → native evaluation → report`

## Features

- Review healthy service checks and flag readings that meet the cutoff rule.
- Use the fallback observations when the primary set is empty.

## Prerequisites

Use ansible-core and its supported Python runtime. Use a text editor and a local practice directory.

## Complete Ansible source code

```yaml
- name: Read healthy service checks for project 047643
  hosts: localhost
  connection: local
  gather_facts: false
  vars:
    fixture:
      project: '047643'
      asset: DESK-14
      metric: healthy service checks
      limit: 3
      comparison: lt
      finding: health checks failed
      mode: recovery
      primary: []
      fallback:
      - 2
      - 4
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

The comparison is lt with limit 3. A flagged row means health checks failed. Use the fallback observations when the primary set is empty.

## Sample input data

The example embeds these synthetic inputs.

```json
{
  "project": "047643",
  "asset": "DESK-14",
  "metric": "healthy service checks",
  "limit": 3,
  "comparison": "lt",
  "finding": "health checks failed",
  "mode": "recovery",
  "primary": [],
  "fallback": [
    2,
    4
  ]
}
```

## Execution instructions

Save the block as `playbook.yaml`, then run `ansible-playbook -i localhost, playbook.yaml`.

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

Executed the play with ansible-core on localhost and asserted its report. A negative-reading variant in the batch failed the input assertion.

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

The play operates on synthetic observations on localhost and changes no remote hosts.

## Learning outcomes

Practice service check using Ansible and check a repeatable recovery workflow in the help desk scenario.

## Conclusion

The project records an observable service check result for training case `DESK-14`.

[Back to the Ansible project index](../README.md)
