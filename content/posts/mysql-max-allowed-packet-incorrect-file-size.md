---
title: "MySQL's `max_allowed_packet` and BLOB columns"
date: 2025-10-05T09:47:47-05:00
description: Why your MySQL BLOBs might hit `max_allowed_packet` limitations sooner than you expect -- and how prepared statements solve it.
draft: false
tags: [mysql, nodejs, javascript]
---

I work on a legacy system that still uses MySQL as its primary datastore. A few weeks ago, we were running into an issue where we were hitting a `max_allowed_packet` error on inserts that seemed well within the configured 16 MiB limit. Here's the log message:

``` json
{
  "msg": "Failed to update contacts backup in database",
  "errMsg": "Got a packet bigger than 'max_allowed_packet' bytes",
  "contactBlobSizeInBytes": 9459281
}
```

Based on this output, the blob size is 9,459,281 bytes -- nearly 9.5MiB. Not even close to the limit of 16MiB. There's going to be other data and more overhead in the packet which is sent to MySQL, but there's no way it's enough to reach the limit of 16MiB.

### What is the `max_allowed_packet` setting?

`max_allowed_packet` in MySQL is a system variable that defines the maximum size of a communication packet. This variable sets an upper limit on the size of any single message exchanged between the MySQL server and its clients.

### BLOB columns

The legacy application in question allows users to upload encrypted blobs containing contact backups. These BLOB columns contain binary data which can grow quite large. When inserting these blobs into the database, we log their size for informational purposes, and we could see that the size of the blob was less than half of the size we had set for the `max_allowed_packet` setting in MySQL.

To double-check, I queried the table and verified that the size of the BLOB columns in MySQL matched what we were logging:

``` sql
SELECT LENGTH(`contacts_backup`) FROM `user` WHERE id = 123
```

### Inserting binary data

While I was querying the tables to confirm the size of the blobs in the database, I noticed that the data in the columns was printed in hexadecimal, and then it dawned on me: if the binary data is serialized to hexadecimal as a part of the insert statement, it will effectively double in size. This is because a value which is one binary byte becomes two bytes when stringified and serialized as hex.

So my hypothesis became that if the encrypted blob is being hexified when creating the MySQL packet, it would effectively double in size and hit the `max_allowed_packet` limit well before we had planned. With that in mind, I went back to the code and looked at how we’re inserting the data.

``` js
const result = await db.query('UPDATE user SET contacts_backup = ? WHERE id = ?', [contactsBlob, uid])
```

Aha! This is using the `.query` method in NodeJS’s MySQL library. This library offers two methods for interacting with the database: `.query` and `.execute`. I usually have to Google the difference between the two methods, but after this incident I don’t think I will be forgetting anytime soon.

### NodeJS MySQL `.query` vs `.execute`

If, like me, you were to look up the difference between these two methods, you would probably end up at [this stackoverflow answer](https://stackoverflow.com/a/53219297), where it describes the difference as:

> With `.query()`, parameter substitution is handled on the client
>
> With `.execute()` prepared statement parameters are sent from the client as a serialized string and handled by the server. Since let data = req.body is an object, that's not going to work.

Okay. I think that makes sense. So how does that fit in here? The big difference in our case is that `.execute` uses prepared statements when interacting with the database. Prepared statements end up serializing the data in the execution phase of the query, and the difference is that they are serialized to match the format of the column. As a result, the data in the parameters of the query _is sent in binary format_. This is exactly what we want! The encrypted blobs will no longer be hexified before being sent to the MySQL server, and should fit well within the `max_allowed_packet` limit which we had previously defined.

Let's try it out. Here’s the updated code:

``` js
// Use db.execute to run as a prepared statement and serialize the backup as binary
const result = await db.execute('UPDATE user SET contacts_backup = ? WHERE id = ?', [contactsBlob, uid])
```

This change resolved the issue immediately. We could now insert encrypted BLOBs up to the expected size without encountering any `max_allowed_packet` errors.

## Conclusion

It’s 2025 so these sorts of MySQL issues are (hopefully) becoming more and more esoteric, but it’s still fun to learn new things about legacy systems. This was a great reminder that how you insert data into your database can matter just as much as what data you insert.

So if you’re inserting large binary blobs into your database, use prepared statements. You’ll avoid prematurely hitting limits like `max_allowed_packet` and improve performance all at the same time.