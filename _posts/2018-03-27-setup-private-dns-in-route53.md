---
layout: post
title: "Setup Your Private Domain Name System in AWS"
description: "A handful guide on setting your own DNS system using AWS Route53"
thumb_image: "route53.png"
tags: [route53, aws, dns, domain]
canonical_url: "https://medium.com/@avishayil/setup-your-private-domain-name-system-in-aws-6b6eda68ccc0"
---

{% include image.html path="route53.png" path-detail="route53.png" alt="Route53" %}

## Introduction ##

During my work, I encountered several times in a need of architecting & developing internal systems, which requires access from specific people inside the company. In terms of visibility & security, you probably don’t want these systems to be available to the internet, and therefore you must find a solution.

On AWS, you have the option to create private subnets inside your VPC and access the application through VPN. That seems like a nice solution, right? but when you want to publish the application internally, you probably don’t want to deliver it with an internal IP address such as `10.0.0.79`.

In the solution offered in this post, we’ll learn how to use **AWS Route53 Private Hosted Zones**, alongside with Amazon Active Directory Services in order to achieve a solution that will make the domain `ec2.localhost` visible to the people residing inside your VPC, with the help of the VPN connection.

## Getting Started ##

### Prerequisites ###

Make sure you’re in a region with supports Directory Service Simple AD, We’ll be creating our infrastructure in Ireland for this guide purposes. For a full list of regions supporting Simple AD, visit the [3] reference and search for Simple AD.

### VPC ###

- Create VPC and allocate an IP CIDR (10.0.0.0/16). I found this tool useful to translate IP CIDR to IP range: https://www.ipaddressguide.com/cidr

- Create two subnets on two different availability zones. If you’ll accidentally create the two subnets on the same availability zone, you’ll have to recreate one of them, which concludes removing all instances on the unneeded availability zone. For example:
  - First Subnet CIDR — `10.0.0.0/24`
  - Second subnet CIDR — `10.0.1.0/24`

- Create a route table and make sure you’re associating at least the first subnet to it, so the instances that we’ll want to access will be available for local access (Communication between instances) and internet access (Software Updates, VPN).

- Create an Internet Gateway and associate it to your VPC. Update the route table you created earlier with association to the IGW you just created. You want to IGW to access the whole internet, so your VPN server will be able to operate normally with computer anywhere in the world. For that reason, you’d probably want to specify `0.0.0.0/0` as you’re destination CIDR.

- Create a Virtual Private Gateway (VGW) and associate it with your VPC.
Update you’re route table the VGW will be able to access all the instances inside the VPC — CIDR `10.0.0.0/8`.

### EC2 ###

- Create an EC2 instance and associate it with the VPC you created earlier. Make sure that the security group you’re associating to the EC2 instance allows access only to CIDR inside the VPC. For testing purposes, allow access on port 22 to the CIDR: `10.0.0.0/8` which will allow access form any address range of `10.0.0.0–10.255.255.255`.

### Route53 ###

- Go to Route 53 and create new private hosted zone. Associate the hosted zone with your VPC. Name the hosted zone as the domain name you’d like to privately access (For example, ec2.localhost). you can choose any name you want since you’re going to resolve this domain name from your own DNS server you’re going to create in a few minutes.

- Create a new A record on your newly created private hosted zone. Name it whatever you want (You can keep the name empty for resolving ec2.localhost). Point the A record to the IP address of the ec2 instance you created earlier.

### Directory Service ###

- Go to Amazon Directory Service. Create a new Simple AD (With whatever name you’ll choose) and associate it with your VPC, on both subnets you created earlier.

- Take a note of the DNS addresses you got from the Simple AD. You’ll need to specify them on your local network adapter so you’ll have access to the private hosted zone on Route 53 that you created earlier.

### VPN and DNS routing ###

- Create another EC2 instance — this is going to be the VPN server you’re going to connect every time you’ll want to access resources inside your VPC. Hackernoon provided [a wonderful guide](https://hackernoon.com/using-a-vpn-server-to-connect-to-your-aws-vpc-for-just-the-cost-of-an-ec2-nano-instance-3c81269c71c2) details how to create this kind of server quickly, please follow the steps provided on the blog posts and continue to the next section.

- Configure your OpenVPN server — using the configuration file /etc/openvpn/server.conf, configure the server to push route to your VPC, uncommenting the following line: `push "route 10.0.0.0 255.255.255.0"`. Next, you’ll need to configure the server to point the requests to your newly created DNS server from Simple AD you configured in the Directory Service step. Uncomment and edit the following line: `push "redirect-gateway def1"`. Then, define the addresses: `push "dhcp-option DNS 10.0.x.x"` twice, using the DNS addresses you noted.

### References ###
- [Creating a private hosted zone on Amazon Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/hosted-zone-private-creating.html)
- [Create Simple AD on Amazon Directory Service](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/gsg_create_directory.html)
- [Directory Service Regions](https://docs.aws.amazon.com/general/latest/gr/rande.html#ds_region)

Avishay.
