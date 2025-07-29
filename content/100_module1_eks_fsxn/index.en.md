---
title : "Module 1 - Multi-AZ scenario: Using Amazon FSx for NetApp ONTAP with Amazon EKS"
weight : 100
---
-------------------------------------------------------------

## What is Amazon FSx for NetApp ONTAP?

Amazon FSx for NetApp ONTAP is a fully managed service that provides highly reliable, scalable, performant, and feature-rich file storage built on NetApp’s popular ONTAP file system. It provides the familiar features, performance, capabilities, multi-protocol (iSCIS/NFS/SMB), and APIs of NetApp file systems with the agility, scalability, and simplicity of a fully managed AWS service.

In terms of how Amazon FSx for NetApp ONTAP works, refer to the [official document](https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/how-it-works-fsx-ontap.html).

### Integrates with EKS and Containers

Amazon Elastic Kubernetes Service (EKS) released CSI driver for FSx for NetApp ONTAP, allowing easy and fast integration with containers. FSx for NetApp ONTAP is perfectly suited for containerized applications and workloads requiring low latency and high throughput. Containers in a Pod need access to shared data that needs to be persistent beyond the life of the pod, and Amazon FSx for NetApp ONTAP is the one of the high performance choice available.


## About the Setup

The infrastructure comprises of an Amazon EKS cluster with two EC2 worker nodes and Amazon FSx for NetApp ONTAP file system that spans across multiple availability zones. After infrastructure deployment, we will walk through how to leverage NetApp’s [Trident Container Storage Interface (CSI)](https://netapp-trident.readthedocs.io/en/stable-v19.01/kubernetes/trident-csi.html) to create storage volume powered by Amazon FSx for NetApp ONTAP for a MySql database that runs on Amazon EKS cluster. NetApp's Trident Container Storage Interface (CSI) driver provides a CSI interface that allows Amazon EKS clusters to manage the lifecycle of Amazon FSx for NetApp ONTAP file systems.The test environment could be created quite easily with a Infrastructure of Code (IaC) approach thanks to AWS CloudFormation’s capability and we will dive deep into how to deploy Trident CSI Operator into the Amazon EKS cluster via [Helm](https://helm.sh/), and creating the storage class, persistent volume claims so as to let the application pod mount on the volume provided by Amazon FSx for NetApp ONTAP.

::children