# lambda-cloudwatch-slack

An [AWS Lambda](http://aws.amazon.com/lambda/) function for better
Slack
notifications. [Check out the blog post](https://assertible.com/blog/npm-package-lambda-cloudwatch-slack).

[![BuildStatus](https://travis-ci.org/assertible/lambda-cloudwatch-slack.png?branch=master)](https://travis-ci.org/assertible/lambda-cloudwatch-slack)
[![NPM version](https://badge.fury.io/js/lambda-cloudwatch-slack.png)](http://badge.fury.io/js/lambda-cloudwatch-slack)


## Overview

This function was originally derived from the
[AWS blueprint named `cloudwatch-alarm-to-slack`](https://aws.amazon.com/blogs/aws/new-slack-integration-blueprints-for-aws-lambda/). The
function in this repo improves on the default blueprint in several
ways:

**Better default formatting for CloudWatch notifications:**

![AWS Cloud Notification for Slack](https://github.com/assertible/lambda-cloudwatch-slack/raw/master/images/cloudwatch.png)

**Support for notifications from Elastic Beanstalk:**

![Elastic Beanstalk Slack Notifications](https://github.com/assertible/lambda-cloudwatch-slack/raw/master/images/elastic-beanstalk.png)

**Support for notifications from Code Deploy:**

![AWS CodeDeploy Notifications](https://github.com/assertible/lambda-cloudwatch-slack/raw/master/images/code-deploy.png)

**Basic support for notifications from ElastiCache:**

![AWS ElastiCache Notifications](https://github.com/assertible/lambda-cloudwatch-slack/raw/master/images/elasticache.png)

**Support for encrypted and unencrypted Slack webhook url:**


## Configuration

Clone this repository and then follow the steps below:


### 1. Configure AWS environment

Fill in the variables at the top of the `Makefile.vars.example`. For example, your
variables may look like this:

```
LAMBDA_FUNCTION_NAME=cloudwatch-to-slack
AWS_REGION=us-west-2
AWS_ROLE=arn:aws:iam::123456789123:role/lambda_exec_role
AWS_PROFILE=default
```

When you're done, copy the file to `Makefile.vars`:

```
$ cp Makefile.vars.example Makefile.vars
```

### 2. Setup Slack hook

Follow these steps to configure the webhook in Slack:

  1. Navigate to
     [https://slack.com/services/new](https://slack.com/services/new)
     and search for and select "Incoming WebHooks".

  3. Choose the default channel where messages will be sent and click
     "Add Incoming WebHooks Integration".

  4. Copy the webhook URL from the setup instructions and use it in
     the next section.

  5. Click 'Save Settings' at the bottom of the Slack integration
     page.

### 3. Configure AWS Lambda script

Next, open `deploy.env.example`, there are several configuration
options here. At a minimum, you must fill out `UNENCRYPTED_HOOK_URL`
(or `KMS_ENCRYPTED_HOOK_URL`) and `SLACK_CHANNEL` (the name of the Slack room to send messages).

When you're done, copy the file to `deploy.env`:

```
$ cp deploy.env.example deploy.env
```

#### Encrypted the Slack webhook URL

If you don't want or need to encrypt your hook URL, you can use the
`UNENCRYPTED_HOOK_URL`.  If this variable is specified, the
`KMS_ENCRYPTED_HOOK_URL` is ignored.

If you **do** want to encrypt your hook URL, follow these steps to
encrypt your Slack hook URL for use in this function:

  1. Create a KMS key -
     http://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html.

  2. Encrypt the event collector token using the AWS CLI.
     `$ aws kms encrypt --key-id alias/<KMS key name> --plaintext "<SLACK_HOOK_URL>"`

     Note: You must exclude the protocol from the URL
     (e.g. "hooks.slack.com/services/abc123").

  3. Copy the base-64 encoded, encrypted key (CiphertextBlob) to the
     ENCRYPTED_HOOK_URL variable.

  4. Give your function's role permission for the kms:Decrypt action.
     Example:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1443036478000",
            "Effect": "Allow",
            "Action": [
                "kms:Decrypt"
            ],
            "Resource": [
                "<your KMS key ARN>"
            ]
        }
    ]
}
```


### 4. Deploy to AWS Lambda

The final step is to deploy the integration to AWS Lambda:

    make deploy

## Tests

With the variables filled in, you can test the function:

```
npm install
make test
```

## Caveats

- Environment variables specified in `deploy.env` may not show up on
  AWS Lambda but are still in use.

- `node-lambda` appends `-development` to Lambda function names. To
  fix this, check out the `.env` file created by `node-lambda` and set
  the `AWS_ENVIRONMENT` var to an empty string, like
  `AWS_ENVIRONMENT=`

## License

MIT License
