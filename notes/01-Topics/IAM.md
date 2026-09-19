
Use an IAM role when an AWS service or user needs access to AWS resources.
Roles provide temporary credentials and help avoid long-term access keys.

## Cross-Account Access

You can establish cross-account access through a trust relationship.

Example: Role A in Account A needs to access resources in Account B.

- Role A needs `sts:AssumeRole` permission for Role B.
- Role B's trust policy must trust Role A.
- Role B's permission policy defines what it can do in Account B.

**Trust Policy → WHO can assume the role**
**Permission Policy → WHAT the role can do**

## Deny

General IAM evaluation rule:

`Explicit Deny > Explicit Allow > Implicit Deny`

## Permissions Boundary

Defines the maximum permissions that a user or role can have.

⚠️ A permissions boundary does NOT grant permissions.

Effective permissions:
`Identity Policy ∩ Permissions Boundary`

## SCP

An AWS Organizations guardrail.

Defines the maximum available permissions for member accounts and OUs.

⚠️ An SCP does NOT grant permissions.
⚠️ It also applies to the root user of a member account.

## PassRole

`iam:PassRole`

Lets a principal pass an IAM role to an AWS service.

Example:
Developer creates Lambda → passes LambdaExecutionRole to Lambda.

## MFA Condition

Use:

`aws:MultiFactorAuthPresent`

when access MUST require MFA.

## S3 via VPC Endpoint

Condition key:

`aws:SourceVpce`

Use it when access must come through a specific VPC endpoint.

## SSE-KMS Read

To read an S3 object encrypted with SSE-KMS, the role typically needs:

- `s3:GetObject`
- `kms:Decrypt`

Also check the applicable:
- S3 bucket policy
- KMS key policy

⚠️ `s3:DecryptObject` does NOT exist.