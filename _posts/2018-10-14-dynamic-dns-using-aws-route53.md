---
layout: post
title: "Dynamic DNS Using AWS Route 53"
description: "With this solution, your home workloads doesn't require purchasing a static IP from your ISP"
thumb_image: "route53.png"
tags: [route53, aws, dns, domain]
canonical_url: "https://medium.com/@avishayil/dynamic-dns-using-aws-route-53-60a2331a58a4"
---

{% include image.html path="route53.png" path-detail="route53.png" alt="Route53" %}

## Introduction ##

Until not long ago, I was using DuckDNS services to get my home webserver up and running. As my ISP is demanding a respectful amount of money to reserve a static IP address I was looking for dynamic dns solutions to point my home IP address to.

I chose an AWS service because I had an account there and I already have some services running over there. Some LightSail websites I’m running for personal use, Elastic IP’s etc. That’s why it was comfortable for me to use Route 53 and get consolidated billing instead of splitting it into another service. I did not made a price comparison for this specific need, but if it matters for you, you should do it across cloud platform and choose the best solution for you.

## Getting Started ##

### Set up a Hosted Zone in AWS Route 53 ###

We assume that you own a domain name. If you don’t, you could buy one at one of the domain name registrars, or at AWS Route 53 itself. If you bought it outside of AWS, you need to set up a Hosted Zone using Route 53 and copy the records to the DNS manager of your domain name registrar control panel.

First of all, you’ll need to login to the AWS console, then head Route 53. Over there, head to Hosted Zones, then create on the blue button that says “Create Hosted Zone”. Fill the domain name and leave Type as Public Hosted Zone.

When you click on your newly created Hosted Zone, you’ll typically see two records - NS record(s) and SOA record(s). The ones that matters for now would be the NS records - Take a note of them. Head to your current domain name registrar, then put the NS records you copied before on the appropriate place on the DNS control panel of your registrar. You may need to contact your registrar support and ask them for help doing that.

Usually, these changes will take around 24–48 hours to propagate globally, so be patient. In the meanwhile, we can go to the other part of the guide and see how can we work with the AWS CLI and dynamically change the A record on our domain name.

### Set up Your Home Server to Query AWS Route 53 ###

First of all, if you didn’t have AWS CLI running on your machine until now, this is the time to install it. You got plenty of ways to do that and AWS documentation covers the best of them here: [https://docs.aws.amazon.com/cli/latest/userguide/installing.html](https://docs.aws.amazon.com/cli/latest/userguide/installing.html)

Since we want to query Route53 from our environment, we’ll have to make sure that we set up the proper credentials to do that. It is advised to create an IAM user and assign it an IAM role that specifically allows to make changes on Route53. Once we have that user and his credentials, we can configure our machine using aws configure and type in the credentials.

After we’re done with the credentials, we need to set up the script that will check for ip changes and query Route53, if applicable. The script will look like that, slightly changed from the amazing post that Will Warren wrote:

[https://willwarren.com/2014/07/03/roll-dynamic-dns-service-using-amazon-route53/](https://willwarren.com/2014/07/03/roll-dynamic-dns-service-using-amazon-route53/)

<script src="https://gist.github.com/avishayil/d24e6475a0ddf674adbbcbdcd7be9029.js"></script>

Few important variables we need to notice:

- **ZONEID** — The ID of the Hosted Zone we created earlier on Route53.
- **RECORDSET** — The CNAME we would like to update while the script runs.
- **PATH** — Might be needed for AWS CLI to run properly.

Now that we have our script, let’s try to run it! If it succeeds, we’re going to see that change on the CNAME with the updated ip address of our machine.
Next step will be to set up a cron job that will make sure we’re updated every few minutes. For that, let’s run crontab -e and add the following on a new line:

  `*/1 * * * * ~/path/to/route.sh >/dev/null 2>&1`

  That line will make sure that we’re checking every minute that our ip address haven’t been changed. If it did, it will immediately query Route53 and update the necessary record to the updated value.
  