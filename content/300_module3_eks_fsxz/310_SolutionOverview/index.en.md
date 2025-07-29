---
title : "Solution overview"
weight : 310
---
-------------------------------------------------------------



## Solution Overview

In this module you will deploy Amazon FSx for OpenZFS CSI driver for the existing 2 node Amazon EKS cluster provisioned for this workshop earlier. Amazon FSx for OpenZFS will be auto-provisioned using CSI driver when you configure storage class and persistent volume claim (PVC). On test pods we will simulate performance test for OpenZFS, test usecases for compression, backup optimisation, high-parallel throughput. The integration helps you to build business continuity and DR for the modern apps leveraging the persitent storage layer of FSx for OpenZFS file system. Run the performance test on the Amazon FSx for OpenZFS file system and capture the metrics. 

## High-level architecture diagram
![FSxZ-Architecture](/static/images/FSxZ-Architecture.png)



