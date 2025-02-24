---
title: Connecting Github Actions to AWS
date: "2025-01-01T18:00:01.123Z"
description: "Connecting GitHub Actions to AWS Using OIDC and an Assumed Role"
---

# Connecting GitHub Actions to AWS Using OIDC and an Assumed Role

When deploying applications or managing AWS infrastructure through GitHub Actions, using OpenID Connect (OIDC) provides a secure way to authenticate without requiring static AWS credentials. This guide will walk you through setting up an AWS IAM role, configuring an OIDC identity provider, and updating your GitHub Actions workflow to assume the role.

## Step 1: Create an IAM Role for GitHub Actions

1. Navigate to the AWS IAM Console.
2. Click on **Roles** > **Create role**.
3. Select **Web identity** as the trusted entity type.
4. Choose **OpenID Connect (OIDC)** as the identity provider.
5. Enter the OIDC provider URL: `https://token.actions.githubusercontent.com`
6. Set the audience to `sts.amazonaws.com`.
7. Click **Next** and proceed to configure permissions.

### IAM Trust Policy
After creating the role, update the trust policy to allow GitHub Actions to assume it:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::<YOUR AWS ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                },
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": [
                        "repo:<YOUR GITHUB USERNAME>/<YOUR GITHUB REPO NAME>:*"
                    ]
                }
            }
        }
    ]
}
```

Replace:
- `<YOUR AWS ACCOUNT_ID>` with your actual AWS account ID.
- `<YOUR GITHUB USERNAME>` with your GitHub username.
- `<YOUR GITHUB REPO NAME>` with your repository name.

### Attach Required Policies

Once the role is created, attach necessary policies that grant permissions required for your workflow. For example, if deploying to S3, attach `AmazonS3FullAccess`. The role should be named **GitHubActionsRole**.

## Step 2: Configure OIDC Identity Provider in AWS

1. Navigate to [AWS IAM Identity Providers](https://us-east-1.console.aws.amazon.com/iam/home?region=eu-west-2#/identity_providers).
2. Click **Add provider** and select **OpenID Connect (OIDC)**.
3. Enter the provider URL: `https://token.actions.githubusercontent.com`
4. Set the audience to `sts.amazonaws.com`.
5. Click **Add provider** to finalise.

## Step 3: Update GitHub Actions Workflow

Modify your GitHub Actions workflow file (e.g., `.github/workflows/deploy.yml`) to assume the AWS role:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::<YOUR AWS ACCOUNT_ID>:role/GitHubActionsRole
          role-session-name: GitHubActionsSession
          aws-region: eu-west-2

      - name: Verify AWS Authentication
        run: aws sts get-caller-identity
```

Replace `<YOUR AWS ACCOUNT_ID>` with your actual AWS account ID.

## Step 4: Run the Workflow

Commit and push the changes to your repository. Navigate to the **Actions** tab in GitHub to verify that the workflow executes successfully. If set up correctly, the `aws sts get-caller-identity` command should return your AWS account details, confirming a successful OIDC authentication.

## Conclusion

By using OIDC, you eliminate the need for long-lived AWS access keys in GitHub Secrets, improving security and simplifying access management. With this setup, GitHub Actions can securely interact with AWS services using short-lived credentials from an assumed IAM role.

