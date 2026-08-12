# Remediation Recommendations

- Use least-privilege IAM policies and grant only required actions.
- Review CloudTrail events for unexpected IAM enumeration activity.
- Investigate recurring `AccessDenied` and `UnauthorizedOperation` events.
- Determine whether denied console-generated API calls are required for a job role
  before granting additional permissions.
- Require MFA for administrative users.
- Retain CloudTrail logs in a protected S3 bucket.
- Enable CloudTrail log-file validation to help detect log-file modification.
- Review unused permissions and remove unnecessary access.
