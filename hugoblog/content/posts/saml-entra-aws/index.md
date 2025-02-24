---
title: AWS + Azure - Single Sign On 
date: "2024-11-19T15:00:01.123Z"
description: "Configuring Azure Entra ID to login to AWS"
---
# AWS + Azure - Single Sign-On: Configuring Azure Entra ID to Login to AWS via Terraform

Single Sign-On (SSO) is a powerful feature that allows users to access multiple applications with a single set of credentials. Integrating Azure Entra ID (formerly Azure Active Directory) with Amazon Web Services (AWS) enables seamless access management across both platforms. In this blog post, we'll explore how to configure Azure Entra ID to enable SSO to AWS using Terraform.

## Introduction

The integration of Azure Entra ID with AWS allows organisations to leverage their existing identity infrastructure to control access to AWS resources. This is achieved through SAML-based authentication, where Azure Entra ID acts as the Identity Provider (IdP) and AWS acts as the Service Provider (SP).

We'll walk through the steps to:

- Create AWS IAM resources using Terraform.
- Configure an Azure AD claims mapping policy using Terraform.
- Set up the Azure Entra ID application and service principal.
- Assign the claims mapping policy to the Azure application.
- Test the SSO configuration.

## Prerequisites

- An Azure subscription with permissions to manage Azure Entra ID.
- An AWS account with permissions to create IAM roles and SAML providers.
- Terraform installed and configured on your machine.
- Terraform providers for Azure AD, AWS, and SAML.

## Overview

To set up SSO between Azure Entra ID and AWS, we'll:

1. Create AWS IAM role and SAML provider using Terraform.
2. Define the necessary SAML claims in Azure Entra ID using Terraform.
3. Configure the Azure AD claims mapping policy using Terraform.
4. Set up the Azure Entra ID application and service principal using Terraform.
5. Assign the claims mapping policy to the Azure application.
6. Test the SSO configuration.

## Step 1: Create AWS IAM Resources Using Terraform

First, we'll create an AWS IAM role and SAML provider using Terraform.

### AWS IAM SAML Provider

```hcl
resource "aws_iam_saml_provider" "aws_single_sso" {
  name                   = "AzureAD"
  saml_metadata_document = saml_metadata.aws_single_sso.metadata
}
```

- **Explanation**: Creates a SAML provider in AWS using the metadata from Azure Entra ID.

### AWS IAM Role

```hcl
resource "aws_iam_role" "power_user_assumable_role" {
  name = "PowerUser-via-Azure"

  assume_role_policy = data.aws_iam_policy_document.azure_ad_trust_policy.json
}
```

- **Explanation**: Creates an IAM role that trusts the SAML provider for federated access.

### IAM Trust Policy

```hcl
data "aws_iam_policy_document" "azure_ad_trust_policy" {
  statement {
    actions = [
      "sts:AssumeRoleWithSAML",
      "sts:TagSession"
    ]
    effect = "Allow"
    principals {
      identifiers = [
        aws_iam_saml_provider.aws_single_sso.arn
      ]
      type = "Federated"
    }
    condition {
      test     = "StringEquals"
      variable = "SAML:aud"
      values = [
        "https://signin.aws.amazon.com/saml"
      ]
    }
  }
}
```

- **Explanation**: Defines the trust relationship between the IAM role and the SAML provider.

## Step 2: Configure Azure AD Claims Mapping Policy Using Terraform

We'll define the claims mapping policy that specifies which claims are sent to AWS during authentication.

```hcl
resource "azuread_claims_mapping_policy" "aws_attributes" {
  display_name = "AWSSSOClaimsMapping"

  definition = [
    jsonencode({
      ClaimsMappingPolicy = {
        Version = 1
        ClaimsSchema = [
          {
            SamlClaimType = "https://aws.amazon.com/SAML/Attributes/Role"
            Value         = "${aws_iam_role.power_user_assumable_role.arn},${aws_iam_saml_provider.aws_single_sso.arn}"
          },
          {
            SamlClaimType = "https://aws.amazon.com/SAML/Attributes/SessionDuration"
            Value         = "3600"
          },
          {
            SamlClaimType    = "https://aws.amazon.com/SAML/Attributes/RoleSessionName"
            ID               = "RoleSessionName"
            Source           = "transformation"
            TransformationID = "extractMailPrefix"
          },
          {
            ID            = "mail"
            SamlClaimType = "https://aws.amazon.com/SAML/Attributes/PrincipalTag:Email"
            Source        = "user"
          }
        ]
        ClaimsTransformations = [
          {
            ID                   = "extractMailPrefix"
            TransformationMethod = "ExtractMailPrefix"
            InputClaims = [
              {
                ClaimTypeReferenceId    = "mail"
                TransformationClaimType = "mail"
              }
            ]
            OutputClaims = [
              {
                ClaimTypeReferenceId    = "RoleSessionName"
                TransformationClaimType = "outputClaim"
              }
            ]
          }
        ]
      }
    })
  ]
}
```

