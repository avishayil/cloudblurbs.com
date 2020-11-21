---
layout: post
title: "Deploy Elastic Beanstalk Application using Bitbucket Pipelines"
description: "You have an Elastic Beanstalk applications and your'e using bitbucket for source control? here's your way to automate your deployment pipeline"
thumb_image: "bitbucket-pipelines-header.jpg"
tags: [bitbucket, elastic-beanstalk, pipelines, aws]
canonical_url: "https://medium.com/@avishayil/deploy-to-elastic-beanstalk-using-bitbucket-pipelines-189eb75cf052"
---

{% include image.html path="bitbucket-pipelines-header.jpg" path-detail="bitbucket-pipelines-header.jpg" alt="React Native" %}

## Introduction ##

For a long period of time, I was using custom git hooks on my own ec2 instance that ran Gogs open source software as a git server. I was using these hooks to make sure that every push to the repository goes through and being deployed to our staging server.

Recently, I started to implement Atlassian products for our work platform. We’re using Confluence for shared work, Jira for agile deployment and customer service, and Bitbucket for source code management. I was looking for solutions to offload my old git server that was taking an ec2 machine. Other advantages were the tight integration that Jira has to offer with Bitbucket, closing issues with commits, etc.

Bitbucket had announced not-long-ago Pipelines as their new product. Basically, you can use custom yaml files to describe what happens after you push code to your repository. Bitbucket will lift some instance and allocate computing power for you, for running the code you describe in the yaml, and you’re paying for the time it takes to run. You can also use some caching techniques to make sure you don’t download large libraries over and over again.

## Getting Started ##

### Creating IAM Role to enable secure access to your AWS resources ###

First of all, we’ll need to create the credentials that would be used to access your S3 and Elastic Beanstalk resources on AWS. In order to do that, we’ll need to create a new user account (let’s say, bitbucket) on your AWS account and provide it the following access:

- **S3**: ListBucket, GetBucketLocation, GetObject and PutObject to the specific S3 resource.
- **Elastic Beanstalk**: Create application version, and update environment with the appropriate application version. Make sure you select the specific environment you want to update (in this case, staging).
- **Other Services**: Not sure how to explain each and every service, but after ping-pong with AWS support and hard work trying all configurations that was the most preferable.

