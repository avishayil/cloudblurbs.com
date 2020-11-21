---
layout: post
title: "Using AWS Lambda To Auto Recover Unresponsive Multi-AZ RDS Cluster"
description: "Investigating RDS cluster inresponsivness can lead to a long downtime. Here is an automation that allows your site to stay up."
thumb_image: "aws.png"
tags: [lambda, aws, rds, sns, cloudwatch]
canonical_url: "https://medium.com/@avishayil/using-aws-lambda-to-auto-recover-unresponsive-multi-az-rds-cluster-dca09efed70a"
---

{% include image.html path="aws.png" path-detail="aws.png" alt="Amazon Web Services" %}

## Introduction ##

One of the obstacles I encountered during moving our whole server setup from Hosting services to AWS, is the Multi Availability Zones setup and the fact that it wasn’t redundant to other events, rather than shutdowns / unexpected falls.

When trying to save some $$ for my company, I decided to use multi-az t2.medium RDS cluster. This setup gave me, except for the redundancy and quick recovery during Layer-7 attacks, the ability to promote the replica database to master when I was running out of CPU credits for the instance I was using. This happened usually once a day, depends on the traffic our website experienced, and the “solution” was to manually login to the RDS console and trigger a failover.

This worked well for a few days, until I figured out that there is no out-of-the-box, automated solution to trigger this failover when the instance experienced high select latencies (starting from >10ms) and makes our website not responsive for some of our readers. Also, our Elastic Beanstalk application went to warning state, stating that around 5% of the users are receiving 5xx errors trying to access the web application.

I’ll try to explain here what I did to achieve the situation, and you can take this example and customize it for your own needs. Of course, there are some improvements that can be made, but because reasons of time and other requirements some of the code could be written better.

## Tackling the Problem ##

This walkthrough involves the following AWS services:
1. **RDS** — Relational Database Service
2. **CloudWatch** — Cloud & Network Monitoring Services
3. **SNS** — Simple Notification Service
4. **Lambda** — Serverless Computing Platform

The idea is to create a Lambda function, that will be triggered by SNS when a CloudWatch Alarm is going to ALARM state, then it will use the function to connect to the RDS database, discover what instance is acting as the reader replica, and promote it to writer. Sounds complicated? in fact it doesn’t.

Here are the instruction to configure each AWS service:

### RDS ###

Make sure that your’e using a multi az RDS Cluster
You have to make sure that your RDS instance is a Cluster. How can you tell that? just head to the RDS console and select ‘Clusters’ on the side menu. You should see your RDS cluster there. When you’ll click on the cluster you’ll notice the DB Cluster and Cluster Endpoint parameters, please write both of them down, and make sure you remember your username and password, to access the cluster.

{% include image.html path="rds-cluster.png" path-detail="rds-cluster.png" alt="RDS Cluster" %}

### CloudWatch ###

Your’e going to set a new Alarm — for my use case it was select latency. You’ll need to access the CloudWatch Management Console, choose ‘Create Alarm’ then select the appropriate metric — in our case that would be ‘SelectLatency’ under ‘RDS > DBClusterIdentifier’ metrics category. You’ll need to create 2 alarms — one for each database, the master and the replica in order to make sure they switch.

{% include image.html path="cloudwatch-create-alarm.png" path-detail="cloudwatch-create-alarm.png" alt="Create CloudWatch Alarm" %}

{% include image.html path="cloudwatch-define-alarm.png" path-detail="cloudwatch-define-alarm.png" alt="Define CloudWatch Alarm" %}

### SNS ###

You’ll need to create a new topic and a new subscription, in order to subscribe to the Alarm and trigger the Lambda function, as well as receiving an email notification every time the alarm is changing it’s state to ALARM.

{% include image.html path="sns.png" path-detail="sns.png" alt="SNS Topic" %}

### Lambda ###

You’re going to create a new lambda function that utilizes the RDS object of the AWS SDK library for Node.js. The function will create a connection to the mysql cluster, see which database is promoted to writer, then use the failover function to switch between the writer and reader. Because the code requires an external library to run (mysql) you’ll need to run npm install mysql on a local project, then upload the whole function folder as a zip file to Lambda.

<script src="https://gist.github.com/avishayil/27f16de2fdb773c95940bb938109dff4.js"></script>

On the function triggers, you’ll be able to choose the SNS you defined earlier:

{% include image.html path="lambda-triggers.png" path-detail="lambda-triggers.png" alt="Lambda Triggers" %}

You could also set up CloudWatch to run the function only when you are asleep, and even receive notifications to your phone and do it manually during working hours, so this setup would execute only at night time.

After you finished the set up, you should just sit, and see how the function performs. We use this in production, and the result is great: if one database falls because of an attack or any other reason, the downtime minimizes to minimum and the database recovers automatically.

Avishay.
