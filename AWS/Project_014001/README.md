# AWS: Inventory — Baseline Setup for School Lab (Project 014001)

## Project overview

Shared classroom equipment provides the setting for this inventory exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The school lab needs a repeatable inventory baseline that another student can verify.

## Project objectives

- Record the school lab inventory baseline.
- Create a CloudFormation declaration with asset tags, encryption, versioning, and public access blocked.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

AWS CloudFormation YAML and Amazon S3.

## System architecture

`Inventory parameters → infrastructure template → local validation → declared asset baseline`

## Features

- Record the school lab inventory baseline.
- Create a CloudFormation declaration with asset tags, encryption, versioning, and public access blocked.

## Prerequisites

Install cfn-lint to check the local template. Use a text editor and a local practice directory.

## Complete AWS source code

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Practice inventory baseline for one school lab bucket
Resources:
  LabInventory:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
    UpdateReplacePolicy: Retain
    Properties:
      VersioningConfiguration:
        Status: Enabled
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
      Tags:
        - Key: Asset
          Value: LAB-17
        - Key: Environment
          Value: practice
```

## Implementation explanation

Create a CloudFormation declaration with asset tags, encryption, versioning, and public access blocked.

## Sample input data

Use this example input.

```json
{
  "asset_tag": "LAB-17",
  "environment": "practice"
}
```

## Execution instructions

Save the source as `inventory.yaml` and run `cfn-lint -t inventory.yaml`.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "resource_type": "AWS::S3::Bucket",
  "asset_tag": "LAB-17",
  "versioning": "Enabled",
  "blocks_public_access": true
}
```

## Testing and validation

cfn-lint accepted the CloudFormation template. A local check read the resulting template fields shown below.

```json
{
  "resource_type": "AWS::S3::Bucket",
  "asset_tag": "LAB-17",
  "versioning": "Enabled",
  "blocks_public_access": true
}
```

## Error handling

Check syntax, resource types, and input names before deployment. Always use an authorized practice resource group or project for provider-side validation.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

This example covers the declared configuration. No cloud resources were provisioned or checked in a live account.

## Learning outcomes

Practice inventory using AWS and check a repeatable baseline setup workflow in the school lab scenario.

## Conclusion

The project records an observable inventory result for training case `LAB-17`.

[Back to the AWS project index](../README.md)
