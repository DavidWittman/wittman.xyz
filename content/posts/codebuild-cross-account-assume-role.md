---
title: "Assume a cross-account IAM role in AWS CodeBuild"
date: 2022-02-16T16:31:05-06:00
draft: false
tags: [aws]
---

First, make an IAM role which your CodeBuild Service Role can assume. This means that the Service Role is listed as a "trusted entity" in the trust policy for the particular IAM role which you are trying to assume. Here's an example trust policy:

```
$ aws iam get-role --role-name codebuild-assumed-role | jq '.Role.AssumeRolePolicyDocument'
{
  "Version": "2012-10-17",
  "Statement": [
    {
        "Effect": "Allow",
        "Principal": {
            "AWS": "arn:aws:iam::012345678901:role/CodeBuild-Service-Role"
        },
        "Action": "sts:AssumeRole",
        "Condition": {}
    }
  ]
}
```


Then in your CodeBuild job you can assume this role by doing the following:

``` yaml
version: 0.2

env:
  variables:
    ASSUME_ROLE_ARN: "arn:aws:iam::012345678901:role/codebuild-assumed-role"
  git-credential-helper: yes
phases:
  pre_build:
    commands:
      - aws configure set profile.assumerole.role_arn "$ASSUME_ROLE_ARN"
      - aws configure set profile.assumerole.credential_source EcsContainer
  build:
    commands:
      - AWS_PROFILE=assumerole aws sts get-caller-identity
```
