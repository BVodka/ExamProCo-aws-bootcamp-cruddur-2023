# Week 0 — Billing and Architecture
# Required Homework

### Installed AWS CLI

- Installed the AWS CLI via the Gitpod enviroment.
- Setup AWS CLI to use partial autoprompt mode.
- The bash commands we are using are the same as the [AWS CLI Install Instructions]https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html


Updated my `.gitpod.yml` to include the following task.

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

### Created a new User and Generated AWS Credentials

- `Enable console access` for the user
- Create a new `Cloud Administrator` Group and applied `AdministratorAccess`
- Create the a user ```cloud_user``` and assigned it to the Cloud Administrator group
- Created Security credentials for the user and downloaded the .csv file


### Set Env Vars

I set up these credentials for the current bash terminal and persisted it afterwards. 
```
export AWS_ACCESS_KEY_ID=""
export AWS_SECRET_ACCESS_KEY=""
export AWS_DEFAULT_REGION=us-east-1
```

```
gp env AWS_ACCESS_KEY_ID=""
gp env AWS_SECRET_ACCESS_KEY=""
gp env AWS_DEFAULT_REGION=us-east-1
```

### I verified that AWS CLI is working and I am the expected user using the command below

```sh
aws sts get-caller-identity
```

## Enabled Billing and created billing alerts 
![Created a budget for monthly expenditure](assets/TrackCreditSpend.JPG)

### Created SNS Topic

## SNS Topic
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

Checked email and confirmed the subscription

### Created Alarm

- [aws cloudwatch put-metric-alarm](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html)
- [Create an Alarm via AWS CLI](https://aws.amazon.com/premiumsupport/knowledge-center/cloudwatch-estimatedcharges-alarm/)
- Update the configuration json script with the TopicARN we generated earlier
- We are just a json file because --metrics is is required for expressions and so its easier to us a JSON file.

```sh
aws cloudwatch put-metric-alarm --cli-input-json file://aws/json/alarm_config.json
```

## Created an AWS Budget

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
[Link to Logical Diagram of the Crudder App](https://lucid.app/lucidchart/ec7f7d0f-bdad-4ae3-ac43-79ed2cc61e09/edit?viewport_loc=-662%2C8%2C2594%2C1302%2C0_0&invitationId=inv_08a823df-0043-46fd-83bf-8a86cea7fc42) 
