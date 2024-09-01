# S3 Bucket Policy

## Policy JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-bucket/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "192.0.2.0/24"
        }
      }
    }
  ]
}

```
## Explanation

- **Effect**: The action to be taken (Allow or Deny).
- **Principal**: The user or role to which the policy applies. `*` means all users.
- **Action**: The specific actions allowed (e.g., `s3:GetObject`).
- **Resource**: The bucket and objects the policy applies to.
- **Condition**: Additional conditions for the policy, such as IP address restrictions. This means that the specified source IP will be the only one allowed.
