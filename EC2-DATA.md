### Getting Meta data and Dynamic data
# Getting Meta Data and Dynamic Data

To improve security, Instance Metadata Service v2 (IMDSv2) now requires an authentication token to get metadata or dynamic data.

## Step 1: Get the Token

```sh
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
-H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
```

### Step 2: Use the Token to Call Meta-Data or Dynamic Data 
##### call meta-data 
```sh
curl -H "X-aws-ec2-metadata-token: $TOKEN" -v http://169.254.169.254/latest/meta-data/
```

##### call dynamic data
```sh
curl -H "X-aws-ec2-metadata-token: $TOKEN" -v http://169.254.169.254/latest/dynamic/instance-identity/

```