The JSON looks as follows:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "s3:Get*",
                "s3:CreateBucket",
                "sns:Unsubscribe",
                "elasticbeanstalk:CreateApplicationVersion",
                "cloudformation:CreateChangeSet",
                "autoscaling:*",
                "s3:List*",
                "cloudwatch:Describe*",
                "sns:OptInPhoneNumber",
                "sns:CheckIfPhoneNumberIsOptedOut",
                "cloudformation:ContinueUpdateRollback",
                "s3:GetObjectAcl",
                "sns:ListEndpointsByPlatformApplication",
                "sns:SetEndpointAttributes",
                "rds:Describe*",
                "elasticbeanstalk:DescribeEnvironments",
                "sns:DeletePlatformApplication",
                "sns:SetPlatformApplicationAttributes",
                "cloudformation:Estimate*",
                "cloudformation:UpdateStack",
                "elasticloadbalancing:Describe*",
                "sns:Subscribe",
                "sns:ConfirmSubscription",
                "s3:DeleteObject",
                "cloudformation:List*",
                "elasticloadbalancing:RegisterInstancesWithLoadBalancer",
                "cloudformation:ExecuteChangeSet",
                "sns:ListSubscriptionsByTopic",
                "sns:CreateTopic",
                "sns:GetPlatformApplicationAttributes",
                "cloudformation:SignalResource",
                "elasticbeanstalk:Describe*",
                "elasticbeanstalk:DeleteApplicationVersion",
                "elasticbeanstalk:CreateStorageLocation",
                "sns:GetSubscriptionAttributes",
                "sns:DeleteEndpoint",
                "s3:PutObject",
                "s3:GetObject",
                "sns:ListPhoneNumbersOptedOut",
                "sns:GetEndpointAttributes",
                "cloudformation:DeleteStack",
                "elasticbeanstalk:List*",
                "elasticbeanstalk:UpdateEnvironment",
                "sns:GetSMSAttributes",
                "cloudformation:CreateUploadBucket",
                "elasticbeanstalk:Check*",
                "cloudformation:CancelUpdateStack",
                "sns:DeleteTopic",
                "sns:ListTopics",
                "sns:CreatePlatformEndpoint",
                "cloudformation:UpdateTerminationProtection",
                "s3:ListBucket",
                "sns:SetTopicAttributes",
                "s3:GetBucketPolicy",
                "cloudformation:DeleteChangeSet",
                "elasticbeanstalk:RequestEnvironmentInfo",
                "sns:Publish",
                "cloudwatch:Get*",
                "s3:PutObjectAcl",
                "sns:GetTopicAttributes",
                "sns:CreatePlatformApplication",
                "sns:SetSMSAttributes",
                "cloudwatch:List*",
                "sns:ListSubscriptions",
                "cloudformation:Describe*",
                "cloudformation:PreviewStackUpdate",
                "ec2:Describe*",
                "sns:SetSubscriptionAttributes",
                "cloudformation:Validate*",
                "cloudformation:CreateStack",
                "s3:PutBucketPolicy",
                "elasticbeanstalk:RetrieveEnvironmentInfo",
                "s3:GetBucketLocation",
                "sns:ListPlatformApplications",
                "cloudformation:Get*"
            ],
            "Resource": "*"
        }
    ]
}
```

{% include image.html path="bitbucket-pipelines-policy.png" path-detail="bitbucket-pipelines-policy.png" alt="Bitbucket Pipelines Policy" %}

After you’re done, keep the Access Key ID and Secret Access Key, you’re going to use them later.

### Enabling pipelines in the Bitbucket repository ###

Before dive in to the details, we need to enable Pipelines in our repository and define some environment variable to work with. We’ll go to Pipelines on the left menu, than choose Python and Enable button to get started.

{% include image.html path="bitbucket-pipelines.png" path-detail="bitbucket-pipelines.png" alt="Bitbucket Pipelines" %}

{% include image.html path="bitbucket-pipelines-language.png" path-detail="bitbucket-pipelines-language.png" alt="Bitbucket Pipelines Language" %}

We’ll see an empty build going on. After you’re done, you’ll notice that the application created a new S3 bucket for your use, usually named around elasticbeanstalk-eu-central-1-account-id. Take a note of this name.

Now, we need to set the necessary environment variables for the script to work properly. We’re going to use the Access Key ID and Secret Access Key we noted earlier, and the S3 Bucket name we got from the first pipelines automatic build:

{% include image.html path="bitbucket-pipelines-environment-variables" path-detail="bitbucket-pipelines-environment-variables.png" alt="Bitbucket Pipelines Environment Variables" %}

Next, we’ll fetch the changes from the git repository and edit the needed files.

Basically, we’re talking about two main files you’ll need to place on the root folder of the application, courtesy of awslabs. The first one is the famous `bitbucket-pipelines.yml`. That Bitbucket Pipelines created for us. We need to alter it a little bit to the following example:

[bitbucket-pipelines.yml](https://bitbucket.org/awslabs/aws-elastic-beanstalk-bitbucket-pipelines-python/src/e646d081c485c57801feeda9a16b2de2134668eb/bitbucket-pipelines.yml)

Let’s talk about this file a little bit, since the other file will not change in most cases.

- The first 3 lines are self explanatory. Perform update on the running instance to make sure we can zip the application, than install the zip library.
- After that, we use python package manager, pip to install the boto3 library, which includes the necessary commands connecting to the aws sdk.
- After that, We’ll zip the application and then run the second file, beanstalk_deploy.py which contains three functions:
  1. `upload_to_s3` — which is responsible for uploading the zipped application to Amazon S3 storage. The function uses the environment variable `S3_BUCKET` we defined before and uploading the package to this bucket. The name will be the application name from Elastic Beanstalk, with a timestamp and the suffix `bitbucket_builds.zip`.
  2. `create_new_version` — responsible for creating a new application version in Elastic Beanstalk.
  3. `deploy_new_version` — Takes the zipped package we uploaded to S3 before, than deploys it to the new application version we created in the `create_new_version` function.

[beanstalk-deploy.py](https://bitbucket.org/awslabs/aws-elastic-beanstalk-bitbucket-pipelines-python/src/e646d081c485c57801feeda9a16b2de2134668eb/beanstalk_deploy.py)

You’re more than welcome to change the `bitbucket-pipelines.yml` file and suit it for your needs. After you’re done, just commit, push, and see your buildscript in action.

Avishay.
