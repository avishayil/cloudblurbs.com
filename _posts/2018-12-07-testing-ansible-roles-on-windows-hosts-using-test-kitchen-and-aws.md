---
layout: post
title: "Testing Ansible Roles on Windows Hosts Using Test-Kitchen and AWS"
description: "Writing Ansible roles for Windows? here's a suggested architecture for integration tests"
thumb_image: "route53.png"
tags: [ansible, roles, windows, aws, testing]
canonical_url: "https://medium.com/@avishayil/testing-ansible-roles-on-windows-hosts-using-test-kitchen-and-aws-b6aa69e8b29"
---

{% include image.html path="ansible.png" path-detail="ansible.png" alt="Ansible" %}

## Introduction ##

Working a-lot with Ansible lately, I was encountering in a real blocking problem. As we were working on deploying our software on Windows targets, We were constantly trying to find ways to test our roles, but we encountered some issues, mostly related to our development environments. Writing these lines, i’m using my personal Macbook Air so don’t be confused because of that.

The fact we were working on a Cyber Security company requires compromises on development environments. Always working in a virtual environment has it’s limitations, and therefore it was not possible to use a solution that deploys VirtualBox VM’s, Docker or LXC containers, and manage all of these using Vagrant, for example. Therefore, I was looking for a way to provision machine using the existing resources on our team. I discovered that Test-Kitchen could help me doing that using it’s EC2 driver.

## Getting Started ##

### Test-Kitchen ###

Rule of thumb: Ansible roles are great, please use Ansible roles. I’m not going to elaborate on this furthermore, but just use roles whether it’s possible. Test-Kitchen is pretty easy to setup, and consists few steps:

1. Download ChefDK from the Chef download website, according to your platform (I assume that you’re using Linux or macOS as your development machine).
2. Install the chef gems for test-kitchen and it’s Ansible plugin. You can find further instructions on the Git repository in the end of this article.
3. Make sure you have Python installed. I personally recommend python 2.7.15, as we’re mostly using it for our projects. Whether your development machine is a VDI, virtual machine or even Cloud9 instance (where you get this out of the box).
4. Make sure you have AWS CLI installed and configured (using aws configure) with proper credentials to create and describe EC2 instances, security groups and keypairs. Try to use a sufficient and permissive user (but not on any production account, of course!).

### The .kitchen.yml File ###

Once we finished setting up our development environment, we want to describe how exactly we are going to provision the testing environment for our role and how to test it. This is why the .kitchen.yml file exists, Let’s take a look:

<script src="https://gist.github.com/avishayil/1640a0a528a01b49cbd356324afd2187.js"></script>

This YAML file is pretty-much self explanatory, but let’s focus on a few important things:

1. On the `windows-2016` platform, we can see the the `verifier` is `pester`. We get that ability out of the box with our setup, so we can use Pester tests to describe tests inside our Windows targets using PowerShell.
2. The `inventory/hosts` file does not exist yet, your’e absolutely right. That’s because we still haven’t run the script that would create that file dynamically, querying our AWS account.
3. The `provisioner` section of the code defines what will happen when we’ll run `kitchen converge`. In this case, we need that the `ansible-playbook` process will play our `default.yml` playbook on our dynamically-created inventory.
4. If you are working on a development machine, you’re more than welcome to copy this file and overwrite it using a `.kitchen.local.yml` file on the same directory. It will use the `.kitchen.yml` file as a base and will add differences on the local file, but remember — `.kitchen.local.yml` file is gitignored. Therefore, any change you’d like to contribute to the ci-cd process should be on `.kitchen.yml` file.
5. The `driver-config` section of the EC2 driver has a ton of options, such as using an IAM profile to give the created machine the necessary permissions to perform actions against the AWS ecosystem. You can see all the configuration options [here](https://github.com/test-kitchen/kitchen-ec2).

### Time to Get Dirty ###

Now, it’s time to clone our example repository and see how it works. On the example project, the following steps will allow us to create, test and destroy our testing environment as many times as we want. The Ansible role itself, defined on the repository is bloody simple: it will download and silently install Google Chrome on our Windows target.

The following steps will be included in the flow, that can be easily automated to be comply with CI-CD environments:

1. `kitchen create` — will create our environment on AWS. It will look for our default region, then launch the EC2 instances on the default VPC inside the default subnet (should be public). The demo environment contains an Ansible machine and Windows 2016 Server target host, both configured with user_data to make sure that they have the software needed to perform the necessary operations (Ansible with Windows hosts support on RHEL, WinRM allowed on the firewall and password changed to `Kitchen` on the Windows host).
2. Run the script `inventory/generate_inventory.sh` — It will dynamically create hosts file that would contain all the EC2 instances on our AWS account. We’re using tagging technique here — our Windows target launched with the tag `kitchen-type: windows`, what makes us easy to identify it using the dynamic inventory, and run the role on the right hosts using the default playbook.
3. `kitchen converge` — will transfer the role to our newly created Ansible machine, install Chef on the windows target and then Run the default playbook, defined on the `.kitchen.yml` file on the Windows host.
4. `kitchen verify` — will push the test files, located on `tests/integration` folder to the target machine, then run them. On this role, we’re using Pester to test our Windows target with PowerShell. The output will be how many tests run and passed, what will allow us to decide if the role succeeded or not.
5. `kitchen destroy` — will terminate all our instances, keypairs and security groups and leave our AWS environment clean and ready for the next `kitchen create.`

{% include image.html path="kitchen-test.png" path-detail="kitchen-test.png" alt="Kitchen Test" %}

I hope this solution will assist you testing your Windows Ansible roles.

Avishay.