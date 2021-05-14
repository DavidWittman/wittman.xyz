---
title: "The Packer File Provisioner for Windows is Miserably Slow"
date: 2021-05-14T10:56:57-05:00
draft: false
tags: [packer, windows, aws, s3]
---

I was building a Windows AMI with Packer which was taking for-ev-er because part of the provisioning process
uploaded an MSI installer using the [file provisioner](https://www.packer.io/docs/provisioners/file). Uploading
a 15MiB file was taking upwards of an hour, and feedback loops for these AMI builds are already long so this made
the wait time pretty much unbearable.

It turns out this is caused by Packer uploading via WinRM, something which is called out specifically in the
[documentation](https://www.packer.io/docs/provisioners/file#slowness-when-transferring-large-files-over-winrm):

> Because of the way our WinRM transfers works, it can take a very long time to upload and download even moderately sized files. If you're experiencing slowness using the file provisioner on Windows, it's suggested that you set up an SSH server and use the ssh communicator. If you only want to transfer files to your guest, and if your builder supports it, you may also use the http_directory or http_content directives. This will cause that directory to be available to the guest over http, and set the environment variable PACKER_HTTP_ADDR to the address.

I wasn't really interested in setting up an SSH server or doing the HTTP thing, so I configured my build to pull
the files from Amazon S3 instead.

## Copying files from S3 during Windows Packer builds

First and foremost, copy the files you need during the build to your S3 bucket. We're going to pretend our bucket
is named `my-bucket` for these examples.

```
aws s3 sync ./files/ s3://my-bucket/
```

Now, modify your Packer template to create an IAM Instance Role so that the temporary EC2 instance has access to
the S3 bucket. The easiest way to do this is by using the [temporary_iam_instance_profile_policy_document](https://www.packer.io/docs/builders/amazon/ebs#temporary_iam_instance_profile_policy_document) parameter:

``` json
{
  "builders": [
    {
      "type": "amazon-ebs",
      "temporary_iam_instance_profile_policy_document": {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Action": [
              "s3:GetObject",
              "s3:ListBucket"
            ],
            "Effect": "Allow",
            "Resource": [
              "arn:aws:s3:::my-bucket",
              "arn:aws:s3:::my-bucket/*"
            ]
          }
        ]
      },
...
```

Finally, add a provisioner to pull down the files from Amazon S3 during your AMI build. This step installs
the AWS CLI via [chocolatey](https://chocolatey.org/) and then runs `aws s3 sync`:

``` json
{
  "provisioners": [
    {
      "type": "powershell",
      "inline": [
        "Set-ExecutionPolicy Bypass -Scope Process -Force",
        "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))",
        "choco install -y awscli",
        "aws s3 sync s3://my-bucket/ C:\\Temp\\"
      ]
    }
  ],
...
```

Now all the files in your S3 bucket will be available at `C:\Temp\` on the Packer Build instance, in a small
fraction of the time which it would take to complete when using WinRM.
