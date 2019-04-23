---
layout: post
title:  "AWS Glacier storage - A short overview"
author: PrincipalStream
comments: true
excerpt_separator: <!--more-->
categories: [AWS]
tags: [AWS, Glacier]
---

If your are thinking about an extermely low cost solution to store your archivel data then Amazon Glacier could be a right fit. 

One can store data in Amazon Glacier as archives. 
One can seamlessly move data between Amazon Glacier and Amazon S3 using S3 data lifecycle policies.

<!--more-->

If the data in the usecase is rapidly changing or client client needs immediate access then Amazon Glacier is not a good fit.

Amazon Glacier is designed to store data that is infrequently accessed and long lived.

The single archive limit is 40 TB, but there is no limit to the total amount of data that can store in the service. One can do multi part upload to improve the upload experience for larger archives.

It redundantly stores data in multiple facilities and on multiple devices within each facility. This service performs regular, systematic data integrity checks and is built to be automatically self-healing.

The account owner and other users specified by the owner in IAM policy can access the data in Amazon Glacier. This service use server side encryption using AES-256, but customers who wants to manage their own keys can encrypt their data before uploading it to Glacier.

Data compliance control could be achieved by setting Glacier vault lock policy and then lock the policy from future edits.

Amazon Glacier is integrated with AWS Cloudtrail which logs the details of any API calls to data access and then delivered to an Amazon S3 bucket that account owner specify.

There are two ways to use Amazon Glacier.
1. Amazon Glacier API.
2. Amazon S3 API.

Object lifecycle managment of S3 bucket provides automatic and policy driven archiving from S3 to Glacier.Owner can specify an absolute or relative time period after which the S3 object will be moved to Glacier. Retrieval puts a copy of the retrieved object in Amazon S3 Reduced Redundancy Storage (RSS) for a specified retention period. The original archived object will be there in Glacier.

Please note that the objects archived using Amazon S3 life cycle policies can only be retrieved by using the Amazon S3 API or Amazon S3 console.

There is no minimum fee for Amazon Glacier and it is based on pay for the usage. Upto 5 percent of average monthly storage can be retrieved for free each month. A prorated charge also applies for items deleted prior to 90 days' passage.

## Conclusion

Amazon Glacier is a storage system which is suitable for archived data storage. Amazon Glacier provides APIs to upload and retrived data and besides that it works along with Amazon S3 bucket as a storage class.

Note: *The above notes are prepared while preparing for AWS certification. Please refer Amazon Web Service for the most updated information on Amazon Glacier.*