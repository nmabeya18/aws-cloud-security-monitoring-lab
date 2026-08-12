# AWS Cloud Security Monitoring Lab

## Objective

Built an AWS security monitoring lab to centralize management-event logging,
validate least-privilege IAM controls, and investigate IAM enumeration and
denied API activity.

## Technologies

- AWS IAM
- AWS CloudTrail
- Amazon CloudWatch Logs
- Amazon S3
- AWS Console
- CloudWatch Logs Insights

## Architecture

The lab uses IAM identities with least-privilege permissions, a multi-Region
CloudTrail trail, Amazon S3 for log storage, and CloudWatch Logs for event
monitoring and investigation.


## Test Case 1: IAM Enumeration Activity

CloudTrail Event History in `us-east-1` captured IAM management events generated
through the AWS Console.

Observed actions included:

- `ListUsers`
- `ListGroups`
- `ListRoles`
- `ListAccessKeys`
- `ListMFADevices`
- `GetAccountSummary`


### Investigation Fields

The investigation reviewed:

- `userIdentity`
- `eventSource`
- `eventName`
- `eventTime`
- `sourceIPAddress`
- `awsRegion`
- `readOnly`
- `errorCode`

## Test Case 2: Denied API Activity

### Least-Privilege Policy

The lab identity was assigned a policy granting only read access to selected
CloudWatch metric actions:

- `cloudwatch:GetMetricData`
- `cloudwatch:ListMetrics`

The policy did not grant `ec2:DescribeRegions`. AWS therefore implicitly denied
the console-initiated EC2 request, which CloudTrail logged with
`Client.UnauthorizedOperation`.

See [sanitized policy](policies/cloudsec-lab-admin-policy.json).


## Detection Logic

- Investigate unexpected IAM enumeration activity.
- Investigate repeated `AccessDenied` or `UnauthorizedOperation` errors.
- Validate whether an attempted API action is necessary for the principal’s role.
- Review privileged IAM actions, including access-key creation, policy changes,
  role assumption, and user creation.

CloudWatch Logs Insights queries are available in the `queries` directory.

## Findings

1. IAM global-service events appeared in CloudTrail Event History in `us-east-1`.
2. A denied `ec2:DescribeRegions` request appeared in `us-east-2`.
3. Least-privilege access controls prevented an unauthorized API action.
4. CloudTrail fields supported investigation of the user identity, event source,
   event name, Region, and authorization result.

## Remediation Recommendations

- Retain least-privilege IAM policies and remove unused permissions.
- Investigate unexpected IAM enumeration by non-administrative identities.
- Monitor repeated authorization failures for possible misuse or misconfiguration.
- Require MFA for privileged identities.
- Enable CloudTrail log-file validation and protect centralized logs in S3.

## Security Note

All screenshots and evidence were sanitized before publication. No AWS account
IDs, credentials, access keys, tokens, private IP addresses, or raw production
logs are included in this repository.
