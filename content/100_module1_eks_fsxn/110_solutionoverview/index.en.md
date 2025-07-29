---
title: 'Solution Overview'
weight: 110
---
-------------------------------------------------------------

## Solution Overview

The infrastructure for this solution comprises of an Amazon EKS cluster with two EC2 worker nodes and a FSx for NetApp ONTAP file system that spans across Multiple Availability Zones. The two worker nodes and the FSx for NetApp ONTAP filesystem is deployed in the private subnets in the VPC. We will walk you through the steps on how to use NetApp’s Trident Container Storage Interface (CSI driver) to create storage volumes powered by FSx for NetApp ONTAP for a MySQL database running on an Amazon EKS cluster.

The following high-level architecture diagram illustrates the environment:

## High-level architecture diagram

![Diagram](/static/images/FSxN-Architecture.png)
