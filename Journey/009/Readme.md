**Add a cover photo like:**
![AWS Accounts](https://github.com/mattjenks/100DaysOfCloud/blob/main/Journey/007/GT_AWS_Accounts.png?raw=true)

# Terraform - AWS Organizational Policies

## Introduction

I anticipate needing to know which account is costing what in the sense of dollars. Thus I need a way to know what resources are being using which cost center. I need a tag policy applied across my organization.

## Prerequisite

- AWS Tags
- AWS Tag Policies

## Use Case

- I should be able to create a tag policy compliance report
- I should be able to see a cost report by tag

## Cloud Research

- created a terraform module for adding a simple tag policy

```terraform
resource "aws_organizations_policy" "gt_default_tag_policy" {
  description = "The GigaTECH default tag policy for tagging AWS resources"
  name        = "gt_default_tag_policy"
  type        = "TAG_POLICY"

  content = <<CONTENT
{
  "tags": {
    "gt:costcenter": {
      "tag_key": {
        "@@assign": "gt:costcenter",
        "@@operators_allowed_for_child_policies": ["@@none"]
      },
      "tag_value": {
        "@@assign": [
          "r and d",
          "bd"
        ]
      }
    },
    "gt:createdby": {
      "tag_key": {
        "@@assign": "gt:createdby",
        "@@operators_allowed_for_child_policies": ["@@none"]
      },
      "tag_value": {
        "@@assign": [
          "terraform",
          "manual",
          ""
        ]
      }
    },
    "gt:project": {
      "tag_key": {
        "@@assign": "gt:project",
        "@@operators_allowed_for_child_policies": ["@@none"]
      }
    }
  }
}
CONTENT
}

# output the tag policy name and id
output "policy_gt_default_tag_policy" {
  description = "The GigaTECH default tag policy"
  value = {
    name = aws_organizations_policy.gt_default_tag_policy.name
    id   = aws_organizations_policy.gt_default_tag_policy.id
  }
}
```

- applied the tag policy to my root org (which means all my OUs will inherit it)

```terraform
locals {
  common_tags = "${map(
    "gt:costcenter", "${var.costcenter}"
  )}"
}
module "tag_policies_core" {
  source = "./modules/organizations-policies"
  providers = {
    aws = aws.GT-Core-Account
  }
  tags = local.common_tags
}

module "root" {
  source = "./modules/organizations"
  enabled_policy_types = [
    "SERVICE_CONTROL_POLICY",
    "TAG_POLICY"
  ]
  providers = {
    aws = aws.GT-Core-Account
  }
  tags = local.common_tags
  policies = [
    "${module.tag_policies_core.policy_gt_default_tag_policy.id}"
  ]
  aws_service_access_principals = [
    "tagpolicies.tag.amazonaws.com"
  ]
}
```

- Set tags to my local common tags as seen in the tags line in the terraform code above.

## ☁️ Cloud Outcome

- compliance report from AWS
![AWS Accounts](https://github.com/mattjenks/100DaysOfCloud/blob/main/Journey/009/tags_compliance.png?raw=true)
- Currently using AWS free tier, so nothing of real value to show by tag

## Next Steps

Using this infrastructure, start creating a chatbot.

## Social Proof

[twitter](link)
