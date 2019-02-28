---
layout: post
title:  "AWS S3 storage - A short overview"
author: AK
comments: true
excerpt_separator: <!--more-->
categories: [AWS]
tags: [AWS, S3]
---

Amazon S3, simple storage service , is an object storage. It provides a simple web interface to store and retrieve any amount of data from anywhere on the web.

<!--more-->

1. Amazon S3 Standard - Used for general purpose frequently accessed data.
2. Amazon S3 Standard IA - Used for long lived but infrequently accessed objects.

Amazon S3 is used to store and distribute static web content and media
Each object in Amazon S3 has a unique HTTP URL. 
Amazon S3 can serve as an origin store for a content delivery network(CDN), such as CloudFront.

Amazon S3 can be used to host entire static websites.
Amazon S3 is used as a data store for computation and large scale analytics. Because of the horizontal scalability of Amazon S3, the can be access from multiple computing nodes concurrently without being constrained by a single connection.

Amazon S3 is often used as a highly durable, scalable and secure solution for backup and archiving of critical data. 
Amazon S3 cross-region replication can be used to automatically copy objects across S3 buckets in different AWS Regions asynchronously and this could be a soultion for business continuity.

Amazon S3 offers a multipart upload command to upload a single object as a set of parts. Amazon 3 assembles these parts and create the object. This gives the benefit of uploading smaller parts in parallel and restart the upload of smaller parts instead of restarting the upload of the entire large object.

Amazon S3 Transfer Acceleration (need to be enabled in S3 bucket) enables in the S3 bucket fast, easy and secure transfer of files over long distances. Once the acceleration is enabled a new end point (<bucketname>.s3-accelerate.amazonaws.com) is also available apart from the regular endpoint.

Amazon S3 is designed to sustain the concurrent loss of data in two facilities, making it well suited to serve as the primary data storage for mission-critical data. In addition there is option to enable cross region replication on each S3 bucket.

Amazon S3 supports virtually unlimited number of files in any bucket and an bucket can store a virtually unlimited number of bytes.

Writing an access policy allows granting other AWS accounts and users permission to perform resource operations.

In S3 , data at rest can be protected by server side encryption. Client side encryption is also possible but in this case client has to encrypt the data before posting it to S3.

Versiong can be used to preserve, retrieve, and restore every version of every object stored in the Amazon S3 bucket. In addition , by enabling MFA Delete for a bucket, two forms of authentication are required to change the versioning state of the bucket or to permanently delete an object version.

The requests to S3 can be tracked by enabling access logging.

Amazon S3 provides standards-based REST web service APIs for both management and data operations.

Using the AWS CLI for Amazon S3, you can perform recursive uploads and downloads using single folder-level Amazon s3 command and also perform parallel transfers.

Amazon S3 notification feature allows to receive notifications (to SNS, SQS or to Lambda) when certain events happen in your bucket.

With Amazon S3, you pay only for the storage you actually use. There are Data Transfer IN and OUT fees if you enable Amazon S3 Acceleration on the bucket but Amazon determine the charging based on the speed of data transfer.

## Conclusion

Amazon S3 low cost storage solution for the data which are frequently accessed but not frequently changed.

Note: *The above notes are prepared while preparing for AWS certification. Please refer Amazon Web Service for the most updated information on Amazon S3.*