---
title : "Module 2 - Replicating Data Across Regions Scenario: Using Amazon FSx for Lustre & Amazon S3 with Amazon EKS"
weight : 200
---
-------------------------------------------------------------

## What is Amazon FSx for Lustre?

Amazon FSx for Lustre is a fully managed service that provides a high-performance file system optimized for workloads such as machine learning, analytics, high performance computing, electronic design automation and media processing. To support these challenging workloads, Amazon FSx for Lustre can serve as a link to Amazon S3 buckets, provides sub-millisecond access to your data and allows you to read and write data at speeds of up to hundreds of GBs per second and hundreds of thousands of IOPS.

To know more about the FSx for Lustre, refer to the [official document](https://docs.aws.amazon.com/fsx/latest/LustreGuide/what-is.html).

### Integrates with EKS and Containers

Amazon Elastic Kubernetes Service (EKS) released CSI drivers for FSx for Lustre, allowing easy and fast integration with containers. FSx for Lustre is perfectly suited for containerized applications and workloads requiring low latency and high throughput. Containers in a Pod need access to shared data that needs to be persistent beyond the life of the pod, and Amazon FSx for Lustre is one of the sub millisecond performance choice available.

## About the Setup

The infrastructure comprises of an Amazon EKS cluster with two EC2 worker nodes and Amazon FSx for Lustre file system that spans across multiple regions. On this infrastructure, we will walk through how to leverage [Amazon FSx for Lustre CSI Driver](https://github.com/kubernetes-sigs/aws-fsx-csi-driver) to create storage volume powered by Amazon FSx for for Lustre file system that runs on Amazon EKS cluster. Amazon FSx for for Lustre CSI driver provides a CSI interface that allows Amazon EKS clusters to manage the lifecycle of Amazon FSx for for Lustre file systems.The test environment could be created quite easily with a Infrastructure of Code (IaC) approach thanks to AWS CloudFormation’s capability and we will dive deep into how to deploy Amazon FSx for Lustre CSI into the Amazon EKS cluster via [Kustomize](https://kustomize.io/) or [Helm](https://helm.sh/), and creating the storage class, persistent volume claims so as to let the application pod mount on the volume provided by Amazon FSx for Lustre file system.

::children
