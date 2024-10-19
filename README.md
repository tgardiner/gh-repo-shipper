# gh-repo-shipper
[gh-repo-shipper](gh-repo-shipper.yml) is a Github Actions Workflow that archives the repository and uploads it to an S3 bucket.

## Requirements
It requires the following secrets to be configured on the repository:
```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
S3_REPO_BUCKET
```

## AWS Setup
You'll need to create:
* An S3 bucket
* An IAM user with access to upload to the bucket

### Create the S3 bucket
```
aws s3 mb s3://<my-bucket>
```

#### Enable bucket versioning
```
aws s3api put-bucket-versioning \
  --bucket <my-bucket> \
  --versioning-configuration Status=Enabled
```

#### Enable bucket encryption
```
aws s3api put-bucket-encryption \
  --bucket <my-bucket> \
  --server-side-encryption-configuration '{"Rules": [{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "AES256"}}]}'
```

### Create the IAM user
```
aws iam create-user \
  --user-name <my-user>
```

#### Create a policy that allows putting the archive to s3
```
aws iam create-policy \
    --policy-name <my-policy> \
    --policy-document \
'{
  "Version" : "2012-10-17",
  "Statement" : [
    {
      "Effect" : "Allow",
      "Action" : [
        "s3:PutObject"
      ],
      "Resource" : "arn:aws:s3:::<my-bucket>/<my-repository>/repo.tar.gz"
    }
  ]
}'
```

#### Assign the policy to the user
```
aws iam attach-user-policy \
    --policy-arn arn:aws:iam::<my-account>:policy/<my-policy> \
    --user-name <my-user>
```
