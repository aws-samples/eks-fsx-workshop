# Repo Includes - How to run a : 
- ### Multi-AZ Stateful application on EKS with AWS FSx for NetApp ONTAP with Amazon EKS
- ### Replicating Data Across Regions Scenario: Using Amazon FSx for Lustre & Amazon S3 with Amazon EKS
- ### Multi-AZ Stateful application on EKS with OpenZFS with Amazon EKS
<!-- - ### Simulate Oracle DB performance on OpenZFS, test use-cases for compression, backup optimisation, high-parallel throughput -->


## Architecture Diagram for AWS FSx for NetApp ONTAP with Amazon EKS

**About the Setup**
The infrastructure for this solution comprises of an Amazon EKS cluster with two EC2 worker nodes and a FSx for NetApp ONTAP file system that spans across Multiple Availability Zones. The two worker nodes and the FSx for NetApp ONTAP filesystem is deployed in the private subnets in the VPC. We will walk you through the steps on how to use NetApp’s Trident Container Storage Interface (CSI driver) to create storage volumes powered by FSx for NetApp ONTAP for a MySQL database running on an Amazon EKS cluster.

![Diagram](/images/FSxN-Architecture.png)


## Architecture Diagram for Amazon FSx for Lustre & Amazon S3 with Amazon EKS

**About the Setup**
Amazon FSx for Lustre is a fully managed service that provides a high-performance file system optimized for workloads such as machine learning, analytics, high performance computing, electronic design automation and media processing. To support these challenging workloads, Amazon FSx for Lustre can serve as a link to Amazon S3 buckets, provides sub-millisecond access to your data and allows you to read and write data at speeds of up to hundreds of GBs per second and hundreds of thousands of IOPS. Here we are going to explore cross region desaster recovery using Lustre. 


![Diagram](/images/FSxL-Architecture.png)

## Architecture Diagram for AWS FSx for OpenZFS with Amazon EKS
**About the Setup**
Amazon Elastic Kubernetes Service (EKS) released CSI drivers for FSx for OpenZFS, allowing easy and fast integration with containers. FSx for OpenZFS is perfectly suited for containerized applications and workloads requiring low latency and high throughput. Containers in a Pod need access to shared data that needs to be persistent beyond the life of the pod, and Amazon FSx for OpenZFS is the highest performance choice available. Here we are going to explore multi AZ setup.

![Diagram](/images/FSxZ-Architecture.png)


## Project Structure

```
.
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── FSxCFN
│   ├── FSXWorkshopOnEKS.yaml
│   ├── FSxONTAP.yaml
│   ├── S3Buckets.yaml
│   ├── S3ReplicationIAM.yaml
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
├── eks
│   ├── FSxL
│   │   ├── claim.yaml
│   │   ├── fsxL-storage-class.yaml
│   │   ├── pod.yaml
│   │   └── pod_performance.yaml
│   ├── FSxN
│   │   ├── backend-ontap-nas.yaml
│   │   ├── mysql
│   │   │   ├── mysql-client.yaml
│   │   │   ├── mysql-configmap.yaml
│   │   │   ├── mysql-service.yaml
│   │   │   ├── mysql-statefulset.yaml
│   │   │   ├── pod_backup_restore.yaml
│   │   │   ├── pvc-from-snap.yaml
│   │   │   ├── snapshot-class.yaml
│   │   │   └── snapshot.yaml
│   │   ├── pod_performance_different_AZ.yaml
│   │   ├── pod_performance_same_AZ.yaml
│   │   ├── pvc-trident.yaml
│   │   ├── storage-class-csi-nas.yaml
│   │   └── svm_secret.yaml
│   ├── FSxZ
│   │   ├── filesystem
│   │   │   ├── delete_me_sc.yaml
│   │   │   ├── fsxZ-storage-claim.yaml
│   │   │   ├── fsxZ-storage-class.yaml
│   │   │   └── pod-fsxZ-filesystem.yaml
│   │   └── volume
│   │       ├── delete_me_sc.yaml
│   │       ├── fsxZ-volume-claim.yaml
│   │       ├── fsxZ-volume-storage-class.yaml
│   │       └── pod-fsxZ-vol.yaml
│   └── cluster.yaml
└── images
    ├── FSxL-Architecture.png
    ├── FSxN-Architecture.png
    └── FSxZ-Architecture.png
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
