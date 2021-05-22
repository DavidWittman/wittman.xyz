---
title: "Deleting an AMI and its volumes with aws-cli"
date: 2021-05-13T21:04:32-05:00
description: Deleting AMIs completely should not be this difficult.
draft: false
tags: [aws, bash]
---

It is annoyingly tedious to delete an AMI *and* its associated snapshot volumes via the EC2 Console. I think it takes no fewer than 4 clicks and requires some copying and pasting in order to get the right snapshot volume. Here's a Bash function which I use to delete an AMI and its volumes in one fell swoop using the AWS CLI.

``` bash
delete_ami() {
    SNAPSHOTS=$(aws ec2 describe-images --image-ids "$1" --query "Images[*].BlockDeviceMappings[*].Ebs.SnapshotId" --output text)
    aws ec2 deregister-image --image-id "$1"
    for SNAP in $SNAPSHOTS; do
        aws ec2 delete-snapshot --snapshot-id "$SNAP"
    done
}
```

You can call it by passing the AMI ID as the first argument:

``` bash
$ delete_ami ami-0faf6beeff3d4cafe
```

Be gone, wasteful AMI!
