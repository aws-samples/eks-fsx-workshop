---
title : "Module 3 - Using OpenZFS with Amazon EKS"
weight : 300
---
-------------------------------------------------------------

## What is Amazon FSx for NetApp ONTAP?

Amazon FSx for OpenZFS is a fully managed file storage service that makes it easy to move data residing in on-premises ZFS or other Linux-based file servers to AWS without changing your application code or how you manage data. It offers highly reliable, scalable, performant, and feature-rich file storage built on the open-source OpenZFS file system, providing the familiar features and capabilities of OpenZFS file systems with the agility, scalability, and simplicity of a fully managed AWS service. For developers building cloud-native applications, it offers simple, high-performance storage with rich capabilities for working with data.

In terms of how Amazon FSx OpenZFS works, refer to the [official document](https://docs.aws.amazon.com/fsx/latest/OpenZFSGuide/what-is-fsx.html).

### Integrates with EKS and Containers

Amazon Elastic Kubernetes Service (EKS) released CSI drivers for FSx for OpenZFS, allowing easy and fast integration with containers. FSx for OpenZFS is perfectly suited for containerized applications and workloads requiring low latency and high throughput. Containers in a Pod need access to shared data that needs to be persistent beyond the life of the pod, and Amazon FSx for OpenZFS is the highest performance choice available.

## About the Setup

The infrastructure comprises of an Amazon EKS cluster with two EC2 worker nodes and Amazon FSx OpenZFS file system that spans across multiple availability zones. On this infrastructure, we will walk through how to leverage [Amazon FSx for OpenZFS CSI Driver](https://github.com/kubernetes-sigs/aws-fsx-openzfs-csi-driver) to create storage volume powered by Amazon FSx for OpenZFS that runs on Amazon EKS cluster. Amazon FSx for OpenZFS CSI driver provides a CSI interface that allows Amazon EKS clusters to manage the lifecycle of Amazon FSx for OpenZFS file systems.The test environment could be created quite easily with a Infrastructure of Code (IaC) approach thanks to AWS CloudFormation’s capability and we will dive deep into how to deploy Amazon FSx for OpenZFS CSI into the Amazon EKS cluster via [Helm](https://helm.sh/), and creating the storage class, persistent volume claims so as to let the application pod mount on the volume provided by Amazon FSx for OpenZFS.



::children