### Explanation

- **ClaimsSchema**:
  - **Role**: Specifies the AWS IAM role and SAML provider ARNs.
  - **SessionDuration**: Sets the session duration to 3600 seconds.
  - **RoleSessionName**:
    - **ID**: `"RoleSessionName"`.
    - **Source**: `"transformation"`.
    - **TransformationID**: `"extractMailPrefix"`.
  - **PrincipalTag:Email**:
    - **ID**: `"mail"`.
    - **Source**: `"user"` (retrieves `user.mail`).

- **ClaimsTransformations**:
  - **extractMailPrefix**:
    - **TransformationMethod**: `"ExtractMailPrefix"`.
    - **InputClaims**: Uses the `mail` claim.
    - **OutputClaims**: Outputs to `RoleSessionName`.

### Important Notes

- **Property Names Are Case-Sensitive**: Ensure that all property names match the exact casing expected by Azure AD (e.g., `ID`, `TransformationID`).
- **Using `jsonencode()` Correctly**: The `jsonencode()` function expects HCL expressions, not JSON strings.
- **Transformations with Basic Claim Set**: This configuration shows that transformations can be used even when `IncludeBasicClaimSet` is not explicitly set to `false`.

## Step 3: Set Up Azure Entra ID Application and Service Principal Using Terraform

### Azure AD Application Template

```hcl
data "azuread_application_template" "aws_single_sso" {
  display_name = "AWS Single-Account Access"
}
```

- **Explanation**: Retrieves the application template for AWS Single-Account Access.

### Azure AD Application

```hcl
resource "azuread_application" "aws_single_sso" {
  display_name    = "Connection-To-AWS"
  template_id     = data.azuread_application_template.aws_single_sso.id
  owners          = [data.azuread_client_config.current.object_id]
  identifier_uris = ["https://signin.aws.amazon.com/saml#2"]
  public_client {
    redirect_uris = ["https://signin.aws.amazon.com/saml"]
  }
}
```

- **Explanation**: Creates an Azure AD application for AWS SSO. 
Identifier Urls must be unique across the tenant, however for multiple aws accounts to be mapped, they can have a `#` suffix with a numeric identifier which gets removed by Entra when communicating with AWS.

### Azure AD Service Principal

```hcl
resource "azuread_service_principal" "aws_single_sso" {
  client_id    = azuread_application.aws_single_sso.client_id
  use_existing = true
  feature_tags {
    enterprise = true
  }
  owners                        = [data.azuread_client_config.current.object_id]
  preferred_single_sign_on_mode = "saml"
}
```

- **Explanation**: Creates a service principal for the Azure AD application and sets SAML as the preferred SSO mode.

## Step 4: Assign the Claims Mapping Policy to the Azure Application

```hcl
resource "azuread_service_principal_claims_mapping_policy_assignment" "aws_single_sso" {
  service_principal_id     = azuread_service_principal.aws_single_sso.id
  claims_mapping_policy_id = azuread_claims_mapping_policy.aws_attributes.id
}
```

- **Explanation**: Assigns the claims mapping policy to the service principal.

## Step 5: Configure the SAML Signing Certificate and Metadata

### Azure AD Service Principal Token Signing Certificate

```hcl
resource "azuread_service_principal_token_signing_certificate" "aws_single_sso" {
  service_principal_id = azuread_service_principal.aws_single_sso.id
}
```

- **Explanation**: Generates a token signing certificate for the service principal.

### SAML Metadata

```hcl
resource "saml_metadata" "aws_single_sso" {
  url                          = local.saml_metadata_url
  token_signing_key_thumbprint = azuread_service_principal_token_signing_certificate.aws_single_sso.thumbprint
}
```

- **Explanation**: Retrieves the SAML metadata from Azure Entra ID.

