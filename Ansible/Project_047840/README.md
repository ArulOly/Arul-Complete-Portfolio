# Ansible: Documentation Generation — Usability Improvement for Two-Node Lab (Project 047840)

## Project overview

A two-node lab simulation provides the setting for this documentation generation exercise. This project includes source code, sample data, and instructions to run it. The readings are synthetic.

## Problem statement

A two-node lab simulation needs a repeatable check for missing handover steps.

## Project objectives

- Review missing handover steps and flag readings that meet the cutoff rule.
- Use a concise result that labels the metric and its flagged count.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Ansible YAML, assertions, and Jinja filters.

## System architecture

`Synthetic observations → playbook → native evaluation → report`

## Features

- Review missing handover steps and flag readings that meet the cutoff rule.
- Use a concise result that labels the metric and its flagged count.

## Prerequisites

Use ansible-core and its supported Python runtime. Use a text editor and a local practice directory.

## Complete Ansible source code

```yaml
- name: Read missing handover steps for project 047840
  hosts: localhost
  connection: local
  gather_facts: false
  vars:
    fixture:
      project: 047840
      asset: NODE-B
      metric: missing handover steps
      limit: 0
      comparison: gt
      finding: handover is incomplete
      mode: summary
      primary:
      - 1
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

The comparison is gt with limit 0. A flagged row means handover is incomplete. Use a concise result that labels the metric and its flagged count.

## Sample input data

The example embeds these synthetic inputs.

```json
{
  "project": "047840",
  "asset": "NODE-B",
  "metric": "missing handover steps",
  "limit": 0,
  "comparison": "gt",
  "finding": "handover is incomplete",
  "mode": "summary",
  "primary": [
    1
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
  "flagged": 1
}
```

## Testing and validation

Executed the play with ansible-core on localhost and asserted its report. A negative-reading variant in the batch failed the input assertion.

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

The play operates on synthetic observations on localhost and changes no remote hosts.

## Learning outcomes

Practice documentation generation using Ansible and check a repeatable usability improvement workflow in the two-node lab scenario.

## Conclusion

The project records an observable documentation generation result for training case `NODE-B`.

[Back to the Ansible project index](../README.md)
