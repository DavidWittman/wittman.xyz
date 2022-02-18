---
title: "Assume a cross-account IAM role in AWS CodeBuild"
date: 2022-02-16T16:31:05-06:00
draft: false
tags: [aws]
---

Sometimes in a CodeBuild run, you need to use IAM authentication to access resources in another account. In my case, I needed to clone a CodeCommit repository in order to package up some Ansible playbooks for a CodeDeploy run, but there are a variety reasons why you might want to do this. The process wasn't very well defined in the documentation so I figured I'd write it down here so I can reference it later.

First, make an IAM role which your CodeBuild Service Role can assume. This means that the Service Role is listed as a "trusted entity" in the trust policy for the particular IAM role which you are trying to assume. In this example, we're going to be assuming the role `codebuild-assumed-role` from another account. Here's the trust policy we'll use for this role:

``` json
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

This just says that the CodeBuild-Service-Role which I created on the _other_ account (i.e. the role which CodeBuild runs as) is "trusted" to assume this particular role. You can add it to your role like so:

```
$ aws iam update-assume-role-policy --role-name codebuild-assumed-role --policy-document file://./policy.json
```

Then in the CodeBuild job you can assume this role by doing the following:

``` yaml
version: 0.2

env:
  variables:
    ASSUME_ROLE_ARN: "arn:aws:iam::012345678901:role/codebuild-assumed-role"
phases:
  pre_build:
    commands:
      - aws configure set profile.assumerole.role_arn "$ASSUME_ROLE_ARN"
      - aws configure set profile.assumerole.credential_source EcsContainer
  build:
    commands:
      - AWS_PROFILE=assumerole aws sts get-caller-identity
```

Obviously you want to do more than just print out the caller identity, but this is just a silly example.
