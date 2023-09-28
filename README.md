# Repo Includes - How to run a : 
- ### Multi-AZ Stateful application on EKS with AWS FSx for NetApp ONTAP with Amazon EKS
- ### Replicating Data Across Regions Scenario: Using Amazon FSx for Lustre & Amazon S3 with Amazon EKS
- ### Test OpenZFS with Amazon EKS
<!-- - ### Simulate Oracle DB performance on OpenZFS, test use-cases for compression, backup optimisation, high-parallel throughput -->


## About the Setup
The infrastructure comprises of an Amazon EKS cluster with three EC2 worker nodes and a FSxONTAP file system that spans across multiple availability zones. After infrastructure deployment, we will walk through how to leverage NetApp’s Trident Container Storage Interface (CSI) (https://netapp-trident.readthedocs.io/en/stable-v19.01/kubernetes/trident-csi.html)to create storage volume powered by FSxONTAP for a MySql database that runs on Amazon EKS cluster. NetApp's Trident Container Storage Interface (CSI) driver provides a CSI interface that allows Amazon EKS clusters to manage the lifecycle of Amazon FSx for NetApp ONTAP file systems.The test environment could be created quite easily with a Infrastructure of Code (IaC) approach thanks to AWS CloudFormation’s capability and we will dive deep into how to deploy Trident CSI Operator into the Amazon EKS cluster via Helm (https://helm.sh/), and creating the storage class, persistent volume claims so as to let the application pod mount on the volume provided by FSxONTAP.

## Architecture Diagram

![Diagram](/Architecture.png)

## Project Structure

```
.
├── Architecture.png
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── FSxCFN
│   ├── FSxONTAP.yaml
│   ├── S3Buckets.yaml
│   ├── S3ReplicationIAM.yaml
│   ├── WorkshopStudioCFN.yaml
│   ├── fsx-csi-driver.json
│   ├── fsxL-SecurityGroup.yaml
│   ├── fsxZ-SecurityGroup.yaml
│   └── vpc-subnets.yaml
├── LICENSE
├── README.md
├── docs
│   ├── 5.deploy-trident-operator.md
│   ├── 6.provision-trident-volume-and-storage-class.md
│   ├── 7.create-clones-and-snapshots.md
│   └── images
│       ├── SVM_01.png
│       ├── admin.png
│       ├── file.png
│       ├── file1.png
│       ├── file2.png
│       ├── filebrowser.png
│       ├── sample.png
│       ├── sample1.png
│       ├── sample2.png
│       ├── secrets_manager_01.png
│       ├── volume01.png
│       ├── volume02.png
│       ├── volume03.png
│       └── volume04.png
└── eks
    ├── FSxL
    │   ├── claim.yaml
    │   ├── fsxL-storage-class.yaml
    │   ├── pod.yaml
    │   └── pod_performance.yaml
    ├── FSxN
    │   ├── backend-ontap-nas.yaml
    │   ├── mysql
    │   │   ├── mysql-client.yaml
    │   │   ├── mysql-configmap.yaml
    │   │   ├── mysql-service.yaml
    │   │   ├── mysql-statefulset.yaml
    │   │   ├── pod_backup_restore.yaml
    │   │   ├── pvc-from-snap.yaml
    │   │   ├── snapshot-class.yaml
    │   │   └── snapshot.yaml
    │   ├── pod_performance_different_AZ.yaml
    │   ├── pod_performance_same_AZ.yaml
    │   ├── pvc-trident.yaml
    │   ├── storage-class-csi-nas.yaml
    │   └── svm_secret.yaml
    ├── FSxZ
    │   ├── filesystem
    │   │   ├── delete_me_sc.yaml
    │   │   ├── fsxZ-storage-claim.yaml
    │   │   ├── fsxZ-storage-class.yaml
    │   │   └── pod-fsxZ-filesystem.yaml
    │   └── volume
    │       ├── delete_me_sc.yaml
    │       ├── fsxZ-volume-claim.yaml
    │       ├── fsxZ-volume-storage-class.yaml
    │       └── pod-fsxZ-vol.yaml
    └── cluster.yaml
```

## Prerequisites

* An AWS account with necessary permissions to create and manage Amazon VPC, Amazon EKS Cluster, Amazon FSx for NetApp ONTAP file system and Cloudformation stack. 
* eksctl, kubectl and Helm3 have been installed in your laptop. 
    * eksctl: https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html 
    * kubectl: https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html
    * Helm 3: https://docs.aws.amazon.com/eks/latest/userguide/helm.html
* The AWS Command Line Interface (AWS CLI) should have been configured in your working environment. For information about installing and configuring the AWS CLI, see Installing, updating, and uninstalling the AWS CLI version 2 (https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).
* A good understanding of Amazon EKS, Kubernetes CSI, and Amazon FSx for NetApp ONTAP
* The Trident Operator is supported with Kubernetes 1.17 or later . We will provide instructions on installing Trident using Helm in the following steps. 

# Getting started

## Launch VPC Stack
As you clone the referenced GitHub repo, change directory as below:
```
cd FSxCFN
```

Launch the CloudFormation stack to set up the network environment for both FSxONTAP and EKS cluster:
```
aws cloudformation create-stack --stack-name EKS-FSXONTAP-VPC --template-body file://./vpc-subnets.yaml --region <region-name>
```

## Create an Amazon FSx for NetApp ONTAP file system
Launch the Cloudformation stack as below to spin up an Amazon FSx for NetApp ONTAP file system. And you need to fill in the parameters and configuration accordingly. Make you choose those two private subnets and VPC as created in the previous step. 

```
aws cloudformation create-stack \
  --stack-name EKS-FSXONTAP \
  --template-body file://./FSxONTAP.yaml \
  --region <region-name> \
  --parameters \
  ParameterKey=Subnet1ID,ParameterValue=[your_preferred_subnet1] \
  ParameterKey=Subnet2ID,ParameterValue=[your_preferred_subnet2] \
  ParameterKey=myVpc,ParameterValue=[your_VPC] \
  ParameterKey=FSxONTAPRouteTable,ParameterValue=[your_routetable] \
  ParameterKey=FileSystemName,ParameterValue=EKS-myFSxONTAP \
  ParameterKey=ThroughputCapacity,ParameterValue=512 \
  ParameterKey=FSxAllowedCIDR,ParameterValue=[your_allowed_CIDR] \
  ParameterKey=FsxAdminPassword,ParameterValue=[Define password] \
  ParameterKey=SvmAdminPassword,ParameterValue=[Define password] \
  --capabilities CAPABILITY_NAMED_IAM    
```

## Create an Amazon EKS cluster
Change to the directory where cluster.yaml file exists:
```
eksctl create cluster -f ./cluster.yaml
```

## Clean up
```
eksctl delete cluster --name=FSx-eks-cluster --region <region-name>
```

```
aws cloudformation delete-stack --stack-name EKS-FSxONTAP --region <region-name>
```

```
aws cloudformation delete-stack --stack-name EKS-MyFSxONTAP --region <region-name>
```

# Contributing 
See [CONTRIBUTING](https://github.com/aws-samples/eks-fsx-workshop/blob/main/CONTRIBUTING.md) for more information.

# License
This library is licensed under the MIT-0 License. See the LICENSE file.
