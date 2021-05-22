---
title: "Launching a Cross-Account CMK Encrypted AMI in an Auto-Scaling Group"
date: 2021-05-12T18:22:07-05:00
description: >
  How to create the appropriate KMS grants to launch an encrypted AMI on another AWS account in an Auto-Scaling Group.
  It's really easy to mess up if you're just skimming documentation like I did.
tags: [aws]
---

If you have shared an encrypted AMI with another AWS account and instances in an Auto-Scaling Group using that AMI fail
to start, you might see the following State Transition Message in the EC2 console:

> Client.InternalError: Client error on launch

In this case, it's pretty likely that you're running into issues decrypting the AMI using the KMS key on the other account. It's even possible that *your* IAM user is able to decrypt the AMI using the KMS key just fine but the [IAM Service Linked Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) which the ASG service uses cannot. You can test this theory by trying to launch an instance using the same encrypted-and-shared AMI outside of an Auto-Scaling group.

In order to fix this, you'll need to make two changes:

 1. Modify the key policy on the account with the KMS key and AMI so that the other account can use the key and create grants for that key
 2. Add a KMS Grant on the other account (the account with the ASG) to allow the AutoScaling Service Role to use the KMS key

## On the first account

This is the account where the KMS key and AMI reside. Edit the KMS Key Policy or add a grant to allow access from the
other account (ID 222222222222):

``` json
{
   "Sid": "Allow external account 222222222222 use of the CMK",
   "Effect": "Allow",
   "Principal": {
       "AWS": [
           "arn:aws:iam::222222222222:root"
       ]
   },
   "Action": [
       "kms:Decrypt",
       "kms:GenerateDataKeyWithoutPlaintext"
   ],
   "Resource": "*"
},
{
   "Sid": "Allow attachment of persistent resources in external account 222222222222",
   "Effect": "Allow",
   "Principal": {
       "AWS": [
           "arn:aws:iam::222222222222:root"
       ]
   },
   "Action": [
       "kms:CreateGrant"
   ],
   "Resource": "*"
}
```

## On the second account

This is the account where we will be launching the encrypted AMI in an ASG.

``` bash
# 111111111111 = Account with KMS key and AMI
# 222222222222 = Account which is launching instances via ASG
aws kms create-grant \
    --key-id arn:aws:kms:us-east-1:111111111111:key/dead98b3-c864-beef-cafe-0ee3d036f407 \
    --grantee-principal arn:aws:iam::222222222222:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling \
    --operations Decrypt GenerateDataKeyWithoutPlaintext ReEncryptFrom ReEncryptTo CreateGrant
```

That's it! Now your instances in the Auto-Scaling Group should launch successfully.

Source: https://aws.amazon.com/premiumsupport/knowledge-center/kms-launch-ec2-instance/
