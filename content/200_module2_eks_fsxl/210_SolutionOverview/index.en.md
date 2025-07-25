---
title : "Solution overview"
weight : 210
---
-------------------------------------------------------------

## Solution Overview

In this module you will deploy Amazon FSx for Lustre CSI driver for the existing 2 node Amazon EKS cluster provisioned for this workshop earlier. Amazon FSx for Lustre will be auto-provisioned using CSI driver when you configure storage class and persistent volume claim (PVC). Run the performance test on the Amazon FSx for Lustre file system and capture the metrics. You will setup a cross region replication (CRR) of the buckets between primary lab region and us-east-2. List the data replicated from the EKS cluster running in us-east-2. This module to help you understand how easy to auto-provision the Amazon FSx for Lustre and use the Amazon S3 native replication to move the data across region. The integration helps you to build business continuity and DR for the modern apps leveraging the persitent storage layer of FSx for Lustre file system. 

![FSx-Architecture](/static/images/FSxL-Architecture.png)
