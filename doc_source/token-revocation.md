# Revoking tokens<a name="token-revocation"></a>

You can revoke a refresh token for a user using the AWS API\. When you revoke a refresh token, all access tokens that were previously issued by that refresh token become invalid\. The other refresh tokens issued to the user are not affected\.

**Note**  
 [JWT tokens](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-verifying-a-jwt.html) are self\-contained with a signature and expiration time that was assigned when the token was created\. Revoked tokens can't be used with any Amazon Cognito API calls that require a token\. However, revoked tokens will still be valid if they are verified using any JWT library that verifies the signature and expiration of the token\.

You can revoke a refresh token for a user pool client with token revocation enabled\. When you create a new user pool client, token revocation is enabled by default\.

## Enable token revocation<a name="enable-token-revocation"></a>

Before you can revoke a token for an existing user pool client, you must enable token revocation\. You can enable token revocation for existing user pool clients using the AWS CLI or the AWS API\. To do this, call the `aws cognito-idp describe-user-pool-client` CLI command or the `DescribeUserPoolClient` API operation to retrieve the current settings from your app client\. Then call the `aws cognito-idp update-user-pool-client` CLI command or the `UpdateUserPoolClient` API operation\. Include the current settings from your app client and set the `EnableTokenRevocation` parameter to `true`\.

When you create a new user pool client using the AWS Management Console, the AWS CLI, or the AWS API, token revocation is enabled by default\.

After you enable token revocation, new claims are added in the Amazon Cognito JSON web tokens\. The `origin_jti` and `jti` claims are added to access and ID tokens\. These claims increase the size of the application client access and ID tokens\.

The following JSON example shows a request to enable token revocation using the `[CreateUserPoolClient](https://docs.aws.amazon.com/cognito-user-identity-pools/latest/APIReference/API_CreateUserPoolClient.html)` API\.

```
{
    "AccessTokenValidity": 123,
    "AllowedOAuthFlows": [
        "string"
    ],
    "AllowedOAuthFlowsUserPoolClient": true,
    "AllowedOAuthScopes": [
        "string"
    ],
    "AnalyticsConfiguration": {
        "ApplicationArn": "string",
        "ApplicationId": "string",
        "ExternalId": "string",
        "RoleArn": "string",
        "UserDataShared": false
    },
    "CallbackURLs": [
        "string"
    ],
    "ClientName": "string",
    "DefaultRedirectURI": "string",
    "ExplicitAuthFlows": [
        "string"
    ],
    "GenerateSecret": true,
    "IdTokenValidity": 123,
    "LogoutURLs": [
        "string"
    ],
    "PreventUserExistenceErrors": "string",
    "ReadAttributes": [
        "string"
    ],
    "RefreshTokenValidity": 456,
    "SupportedIdentityProviders": [
        "string"
    ],
    "TokenValidityUnits": {
        "AccessToken": "string",
        "IdToken": "string",
        "RefreshToken": "string"
    },
    "UserPoolId": "string",
    "WriteAttributes": [
        "string"
    ],
    "EnableTokenRevocation": true
}
```

## Revoke a token<a name="revoke-tokens-api"></a>

You can revoke a refresh token using the `[RevokeToken](https://docs.aws.amazon.com/cognito-user-identity-pools/latest/APIReference/API_RevokeToken.html)` API operation\. You can also use the `[aws cognito\-idp revoke\-token](https://docs.aws.amazon.com/cli/latest/reference/cognito-idp/revoke-token.html)` CLI command to revoke tokens\. Finally, you can revoke tokens using the [revocation endpoint](https://docs.aws.amazon.com/cognito/latest/developerguide/revocation-endpoint.html)\. This endpoint is available after you add a domain to your user pool\. You can use the revocation endpoint on either an Amazon Cognito hosted domain or your own custom domain\.

**Note**  
To revoke a refresh token, you must use the same client ID that was used to obtain the token\.