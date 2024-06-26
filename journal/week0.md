# Week 0 — Billing and Architecture


## Getting the AWS CLI Working

We'll be using the AWS CLI often in this bootcamp,
so we'll proceed to installing this account.


### Install AWS CLI

- We are going to install the AWS CLI when our Gitpod enviroment lanuches.
- We are are going to set AWS CLI to use partial autoprompt mode to make it easier to debug CLI commands.
- The bash commands we are using are the same as the [AWS CLI Install Instructions]https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html


Update our `.gitpod.yml` to include the following task.

```sh
tasks:
  - name: aws-cli
    env:
      AWS_CLI_AUTO_PROMPT: on-partial
    init: |
      cd /workspace
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install
      cd $THEIA_WORKSPACE_ROOT
```

We'll also run these commands indivually to perform the install manually

### Create a new User and Generate AWS Credentials

- Go to (IAM Users Console](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/users) andrew create a new user
- `Enable console access` for the user
- Create a new `Admin` Group and apply `AdministratorAccess`
- Create the user and go find and click into the user
- Click on `Security Credentials` and `Create Access Key`
- Choose AWS CLI Access
- Download the CSV with the credentials

### Set Env Vars

We will set these credentials for the current bash terminal
```
export AWS_ACCESS_KEY_ID=""
export AWS_SECRET_ACCESS_KEY=""
export AWS_DEFAULT_REGION=us-east-1
```

We'll tell Gitpod to remember these credentials if we relaunch our workspaces
```
gp env AWS_ACCESS_KEY_ID=""
gp env AWS_SECRET_ACCESS_KEY=""
gp env AWS_DEFAULT_REGION=us-east-1
```

### Check that the AWS CLI is working and you are the expected user

```sh
aws sts get-caller-identity
```

You should see something like this:
```json
{
    "UserId": "***REMOVED***",
    "Account": "***REMOVED***",
    "Arn": "arn:aws:iam::***REMOVED***:user/***REMOVED***"
}
```

## Enable Billing 

We need to turn on Billing Alerts to recieve alerts.


- In your Root Account go to the [Billing Page](https://console.aws.amazon.com/billing/)
- Under `Billing Preferences` Choose `Receive Billing Alerts`
- Save Preferences


## Creating a Billing Alarm

### Create SNS Topic

- We need an SNS topic before we create an alarm.
- The SNS topic is what will delivery us an alert when we get overbilled
- [aws sns create-topic](https://docs.aws.amazon.com/cli/latest/reference/sns/create-topic.html)

We'll create a SNS Topic
```sh
aws sns create-topic --name billing-alarm
```
which will return a TopicARN

We'll create a subscription supply the TopicARN and our Email
```sh
aws sns subscribe \
    --topic-arn TopicARN \
    --protocol email \
    --notification-endpoint your@email.com
```

Check your email and confirm the subscription

#### Create Alarm

- [aws cloudwatch put-metric-alarm](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html)
- [Create an Alarm via AWS CLI](https://aws.amazon.com/premiumsupport/knowledge-center/cloudwatch-estimatedcharges-alarm/)
- We need to update the configuration json script with the TopicARN we generated earlier
- We have a json file because --metrics it is required for expressions and so its easier to use a JSON file.

```sh
aws cloudwatch put-metric-alarm --cli-input-json file://aws/json/alarm_config.json
```

## Create an AWS Budget

[aws budgets create-budget](https://docs.aws.amazon.com/cli/latest/reference/budgets/create-budget.html)

Get your AWS Account ID
```sh
aws sts get-caller-identity --query Account --output text
```

- Supply your AWS Account ID
- Update the json files
- This is another case with AWS CLI its just much easier to json files due to lots of nested json

```sh
aws budgets create-budget \
    --account-id AccountID \
    --budget file://aws/json/budget.json \
    --notifications-with-subscribers file://aws/json/budget-notifications-with-subscribers.json
```

## Removing sensitive data from a repository

### Reference -
- [docs.github.com](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)
- [formulae.brew.sh](https://formulae.brew.sh/formula/bfg)

### Steps -
- brew install bfg
- git clone --mirror https://github.com/DSBhadoria/aws-bootcamp-cruddur-2023.git awsbootcamp
- cd awsbootcamp
- bfg --replace-text ./../passwords.txt
- git reflog expire --expire=now --all && git gc --prune=now --aggressive
- git push

Here'passwords.txt' file contains all the sensitive data that intends to be removed from the Git history.

![image](https://github.com/DSBhadoria/aws-bootcamp-cruddur-2023/assets/15330248/82bdc549-3e3e-4ef6-8b9c-f41ae4ff34fe)

![image](https://github.com/DSBhadoria/aws-bootcamp-cruddur-2023/assets/15330248/f39afd9e-21a0-413a-97c3-7cf28dd556d6)

![image](https://github.com/DSBhadoria/aws-bootcamp-cruddur-2023/assets/15330248/56eabf20-2372-4f4b-a66f-171665e1d3e7)

## Recreate Logical Architectural Deisgn
![image](https://github.com/DSBhadoria/aws-bootcamp-cruddur-2023/assets/15330248/3a45f55d-5bb2-459c-8c78-0f5f95bb7809)

Lucid Chart - https://lucid.app/lucidchart/73cbd47e-6a1c-407c-978c-534fc604876c/edit?viewport_loc=-446%2C-199%2C2750%2C1399%2C0_0&invitationId=inv_bf8f9484-3016-4eb3-b498-a6e64fa603ba
