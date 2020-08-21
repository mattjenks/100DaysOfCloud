# Build a basic chatbot using aws lex, aws lambda, serverless and golang

## Introduction

I'm trying to upgrade my skills across the board. My company wants to go all in on automation but they are unsure where to get started. Thus, I thought I would start them off with a basic chatbot that can serve as an internal self service assistant.

## Prerequisite

- [AWS Lex](https://docs.aws.amazon.com/lex/latest/dg/what-is.html)
- [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [Serverless Framework](https://www.serverless.com/)
- [golang](https://golang.org/)

## Use Case

Provide a corporate self service assistant. This assistant must do things that currently take several tedious steps to do. The first thing it will do is request time off and fulfill our process for interacting with HR. Steps that are needed to accomplish this. (non checked items means it is on the todo list, but not covered in this writeup.)

- [x] build golang lambda functions that can fulfill lex intents
- [x] deploy golang lambda functions using serverless framework
- [x] create lex intents with appropriate slots for the use case
- [] implement the leave request intent business logic (in golang) to fulfil the lex intent
  - [] create a pdf from supplied slot information
  - [] email the pdf to HR
- [] implement an authorization scheme for the chatbot that ties to our office 365 cloud corporate database
- [] integrate authorization scheme and chatbot into MS teams as this is our primary collaboration tool
- [] might try and add a flutter front end! *[Hint: Amplify+flutter developer preview!](https://aws.amazon.com/blogs/mobile/announcing-aws-amplify-flutter-developer-preview/*

## Cloud Research

- continued reading [this book](https://www.oreilly.com/library/view/full-stack-serverless/9781492059882/)
https://docs.aws.amazon.com/lambda/latest/dg/golang-package.html
- lambda golang function name and handler name must match! I set handler to main and not the name of the executable file. At first I did not use the serverless framework and ran face first into this with aws command line

```zsh
# note function-name and handler are different! BAD!
aws lambda create-function --function-name leave --runtime go1.x --zip-file file://leave.zip --handler main --role arn:aws:iam::939257149235:role/lambda_basic_execution_role --profile randd_developer
```

- sls (serverless framework shorthand) use invoke local which uses docker on my mac when using golang
  - need to use --path vice -p [sls defect](https://github.com/serverless/serverless/issues/7871) to send test data

- I use aws cross account identities and MFA to [delegate access](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html).
  - I use one password for my TOTP based 1 time passwords via the cli application as seen [here](../010/Readme.md)
  - I created a quick one liner to authenticate into 1password, get a TOTP code and pipe it sls deploy (and paste it to the mac clipboard)
    - "op1trd | sed 's/%//g' | sls deploy -f leave"

## Try yourself

- mkdir self-service-chatbot
- sls create
- sls plugin install --name serverless-offline
- add a provider and use a role to access aws account (serverless.yml). Setup for that is described [here](../008/Readme.md).

```yaml
provider:
  name: aws
  runtime: go1.x
  region: us-east-1
  stage: dev
  profile: randd_developer
  memory: 128
  timeout: 30
```

- researched golang references
  - [AWS lex go](https://github.com/aws/aws-lambda-go/blob/master/events/lex.go)
  - []()
- created golang functions for three intents
  - Greetings
  - Help
  - ScheduledLeave 
- deployed functions taking MFa and role into account
  - "op1trd | sed 's/%//g' | sls deploy"

- in the lex console, created 3 intents
  - Greetings
  - Help
  - ScheduledLeave
- created appropriate slot types as depicted here: ![ScheduleLeave slot types](https://github.com/mattjenks/100DaysOfCloud/blob/main/Journey/016/scheduleleave_slottypes.png?raw=true)
- compiled and testing basic interactions. Worked!

## ☁️ Cloud Outcome

The setup and basic bot interaction are working with my lambda functions fulfilling the intents.

## Next Steps

- [] implement the leave request intent business logic (in golang) to fulfil the lex intent
  - [] create a pdf from supplied slot information
  - [] email the pdf to HR

## Social Proof

✍️ Show that you shared your process on Twitter or LinkedIn

[twitter](link)
