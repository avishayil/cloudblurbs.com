---
layout: post
title: "Pro Tip: Allow Access to AWS Programmatic Accounts from Specific IP Addresses"
description: "Programmatic accounts are a bad idea in general, but at least AWS provides us a way to secure the access to them"
thumb_image: "aws-iam.png"
tags: [aws, iam, security, programmatic]
canonical_url: "https://medium.com/@avishayil/pro-tip-allow-access-to-aws-programmatic-accounts-from-specific-ip-addresses-59056a36dc0"
---

{% include image.html path="aws-iam.png" path-detail="aws-iam.png" alt="AWS IAM" %}

## Introduction ##

IAM policies are a great way to enforce authorization for Groups / Users / Roles to specific services, under specific conditions. In high level, policies are a set of JSON statements which provide certain permissions to entities. Adding another security layer to that, there is an optional block under the policy statements. Like any condition in any programming languages, The condition block returns a boolean output — either true or false, which decides whether a policy grants or denies the request.

In this example, we will demonstrate how a policy can deny or allow access from specific IP address, and how combining it with programmatic users (service accounts) adds more security to it in case of credentials theft.

For our use case, let’s think about the following scenario: An established enterprise company uses a Jenkins to deploy AWS resources on their Development, Staging and Production workloads. As part of the automated continues integration process, Jenkins uses AWS credentials to programmatically call the AWS API to perform its actions.
Now, let’s think what is going to happen if someone performs MITM attack and getting those credentials, or an irresponsible user saves those credentials to GitHub and a bot finds them? A lot of bad things might happen on these cases.

How are we mitigating these kind of attacks? In most cases, Jenkins will run from an organizational network, where the IP address(es) are known upfront. Simply be creating an IAM policy that rejects access if the request is coming from IP address that is not explicitly mentioned in the policy, we can mitigate any incident of stolen credentials.

## Getting Started ##

Create a new IAM policy and name it AWSAllowAccessFromIP. The policy will look as following (Update the IP address placeholder):

<script src="https://gist.github.com/avishayil/c5312f987bd2d7f118e50bed4fcc0391.js"></script>

Now, let’s attach this policy to our programmatic user (service account). This policy will automatically ensure that any request not originates from the mentioned IP address will be denied.
In our example, we’re running the AWS Security audit tool prowler. On the first run without applying the policy, these are the results:

{% include image.html path="iam-allowed.png" path-detail="iam-allowed.png" alt="IAM Allowed" %}

Now, after applying the policy, the AWS API won’t let us to access those actions since the request doesn’t originate from the explicitly mentioned IP address:

{% include image.html path="iam-denied.png" path-detail="iam-denied.png" alt="IAM Denied" %}

I encourage you to keep you AWS environments secure, by using the controls AWS provides.

Avishay.