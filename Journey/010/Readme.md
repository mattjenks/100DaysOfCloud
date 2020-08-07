**Add a cover photo like:**
![AWS Accounts](https://github.com/mattjenks/100DaysOfCloud/blob/main/Journey/007/GT_AWS_Accounts.png?raw=true)

# Terraform cross account users

## Introduction

Oops! I tried to create some resources in my project account. I could not because I could not assume a role with the correct permissions. Turns out, I did not setup my users in my identity account correctly, as described in [Day 9](../009/Readme.md) and [Day 8](../008/Readme.md) and [Day7](../007/Readme.md).

## Prerequisite

- Knowledgeable about aws sts assume role (both source and target accounts)
- Knowledgeable about aws policy creation
- Knowledgeable about Terraform aws resources related to these concepts

## Use Case

- Using Terraform, create a user in the identity account
- Create a role in the identity account to allow assumption of a role in the project account (called R and D)
- Create a role in the project account to allow assumption from a role in the identity account (called R and D)
- Assign the identity account role to the user in the identity account
- Test role assumption.

## Cloud Research/try yourself

- found this blog post helpful [How to Create Cross-Account User Roles for AWS with Terraform](https://blog.container-solutions.com/how-to-create-cross-account-user-roles-for-aws-with-terraform)

- created a policy in the identity account to assume a role in the project account

```terraform
resource "aws_iam_policy" "identity_randd_developer" {
  name        = "identity_randd_developer"
  description = "allow assuming r and d account developer role"
  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Effect   = "Allow",
        Action   = "sts:AssumeRole",
        Resource = "arn:aws:iam::${module.account-randd.id}:role/${aws_iam_role.randd_identity_developer.name}"
    }]
  })
  provider = aws.GT-Identity-Account
}
```

- the hardest part for me was coming up with terraform resource names that identified what I was doing so that I would not get lost in the layers of abstraction
  - I decided on "accountsrc_accountdest_rolename"
  - the above example starts in the identity account, assumes a role in the randd account and the role name is developer

- created a policy in the project(randd) account to allow assumption from the identity account

```terraform
resource "aws_iam_role" "randd_identity_developer" {
  name = "randd_identity_developer"
  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Effect    = "Allow",
        Action    = "sts:AssumeRole",
        Principal = { "AWS" : "arn:aws:iam::${module.account-identity.id}:root" }
    }]
  })
  provider = aws.GT-RandD-Account
}
```

- in each case I had to attach the policies

```terraform
resource "aws_iam_policy_attachment" "randd_identity_developer_attachment" {
  name       = "developer policy to developer role"
  roles      = ["${aws_iam_role.randd_identity_developer.name}"]
  policy_arn = module.policies_randd.policy_administrator.arn
  provider   = aws.GT-RandD-Account
}

# attach the policy directly to the user
resource "aws_iam_user_policy_attachment" "identity_randd_developer_mattatgt" {
  user       = module.mattatgt_identity.aws_iam_user-credentials.name
  policy_arn = aws_iam_policy.identity_randd_developer.arn
  provider   = aws.GT-Identity-Account
}
```

- Note the use of provider in the resources. I had to make sure I was using proper aliased terraform aws providers. For instance:

```terraform
provider "aws" {
  version = "~> 2.70"

  region = var.region

  # lock down which account can operate on this template to only the identity account
  allowed_account_ids = [var.identity_account_id]

  # assume admin role in identity account
  assume_role {
    role_arn = var.identity_account_admin_role
  }
  profile = "GT_CORE_ADMIN"
  alias   = "GT-Identity-Account"
}
```

- Unfortunately, the AWS cli does not yet work
  - followed the aws [cli & mfa instructions](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-role.html)
    - added randd_developer profile to ~/.aws/config
    - added role arn profile to ~/.aws/config
     source profile
added mfa_serial, added role_session_name

```zsh
[profile randd_developer]
role_arn = arn:aws:iam::xxxxxxxxxxxx:role/randd_identity_developer
source_profile = aws_login_profile
mfa_serial = arn:aws:iam::xxxxxxxxxxxx:mfa/aws_login_profile
role_session_name = some_session_name
```

- This was great, I could now run aws commands using my user on the research and development account using the developer role.
- However, I had to retrive that 1 time password, which proved to be a pain
  - I use 1 password
  - The web integration is great for 1 time passwords
  - I needed a way to quickly get 1 time passwords from the cli
  - Enter the [1password cli](https://support.1password.com/command-line/)
    - quick [brew install](https://formulae.brew.sh/cask/1password-cli) "brew cask install 1password-cli"

- Awesome, I can now get my passwords and 1 time passwords from the cli
  - It would be better if I could manage the 30 minute 1 password session and not have to lookup the titles or UUIDs of the passwords I use a lot.
  - Thanks to this [post](https://austincloud.guru/2018/11/27/1password-cli-tricks/) I created some quick functions that were better than the ones I started to create

```zsh
unction opon() {
  if [[ -z $OP_SESSION_my ]]; then
    eval $(op signin my)
  fi
}

function opoff() {
  op signout
  unset OP_SESSION_my
}

function opgp() {
  opon
  op get item "$1" --fields password
}

function op1t() {
  opon
  op get totp "$1"
}
```

- one last bit because I am a lazy typer, pbcopy so that I would not have to manually copy passwords to put them into another shell window
  - [pbcopy](https://osxdaily.com/2007/03/05/manipulating-the-clipboard-from-the-command-line/)

## ☁️ Cloud Outcome

- I now have a user in the identity account that can assume the developer role in another account in my organization

## Next Steps

Using this infrastructure, start creating a chatbot.

## Social Proof

[twitter](https://twitter.com/mejenks/status/1291479015526629383?s=20)
