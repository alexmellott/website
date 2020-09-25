---
title: "Creating Automated Images on Ec2"
date: 2020-09-25T17:03:14-04:00
draft: false
---

Getting started on AWS often requires developers to spend hours creating EC2 instances for specific use cases (i.e. LAMP stacks, MySQL servers, etc.). What I discovered is that I was spending too much time manually spinning up servers and configuring them.

Fortunately, I came across tools made by HashiCorp that enabled me to create custom deployments of standardized EC2 machine images based on pre-set standards. 

The full repo is available on [GitHub](https://github.com/alexmellott/aws-ami-builder) for your review, but let me explain a little bit about how it works.

## How It Works

This repo utilizes a Docker image containing the HashiCorp Packer tool. Packer is cool because it can automate the creation of any type of machine image. Not only can you create custom images for AWS, but you can also create them for virtually any cloud platform, on-prem environment, or even customized containers. It's entirely open-source and is available at [https://www.packer.io/](https://www.packer.io/).

I set up my repo to automatically trigger a GitLab CI/CD pipeline when a push happens. This commit is initially triggered in our `sandbox` branch so we can test it on our AWS sandbox accounts. The pipeline will first lint our `base_ami.json` file to validate our configuration parameters are correct. Then, it will spin up a builder instance in our sandbox EC2 account for testing. After we launch our AMI in the sandbox account and perform QA/QC testing, we then create a merge request from `sandbox` to `master`. The pipeline runs again and creates the master image. What's nice about this is it allows us to keep change history consistent so that the sandbox/master images are always the same. Additionally, we use Packer to copy our newly-created AMIs to each AWS region we use. This saves a ton of time and hassle!

For the initial repo in my company's private GitLab, I created a custom Docker image that builds Packer in compliance with our internal tooling standards. For developers getting started, there's a great [image on Docker Hub](https://hub.docker.com/r/hashicorp/packer/) you can use to get started.

You can easily customize this framework using your own custom scripts. For the shell, I've included a blank `configure.sh` file that you can use to get started. However, you can also use Python, PowerShell, Windows Shell, or even provisioning tools like Ansible, Chef, Puppet, and Salt to provision and set up images.

## Requirements

### Git Branches

This repo is designed for use with two branches: `sandbox` and `master`.

GitLab CI/CD must contain the following variables:

- PRD_AWS_ACCESS_KEY
- PRD_AWS_REGION
- PRD_AWS_SECRET_KEY
- PRD_BUILD_SUBNET_ID
- PRD_BUILD_VPC_ID
- SBX_AWS_ACCESS_KEY
- SBX_AWS_REGION
- SBX_AWS_SECRET_KEY
- SBX_BUILD_SUBNET_ID
- SBX_BUILD_VPC_ID

(Note: if using IAM roles, you do not need to specify an access key or secret key. However, you may need to revise the code to properly query the EC2 Metadata API.)

### AWS IAM Role Permissions

Your attached policy should contain the following permissions for Packer to work

```json
{
  "Version": "2012-10-17",
  "Statement": [{
      "Effect": "Allow",
      "Action" : [
        "ec2:AttachVolume",
        "ec2:AuthorizeSecurityGroupIngress",
        "ec2:CopyImage",
        "ec2:CreateImage",
        "ec2:CreateKeypair",
        "ec2:CreateSecurityGroup",
        "ec2:CreateSnapshot",
        "ec2:CreateTags",
        "ec2:CreateVolume",
        "ec2:DeleteKeyPair",
        "ec2:DeleteSecurityGroup",
        "ec2:DeleteSnapshot",
        "ec2:DeleteVolume",
        "ec2:DeregisterImage",
        "ec2:DescribeImageAttribute",
        "ec2:DescribeImages",
        "ec2:DescribeInstances",
        "ec2:DescribeInstanceStatus",
        "ec2:DescribeRegions",
        "ec2:DescribeSecurityGroups",
        "ec2:DescribeSnapshots",
        "ec2:DescribeSubnets",
        "ec2:DescribeTags",
        "ec2:DescribeVolumes",
        "ec2:DetachVolume",
        "ec2:GetPasswordData",
        "ec2:ModifyImageAttribute",
        "ec2:ModifyInstanceAttribute",
        "ec2:ModifySnapshotAttribute",
        "ec2:RegisterImage",
        "ec2:RunInstances",
        "ec2:StopInstances",
        "ec2:TerminateInstances"
      ],
      "Resource" : "*"
  }]
}
```

