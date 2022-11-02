# IAM roles<a name="iam-roles"></a>

While creating an identity pool, you're prompted to update the IAM roles that your users assume\. IAM roles work like this: When a user logs in to your app, Amazon Cognito generates temporary AWS credentials for the user\. These temporary credentials are associated with a specific IAM role\. With the IAM role, you can define a set of permissions to access your AWS resources\.

You can specify default IAM roles for authenticated and unauthenticated users\. In addition, you can define rules to choose the role for each user based on claims in the user's ID token\. For more information, see [Role\-based access control](role-based-access-control.md)\.

By default, the Amazon Cognito console creates IAM roles that provide access to Amazon Mobile Analytics and to Amazon Cognito Sync\. Alternatively, you can choose to use existing IAM roles\.

Modify IAM roles to allow or restrict access to other services\. To do so, [log in to the IAM Console](https://console.aws.amazon.com/iam/home)\. Then select **Roles**, and select a role\. The policies attached to the selected role are listed in the **Permissions** tab\. You can customize an access policy by selecting the corresponding **Manage Policy** link\. To learn more about using and defining policies, see [Overview of IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/PoliciesOverview.html)\.

**Note**  
As a best practice, define policies that follow the principle of granting *least privilege*\. In other words, the policies include only the permissions that users require to perform their tasks\. For more information, see [Grant Least Privilege](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege) in the *IAM User Guide*\.  
Remember that unauthenticated identities are assumed by users who do not log in to your app\. Typically, the permissions that you assign for unauthenticated identities should be more restrictive than those for authenticated identities\.

**Topics**
+ [Set up a trust policy](#trust-policies)
+ [Access policies](#access-policies)

## Set up a trust policy<a name="trust-policies"></a>

Amazon Cognito uses IAM roles to generate temporary credentials for your application's users\. Access to permissions is controlled by a role's trust relationships\. Learn more about [Role trust and permissions](role-trust-and-permissions.md)\.

**Reuse roles across identity pools**

To reuse a role across multiple identity pools, because they share a common permission set, you can include multiple identity pools, like this:

```
"StringEquals": {
    "cognito-identity.amazonaws.com:aud": [
        "us-east-1:12345678-abcd-abcd-abcd-123456790ab",
        "us-east-1:98765432-dcba-dcba-dcba-123456790ab"
    ]
}
```

**Limit access to specific identities**

To create a policy limited to a specific set of app users, check the value of `cognito-identity.amazonaws.com:sub`:

```
"StringEquals": {
    "cognito-identity.amazonaws.com:aud": "us-east-1:12345678-abcd-abcd-abcd-123456790ab",
    "cognito-identity.amazonaws.com:sub": [
        "us-east-1:12345678-1234-1234-1234-123456790ab",
        "us-east-1:98765432-1234-1234-1243-123456790ab"
    ]
}
```

**Limit access to specific providers**

To create a policy limited to users who have logged in with a specific provider \(perhaps your own login provider\), check the value of `cognito-identity.amazonaws.com:amr`:

```
"ForAnyValue:StringLike": {
    "cognito-identity.amazonaws.com:amr": "login.myprovider.myapp"
}
```

For example, an app that trusts only Facebook would have the following amr clause:

```
"ForAnyValue:StringLike": {
    "cognito-identity.amazonaws.com:amr": "graph.facebook.com"
}
```

## Access policies<a name="access-policies"></a>

The permissions attached to a role are effective across all users that assume that role\. If you want to partition your users' access, you can do so through policy variables\. Be careful when including your users' identity IDs in your access policies, particularly for unauthenticated identities, as these may change if the user logs in\.

For additional security protection, Amazon Cognito applies a scope\-down policy to credentials vended by `GetCredentialForIdentity` to prevent access to services other than the following ones for your unauthenticated users\. You can create an identity using these credentials with access to only the following services: 
+ Amazon API Gateway
+ AWS AppSync
+ Amazon CloudWatch
+ Amazon CloudWatch Logs
+ Amazon Cognito Identity
+ Amazon Cognito Sync
+ Amazon Cognito user pools
+ Amazon Comprehend
+ Amazon DynamoDB
+ Amazon GameLift
+ AWS IoT
+ AWS Key Management Service \(AWS KMS\)
+ Amazon Kinesis Data Firehose
+ Amazon Kinesis Data Streams
+ AWS Lambda
+ Amazon Lex
+ Amazon Location Service
+ Amazon Machine Learning
+ Amazon Mobile Analytics
+ Amazon Personalize
+ Amazon Pinpoint
+ Amazon Polly
+ Amazon Rekognition
+ Amazon SageMaker
+ Amazon SimpleDB
+ Amazon Simple Email Service \(Amazon SES\)
+ Amazon Simple Notification Service \(Amazon SNS\)
+ Amazon Simple Queue Service \(Amazon SQS\)
+ Amazon Simple Storage Service \(Amazon S3\)
+ Amazon Sumerian
+ Amazon Textract
+ Amazon Transcribe
+ Amazon Translate

In addition, there are services that grant access to unauthenticated users, but don't permit all the service's actions for these users\. The following table shows the services with restricted actions\.


| Service | Permission granted for unauthenticated users | 
| --- | --- | 
| AWS Key Management Service | Encrypt, Decrypt, ReEncrypt , GenerateDataKey | 
| Amazon SageMaker | InvokeEndpoint | 
| Amazon Textract | DetectDocumentText, AnalyzeDocument | 
| Amazon Sumerian | View\* | 

If you need access to something other than these services for your unauthenticated users, you must use the basic authentication flow\. If you are getting `NotAuthorizedException` and you have enabled access to the service in your unauthenticated role policy, this is likely the reason\.

### Access policy examples<a name="access-policy-examples"></a>

In this section, you can find example Amazon Cognito access policies that grant only the permissions your identities must complete a specific operation\. You can further limit the permissions for a given identity ID by using policy variables where possible\. For example, using $\{cognito\-identity\.amazonaws\.com:sub\}\. For more information, see [Understanding Amazon Cognito Authentication Part 3: Roles and Policies](https://aws.amazon.com/blogs/mobile/understanding-amazon-cognito-authentication-part-3-roles-and-policies/) on the *AWS Mobile Blog*\.

**Note**  
As a security best practice, policies should include only the permissions that users require to perform their tasks\. This means that you should try to always scope access to an individual identity for objects whenever possible\.

**Example 1: Grant an identity read access to a single object in Amazon S3**

The following access policy grants read permissions to an identity to retrieve a single object from a given S3 bucket\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": ["arn:aws:s3:::mybucket/assets/my_picture.jpg"]
    }
  ]
}
```

**Example 2: Grant an identity both read and write access to identity specific paths in Amazon S3**

The following access policy grants read and write permissions to access a specific prefix "folder" in an S3 bucket by mapping the prefix to the `${cognito-identity.amazonaws.com:sub}` variable\.

With this policy, an identity such as `us-east-1:12345678-1234-1234-1234-123456790ab` inserted via `${cognito-identity.amazonaws.com:sub}` can get, put, and list objects into `arn:aws:s3:::mybucket/us-east-1:12345678-1234-1234-1234-123456790ab`\. However, the identity would not be granted access to other objects in `arn:aws:s3:::mybucket`\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": ["s3:ListBucket"],
      "Effect": "Allow",
      "Resource": ["arn:aws:s3:::mybucket"],
      "Condition": {"StringLike": {"s3:prefix": ["${cognito-identity.amazonaws.com:sub}/*"]}}
    },
    {
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Effect": "Allow",
      "Resource": ["arn:aws:s3:::mybucket/${cognito-identity.amazonaws.com:sub}/*"]
    }
  ]
}
```

**Example 3: Assign identities fine\-grained access to Amazon DynamoDB**

The following access policy provides fine\-grained access control to DynamoDB resources using Amazon Cognito environment variables\. These variables grant access to items in DynamoDB by identity ID\. For more information, see [Using IAM Policy Conditions for Fine\-Grained Access Control](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/specifying-conditions.html) in the *Amazon DynamoDB Developer Guide*\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:BatchGetItem",
        "dynamodb:Query",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem",
        "dynamodb:BatchWriteItem"
      ],
      "Resource": [
        "arn:aws:dynamodb:us-west-2:123456789012:table/MyTable"
      ],
      "Condition": {
        "ForAllValues:StringEquals": {
          "dynamodb:LeadingKeys":  ["${cognito-identity.amazonaws.com:sub}"]
        }
      }
    }
  ]
}
```

**Example 4: Grant an identity permission to invoke a Lambda function**

The following access policy grants an identity permissions to invoke a Lambda function\.

```
{
    "Version": "2012-10-17",
    "Statement": [
         { 
             "Effect": "Allow",
             "Action": "lambda:InvokeFunction",
             "Resource": [
                 "arn:aws:lambda:us-west-2:123456789012:function:MyFunction"
             ]
         }
     ]
 }
```

**Example 5: Grant an identity permission to publish records to Kinesis Data Streams**

The following access policy allows an identity to use the `PutRecord` operation with any of the Kinesis Data Streams\. It can be applied to users that need to add data records to all streams in an account\. For more information, see [Controlling Access to Amazon Kinesis Data Streams Resources Using IAM](https://docs.aws.amazon.com/streams/latest/dev/controlling-access.html) in the *Amazon Kinesis Data Streams Developer Guide*\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "kinesis:PutRecord",
            "Resource": [
                "arn:aws:kinesis:us-east-1:111122223333:stream/stream1"
            ]
        }
    ]
}
```

**Example 6: Grant an identity access to their data in the Amazon Cognito Sync store**

The following access policy grants an identity permissions to access only their own data in the Amazon Cognito Sync store\.

```
{
  "Version": "2012-10-17",
  "Statement":[{
      "Effect":"Allow",
      "Action":"cognito-sync:*",
      "Resource":["arn:aws:cognito-sync:us-east-1:123456789012:identitypool/${cognito-identity.amazonaws.com:aud}/identity/${cognito-identity.amazonaws.com:sub}/*"]
      }]
  }
```