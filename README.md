<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/sns/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# SNS Topic CloudFormation Blueprint

![CodeBuild](https://codebuild.us-west-2.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoiaENwN1g3enJDTVQzMnB1elRTK0tKSW4yaEZCd0hGLzZ5YSt5N1hGQlNvUUNKRHhYd29JRWNlRzhqblNSa0ltMmdhcUlYLzN4N2dHU1RQdHhRZDBycitZPSIsIml2UGFyYW1ldGVyU3BlYyI6Ilk0d2VSbTFXVDh3QWtIa0QiLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master)

[![BoltOps Badge](https://img.boltops.com/boltops/badges/boltops-badge.png)](https://www.boltops.com)

This blueprint provisions an SNS topic.

* All [AWS::SNS::Topic
](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sns-topic.html) properties are configurable with [Parameters](https://lono.cloud/docs/configs/params/). Additionally, properties that require further customization are configurable with [Variables](https://lono.cloud/docs/configs/shared-variables/).
* You can add subscriptions with the `@subscription` variable.

## Usage

1. Add blueprint to Gemfile
2. Configure: configs/sns values
3. Deploy

## Add

Add the blueprint to your lono project's `Gemfile`.

```ruby
gem "sns", git: "git@github.com:boltopspro/sns.git"
```

## Configure

First you want to configure the [configs](https://lono.cloud/docs/core/configs/) files. Use [lono seed](https://lono.cloud/reference/lono-seed/) to configure starter values quickly.

    LONO_ENV=development lono seed sns

To deploy to additional environments:

    LONO_ENV=production  lono seed sns

The generated files in `config/sns` folder look something like this:

    configs/sns/
    ├── params
    │   ├── development.txt
    │   └── production.txt
    └── variables
        ├── development.rb
        └── production.rb

Here's an example of the params config:

configs/sns/params/development.txt

    # Parameter Group: AWS::SNS::Topic
    # DisplayName= # my display name
    # KmsMasterKeyId= # 1234abcd-12ab-34cd-56ef-1234567890ab # also arn:aws:kms:us-east-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
    # TopicName= # my-topic-name

## Deploy

Use the [lono cfn deploy](http://lono.cloud/reference/lono-cfn-deploy/) command to deploy. Example:

    LONO_ENV=development lono cfn deploy sns --blueprint sns --sure
    LONO_ENV=production  lono cfn deploy sns --blueprint sns --sure

If you are using One AWS Account, use these commands instead: [One Account](docs/one-account.md).

## Configure: More Details

### Subscribers

You can use the `@subscription` variable to create subscribers. Example:

configs/sns/variables/development.rb:

```ruby
@subscription = [{
  Endpoint: "me@example.com", # String. Examples: http | https | email | email | sms | sqs | application | lambda
  Protocol: "email", # String
}]
```

Refer to the [SNS Subscribe API](https://docs.aws.amazon.com/sns/latest/api/API_Subscribe.html) docs for more subscription types.

### IAM Permissions

You can also adjust the AWS::SNS::TopicPolicy IAM Document with the `@policy_document` variable. Example:

```ruby
@policy_document =<<~JSON
  {
    "Version": "2008-10-17",
    "Id": "example-ID",
    "Statement": [
      {
        "Sid": "example-statement-ID",
        "Effect": "Allow",
        "Principal": {
          "Service": "s3.amazonaws.com"
        },
        "Action": [
          "SNS:Publish"
        ],
        "Resource": "${SnsTopic}",
        "Condition": {
          "ArnLike": {
            "aws:SourceArn": "arn:aws:s3:*:*:my-test-bucket"
          }
        }
      }
    ]
  }
JSON
```

In this example, it allows s3 to publish to the SNS topic. Make sure to replace the `my-test-bucket` in the example.

### Stack Name Convention

By leveraging the lono Stack Name and [CLI conventions](https://lono.cloud/docs/conventions/cli/), we can organize the configs files in a way that matches the stack name. Example:

    lono cfn deploy news --blueprint sns
    lono cfn deploy alerts --blueprint sns

Will use the corresponding config files:

    configs/sns/development/news.txt
    configs/sns/development/alerts.txt

## IAM Permissions

The IAM permissions required for this stack are described below.

Service | Description
--- | ---
cloudformation | To launch the CloudFormation stack.
sns | sns instance and security group.
s3 | Lono managed s3 bucket
