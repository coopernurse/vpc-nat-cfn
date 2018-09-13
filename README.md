
# VPC CloudFormation Template

## Overview

This repo contains a CloudFormation template that creates the following resources:

* VPC
* 1 public subnet attached to an internet gateway
* bastion / nat host in the public subnet
    * should use an [Amazon NAT instance AMI](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html) here
* 3 private subnets in separate AZs
    * all share a route table that routes outbound traffic via the bastion / nat instance

## Creating and updating stack

Edit `params.json` as desired, then:

```bash
# create
aws cloudformation create-stack --stack-name mystack --template-body file://vpc-nat-cfn.yml --parameters file://params.json

# update
aws cloudformation update-stack --stack-name mystack --template-body file://vpc-nat-cfn.yml --parameters file://params.json
```

## Design

```
 +-------------------------------------------------------------------------------+
 |                                                                           VPC |
 |                                                                 172.20.0.0/16 |
 |  +--------------------+                                                       |
 |  | PublicSubnetA      |                                                       |
 |  |                    |                                                       |
 |  | +-------------+    |                                                       |
 |  | | bastion|nat |    |                                                       |
 |  | +-------------+    <------------------------------------------+            |
 |  |                    |                                          |            |
 |  | 172.20.1.0/24      <--------------+                           |            |
 |  +--------------------+              |                           |            |
 |            ^                         |                           |            |
 |            |                         |                           |            |
 |  +--------------------+    +--------------------+    +---------------------+  |
 |  | PrivateSubnetA     |    | PrivateSubnetB     |    | PrivateSubnetC      |  |
 |  |                    |    |                    |    |                     |  |
 |  |                    |    |                    |    |                     |  |
 |  |                    |    |                    |    |                     |  |
 |  | 172.20.16.0/20     |    | 172.20.32.0/20     |    | 172.20.48.0/20      |  |
 |  +--------------------+    +--------------------+    +---------------------+  |
 |                                                                               |
 |                                                                               |
 +-------------------------------------------------------------------------------+
```
