# Detection Logic

## Objective

Identify AWS management events that may indicate unauthorized activity,
misconfiguration, excessive permissions, or unexpected account enumeration.

## Detection 1: Denied API Activity

### Detection Goal

Identify API requests denied by AWS authorization controls.

### CloudWatch Logs Insights Query

```sql
fields @timestamp, eventSource, eventName, awsRegion, userIdentity.arn, errorCode, errorMessage
| filter ispresent(errorCode)
| filter errorCode like /AccessDenied|UnauthorizedOperation/
| sort @timestamp desc
| limit 50
```

### Fields Reviewed

- `@timestamp` — time of the request
- `userIdentity.arn` — identity that made the request
- `eventSource` — AWS service receiving the request
- `eventName` — attempted API action
- `awsRegion` — Region where the request occurred
- `errorCode` — authorization or API error
- `errorMessage` — reason for the denial

### Lab Finding

CloudTrail captured a denied console-initiated request:

| Field | Observed value |
|---|---|
| Event source | `ec2.amazonaws.com` |
| Event name | `DescribeRegions` |
| Region | `us-east-2` |
| Error code | `Client.UnauthorizedOperation` |
| Result | No identity-based policy allowed `ec2:DescribeRegions` |

### Investigation Steps

1. Identify the principal that generated the denied event.
2. Review the attempted API action and AWS service.
3. Confirm whether the action was expected for the principal's job function.
4. Review the attached IAM policy for an applicable Allow statement.
5. Determine whether the denial reflects appropriate least privilege or a
   configuration issue.
6. Document the finding and avoid granting additional permissions unless the
   action is required.

## Detection 2: IAM Enumeration Activity

### Detection Goal

Identify IAM discovery actions that could indicate administrative review or
potential account enumeration.

### CloudWatch Logs Insights Query

```sql
fields @timestamp, eventSource, eventName, awsRegion, userIdentity.arn, sourceIPAddress, errorCode
| filter eventSource = "iam.amazonaws.com"
| filter eventName in [
  "ListUsers",
  "ListGroups",
  "ListRoles",
  "ListAccessKeys",
  "ListMFADevices",
  "GetAccountSummary"
]
| sort @timestamp desc
| limit 50
```

### Observed IAM Events

The following IAM management events were observed in `us-east-1`:

- `ListUsers`
- `ListGroups`
- `ListRoles`
- `ListAccessKeys`
- `ListMFADevices`
- `GetAccountSummary`

### Investigation Steps

1. Identify the user or role that performed the IAM action.
2. Verify whether the IAM activity is expected for that identity.
3. Review the source IP address, session type, time, and frequency of events.
4. Determine whether the actions were successful or contained an error code.
5. Escalate unexpected enumeration performed by non-administrative identities.

## Tuning Considerations

IAM enumeration actions are not inherently malicious. Administrators and AWS
Console workflows can legitimately generate read-only actions such as
`ListUsers` and `ListRoles`. These events should be treated as suspicious when
they are unexpected for the principal, source location, time, or volume of
activity.
