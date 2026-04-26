---
title: "Why Your S3 Lifecycle Policies Arent Actually Deleting Anything"
date: 2026-04-26T13:28:38-05:00
description: >
  This one misconfiguration was costing my team almost 60% more in S3 storage costs. Here’s what it was.
tags: [aws, s3]
---

Expiring old objects from S3 buckets is easy: just configure a Lifecycle Policy to remove objects after a certain time period and they will automatically be removed for you. However, if you’re using Lifecycle Policies on versioned S3 buckets, you may be forgetting one step which will prevent these objects from actually being deleted.

Versioned S3 buckets are a feature of S3 which allow you to keep multiple versions of a single S3 object. It is used to provide a recovery mechanism in the event of accidental changes or deletions of objects in the bucket. When an object is deleted, a delete marker is used as the current version of the object and the original object becomes an old version.

Maybe you can already see the problem here. It turns out the Lifecycle Policies follow the same rules which normal API users do when it comes to versioned S3 buckets. Objects are not truly deleted when the Lifecycle Policy transitions them to a deleted state. Instead, the object is hidden from view with a delete marker and the content of the original object is stored as a previous version. When this happens, it looks like the object is deleted but you will still be billed for storage of the older version containing the full object.

The correct way to configure Lifecycle Policies in versioned S3 buckets is to configure them to also expire non-current objects versions as well. Here’s an example CDK configuration, but the same principle applies to Terraform, Cloudformation, or the AWS Console:

``` js
const bucket = new s3.Bucket(this, 'MyBucket', {
  versioned: true,
  lifecycleRules: [
    {
      noncurrentVersionExpiration: Duration.days(30),
    },
  ],
});
```

This misconfiguration is a surprisingly common one… multiple teams at my company independently discovered that their buckets weren’t correctly expiring old objects because of versioning. If you’re using versioned S3 buckets, it’s worth auditing your Lifecycle Policy configurations now. Check whether `noncurrentVersionExpiration` is set, and if it isn’t, don’t be surprised if you see a significant drop in your storage bill once you add it. In my case, it was nearly 60% for our largest bucket.

