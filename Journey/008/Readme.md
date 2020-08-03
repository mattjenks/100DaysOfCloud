**Add a cover photo like:**
![AWS Accounts](https://github.com/mattjenks/100DaysOfCloud/blob/main/Journey/007/GT_AWS_Accounts.png?raw=true)

# Terraform - AWS Organizational structure

## Introduction

Following [AWS best practices](https://aws.amazon.com/organizations/getting-started/best-practices/), it's time to think about actually creating the identity account and users within in. This will act as my main account for login and user will have to assume roles to perform actions in other accounts within the [Organization](https://aws.amazon.com/organizations/).

## Prerequisite

- Knowledge of AWS Accounts
- Knowledge of AWS IAM (users/policies)
- KNowledge of Terraform Modules
- KNowledge of AWS organizations

## Use Case

- Create an Organization to which to apply child Organization Units allowing for enterprise wide policies to be applied, regardless of IAM User or account.
- Attach accounts to these organizations for specific purposes, such as creating a single account to log into, this not requiring multiple AWS IAM users for each AWS account.

## Cloud Research 

- I realized that my main terraform file was becoming complex as soon as I started adding organizations and accounts.
- When you refactor terraform files, especially changing the name of modules, this forces terraform to destroy and create, which can lead to to problems if you are not aware.
  - I changed my IAM user module for the root account/organization. This recreated my main admin user. Thus I had to re-configure my aws profile used by terraform.
  - Also, due to the policy applied on day 7, I could not do anything with that user until I re-enabled MFA.
  - **Note**, using [1 Password](https://1password.com/) as my password manager allows for easy setup of MFA and also ease of use during repeated login attempts via the AWS console.


## Try yourself

✍️ Add a mini tutorial to encourage the reader to get started learning something new about the cloud.

### Step 1 — Create a module for [organizations](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/organizations_organization)

```terraform
# main.tf
resource "aws_organizations_organization" "org" {
  feature_set                   = var.feature_set
  aws_service_access_principals = var.feature_set == "ALL" ? var.aws_service_access_principals : null
  enabled_policy_types          = var.feature_set == "ALL" ? var.enabled_policy_types : null
}

# variables.tf
variable "feature_set" {
  description = "After the Terraform docs: 'Specify ALL (default) or CONSOLIDATED_BILLING.'"
  default     = "ALL"
}

variable "aws_service_access_principals" {
  description = "After the Terraform docs: 'List of AWS service principal names for which you want to enable integration with your organization. This is typically in the form of a URL, such as service-abbreviation.amazonaws.com. Organization must have feature_set set to ALL. For additional information, see the AWS Organizations User Guide.'"
  type        = list(string)
  default     = null
}

variable "enabled_policy_types" {
  description = "After the Terraform docs: 'List of Organizations policy types to enable in the Organization Root. Organization must have feature_set set to ALL. For additional information about valid policy types (e.g. SERVICE_CONTROL_POLICY and TAG_POLICY), see the AWS Organizations API Reference.'"
  type        = list(string)
  default     = null
}

variable "tags" {
  description = "The list of tags to be applied to any resources created by this module"
  type        = map(string)
  default     = null
}

variable "policies" {
  description = "The policies to add to the group"
  type        = list(string)
  default     = []
}

# use the module
module "root" {
  source = "./modules/organizations"
  enabled_policy_types = [
    "SERVICE_CONTROL_POLICY",
    "TAG_POLICY"
  ]
  providers = {
    aws = aws.GT-Core-Account
  }
}
```

### Step 1 — Create [organization units](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/organizations_organizational_unit)

- similar to above, I created a module and then reused it to create my OUs

```terraform
module "security" {
  source    = "./modules/organizations-organizational_units"
  name      = "security"
  parent_id = module.root.roots.0.id
  providers = {
    aws = aws.GT-Core-Account
  }
}

module "infrastructure" {
  source    = "./modules/organizations-organizational_units"
  name      = "infrastructure"
  parent_id = module.root.roots.0.id
  providers = {
    aws = aws.GT-Core-Account
  }
}

module "sandbox" {
  source    = "./modules/organizations-organizational_units"
  name      = "sandbox"
  parent_id = module.root.roots.0.id
  providers = {
    aws = aws.GT-Core-Account
  }
}
```

### Step 3 — Create AWS accounts and attach them to the correct OU

- similar to above, I created a module and then reused it to create my accounts

```terraform
### ACCOUNTS - sample identity account from which all IAM users will be spawned
locals {
  role_name = "adminAssumeRole"
}

module "account-identity" {
  source    = "./modules/organizations-accounts"
  name      = "Identity"
  email     = "redacted"
  parent_id = module.security.id
  role_name = local.role_name
  tags      = local.common_tags
  providers = {
    aws = aws.GT-Core-Account
  }
}
```

## ☁️ Cloud Outcome

- Created modules to create AWS organizations
- Created modules to create AWS organizational units
- Created modules to create AWS accounts

## Next Steps

Add organization wide policies

## Social Proof

✍️ Show that you shared your process on Twitter or LinkedIn

[link](link)
