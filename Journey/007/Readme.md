![AWS Accounts](https://github.com/mattjenks/100DaysOfCloud/blob/main/Journey/007/GT_AWS_Accounts.png?raw=true)

# Terraform Modules

## Introduction

I started looking at terraform as I have to setup my new companies cloud usage infrastructure. I need a way to centrally managed multiple cloud/AWS accounts while applying security policies such as [AWS Best Practices](https://aws.amazon.com/organizations/getting-started/best-practices/).

## Prerequisite

- Knowledge of AWS Accounts
- Knowledge of AWS IAM (users/policies)
- KNowledge of Terraform Modules

## Use Case

- Create an AWS account
- Create an IAM user
- Apply Admin permissions to the IAM user
- Apply a policy to the IAM user to disallow all AWS capabilities except account self service until MFA is turned on. This should be a reusable policy as defined by this [tutorial](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_users-self-manage-mfa-and-creds.html)

## Cloud Research

Created a terraform project.
Created a provider.tf to establish my connect to AWS.

```terraform
provider "aws" {
  version = "~> 2.70"

  region = var.aws_region

  # lock down which account can operate on this template to only the core account
  allowed_account_ids = [var.aws_account_id]

  # require specific AWS IAM user
  profile = "GT_CORE_ADMIN"
}
```

Note use of the profile set via aws cli using --profile param.
Created a policies module to specify the reuseable policy.

```terraform
data "aws_iam_policy_document" "GT_UserSelfServiceWithMFA" {
  version = "2012-10-17"

  statement {
    sid    = "AllowViewAccountInfo"
    effect = "Allow"
    actions = [
      "iam:GetAccountPasswordPolicy",
      "iam:GetAccountSummary",
      "iam:ListVirtualMFADevices",
      "iam:ListVirtualMFADevices"
    ]
    resources = ["*"]
  }

  statement {
    sid    = "AllowManageOwnPasswords"
    effect = "Allow"
    actions = [
      "iam:ChangePassword",
      "iam:GetUser",
    ]
    resources = [
      "arn:aws:iam::*:user/$${aws:username}",
      "arn:aws:iam::*:user/GT/$${aws:username}",
      "arn:aws:iam::*:user/external/$${aws:username}"
    ]
  }

  statement {
    sid    = "AllowManageOwnAccessKeys"
    effect = "Allow"
    actions = [
      "iam:CreateAccessKey",
      "iam:DeleteAccessKey",
      "iam:ListAccessKeys",
      "iam:UpdateAccessKey"
    ]
    resources = [
      "arn:aws:iam::*:user/$${aws:username}",
      "arn:aws:iam::*:user/GT/$${aws:username}",
      "arn:aws:iam::*:user/external/$${aws:username}"
    ]
  }

  statement {
    sid    = "AllowManageOwnSigningCertificates"
    effect = "Allow"
    actions = [
      "iam:DeleteSigningCertificate",
      "iam:ListSigningCertificates",
      "iam:UpdateSigningCertificate",
      "iam:UploadSigningCertificate"
    ]
    resources = [
      "arn:aws:iam::*:user/$${aws:username}",
      "arn:aws:iam::*:user/GT/$${aws:username}",
      "arn:aws:iam::*:user/external/$${aws:username}"
    ]
  }

  statement {
    sid    = "AllowManageOwnSSHPublicKeys"
    effect = "Allow"
    actions = [
      "iam:DeleteSSHPublicKey",
      "iam:GetSSHPublicKey",
      "iam:ListSSHPublicKeys",
      "iam:UpdateSSHPublicKey",
      "iam:UploadSSHPublicKey"
    ]
    resources = [
      "arn:aws:iam::*:user/$${aws:username}",
      "arn:aws:iam::*:user/GT/$${aws:username}",
      "arn:aws:iam::*:user/external/$${aws:username}"
    ]
  }

  statement {
    sid    = "AllowManageOwnGitCredentials"
    effect = "Allow"
    actions = [
      "iam:CreateServiceSpecificCredential",
      "iam:DeleteServiceSpecificCredential",
      "iam:ListServiceSpecificCredentials",
      "iam:ResetServiceSpecificCredential",
      "iam:UpdateServiceSpecificCredential"
    ]
    resources = [
      "arn:aws:iam::*:user/$${aws:username}",
      "arn:aws:iam::*:user/GT/$${aws:username}",
      "arn:aws:iam::*:user/external/$${aws:username}"
    ]
  }

  statement {
    sid    = "AllowManageOwnVirtualMFADevice"
    effect = "Allow"
    actions = [
      "iam:CreateVirtualMFADevice",
      "iam:DeleteVirtualMFADevice"
    ]
    resources = [
      "arn:aws:iam::*:user/$${aws:username}",
      "arn:aws:iam::*:user/GT/$${aws:username}",
      "arn:aws:iam::*:user/external/$${aws:username}"
    ]
  }

  statement {
    sid    = "AllowManageOwnUserMFA"
    effect = "Allow"
    actions = [
      "iam:DeactivateMFADevice",
      "iam:EnableMFADevice",
      "iam:ListMFADevices",
      "iam:ResyncMFADevice"
    ]
    resources = [
      "arn:aws:iam::*:user/$${aws:username}",
      "arn:aws:iam::*:user/GT/$${aws:username}",
      "arn:aws:iam::*:user/external/$${aws:username}"
    ]
  }

  statement {
    sid    = "DenyAllExceptListedIfNoMFA"
    effect = "Deny"
    not_actions = [
      "iam:CreateVirtualMFADevice",
      "iam:EnableMFADevice",
      "iam:GetUser",
      "iam:ListVirtualMFADevices",
      "iam:ListMFADevices",
      "iam:ResyncMFADevice",
      "sts:GetSessionToken"
    ]
    resources = ["*"]
    condition {
      test     = "BoolIfExists"
      variable = "aws:MultiFactorAuthPresent"
      values   = ["false"]
    }
  }
}

resource "aws_iam_policy" "GT_UserSelfServiceWithMFA" {
  name        = "GT_UserSelfServiceWithMFA"
  description = "GigaTECH managed user self service policy"
  policy      = data.aws_iam_policy_document.GT_UserSelfServiceWithMFA.json
}
```

created a module for creating a group with custom policies.

```terraform
// module
resource "aws_iam_group" "group" {
  name = var.name
}

resource "aws_iam_group_policy_attachment" "GroupPolicy" {
  count      = length(var.policies)
  group      = aws_iam_group.group.name
  policy_arn = var.policies[count.index]
}

// use of module
module "administrators" {
  source = "./iam-groups"
  name   = "gt-administrators"
  policies = [
    "${module.policies.policy_UserSelfServiceWithMFA.arn}",
    "${module.policies.policy_administrator.arn}"
  ]
}
```

ran tf init (terraform init) but received several confusing errors. Note tf is my terraform zsh function as described [here](https://github.com/mattjenks/100DaysOfCloud/tree/main/Journey/005)
Found this [tutorial](https://chaosgears.com/dont-panic-organize-part-2-of-2/) which aided greatly in what I was trying to do.
finally was able to tf init.
performed a 'tf plan' and then a 'tf apply'.
I made a couple of mistakes, so I had to iterate on the above terraform code (what I showed is final) as I created that user but locked them out. Turns out in the create user, I had specified a default value for path, so my resources in my policy were not correct. I created the user but was locked out and couldn't even self manage the account. I had to manually fix the user by logging in as the root user for the account. Once I fixed the path in my terraform code, I was good to go.


## ☁️ Cloud Outcome

- I know of IaC terraform code in a corporate GH repo for creating the base account and locking down the admin user with a policy I will be able to generically apply.
- Lesson learned
  - good to have the backup root account when iterating through IAM scenarios in case you lock yourself out.

## Next Steps

- Create an identity account where all my users will be created for every account.

## Social Proof

[twitter](https://twitter.com/mejenks/status/1285995416346079235?s=20)