This uses a [terraform provider](https://github.com/rgl/terraform-provider-saml) which acts as a cache because the metadata URL will retrieve a fresh copy every time, causing terraform to redeploy resources on every run.  When you actually want to redeploy the metadata XML to aws, you need to run `terraform state rm saml_metadata.aws_single_sso` to remove it from state so it gets recreated and reuploaded.


### SAML Metadata URL

```hcl
locals {
  saml_metadata_url = "https://login.microsoftonline.com/${azuread_service_principal.aws_single_sso.application_tenant_id}/federationmetadata/2007-06/federationmetadata.xml?appid=${azuread_application.aws_single_sso.client_id}"
}
```

- **Explanation**: Constructs the URL to fetch the federation metadata for the application.

## Step 6: Configure Providers and Client Config

### Azure AD Client Configuration

```hcl
data "azuread_client_config" "current" {}
```

- **Explanation**: Retrieves information about the current Azure AD client configuration.

### Terraform Providers

```hcl
terraform {
  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "3.0.2"
    }

    aws = {
      source  = "hashicorp/aws"
      version = "5.76.0"
    }

    saml = {
      source = "rgl/saml"
    }
  }
}

provider "azuread" {
  # Configuration options
}

provider "aws" {
  # Configuration options
}
```

- **Explanation**: Specifies the required providers and their versions.

## Step 7: Test the SSO Configuration

After applying the Terraform configuration:

1. **Sign In to AWS via Azure Entra ID**: Use the AWS Single Sign-On URL configured in your Azure enterprise application.
2. **Verify the SAML Response**: Use a browser extension like SAML Tracer to inspect the SAML token. Ensure that:
   - The `Role` attribute contains the correct ARNs.
   - The `RoleSessionName` is set to the prefix of the user's email.
   - The `SessionDuration` is correctly set.
   - The `PrincipalTag:Email` attribute is present.

## Troubleshooting Common Errors

### 400 Bad Request Error

This error typically indicates an issue with the policy definition:

- **Incorrect Property Names**: Azure AD policies are case-sensitive. Double-check property names like `ID`, `TransformationID`, and `TransformationMethod`.
- **Invalid JSON Structure**: Use Terraform's `jsonencode()` properly. Provide HCL expressions without including JSON-specific syntax like quoted keys.
- **Unsupported Transformations**: Ensure that the transformation methods used are supported in SAML policies. `ExtractMailPrefix` is supported.

### Transformation Limitations

Azure Entra ID supports specific transformation methods for SAML tokens:

- **Supported Methods**: `ExtractMailPrefix`, `FormatString`, and `Join`.
- **Unsupported Methods**: Methods like `ExtractUpnUserName` are not supported for SAML.

## Conclusion

Configuring Azure Entra ID to enable SSO to AWS using Terraform involves careful crafting of the claims mapping policy and the setup of Azure AD applications and service principals. Key takeaways include:

- **Property Names Must Match Exactly**: Azure AD policies are strict about property name casing.
- **Use Supported Transformations**: Only certain transformation methods are supported for SAML tokens.
- **Correct Usage of `jsonencode()`**: Ensure that HCL expressions are properly provided to `jsonencode()`.
- **Simplify Where Possible**: Use default settings when appropriate to reduce complexity.

By following the steps outlined in this guide and paying close attention to the details, you can successfully integrate Azure Entra ID with AWS for Single Sign-On, enhancing security and simplifying access management.

## References

- [Azure AD Claims Mapping Policy Schema](https://learn.microsoft.com/en-us/azure/active-directory/develop/reference-claims-mapping-policy-type)
- [Azure AD Claims Customisation](https://learn.microsoft.com/en-us/entra/identity-platform/reference-claims-customization#claim-sets)
- [AWS SAML Attributes](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_saml_assertions.html)
- [Terraform `jsonencode()` Function](https://www.terraform.io/language/functions/jsonencode)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform AzureAD Provider](https://registry.terraform.io/providers/hashicorp/azuread/latest/docs)
- [Microsoft Tutorial: AWS Single Account Access](https://learn.microsoft.com/en-gb/entra/identity/saas-apps/amazon-web-service-tutorial)
- [AWS: Troubleshooting Invalid SAML Response](https://docs.aws.amazon.com/IAM/latest/UserGuide/troubleshoot_saml.htm)
- [Microsoft: Customising Claims Mapping Policies](https://learn.microsoft.com/en-us/entra/identity-platform/claims-customization-powershell)
- [Microsoft: SAML Claims Customisation](https://learn.microsoft.com/en-us/entra/identity-platform/saml-claims-customization)
- [Microsoft Tutorial: Configuring SAML SSO with Microsoft Graph](https://learn.microsoft.com/en-us/graph/application-saml-sso-configure-api)
- [Github Issues: AzureAD provider - Claims Transformation](https://github.com/hashicorp/terraform-provider-azuread/issues/1115)

