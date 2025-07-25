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
├── README.md
├── assets
│   └── README.md
├── content
│   ├── 010_introduction
│   │   └── index.en.md
│   ├── 020_setup
│   │   ├── 01_self_paced
│   │   │   └── index.en.md
│   │   ├── 02_aws_event
│   │   │   └── index.en.md
│   │   └── index.en.md
│   ├── 100_module1_eks_fsxn
│   │   ├── 110_solutionoverview
│   │   │   └── index.en.md
│   │   ├── 120_DeployTridentOperator
│   │   │   └── index.en.md
│   │   ├── 130_ProvisionTridentVolumes_StorageClass
│   │   │   └── index.en.md
│   │   ├── 140_DeployMySQL_StatefulApps
│   │   │   └── index.en.md
│   │   ├── 150_ScaleMySQL_Statefulsets
│   │   │   └── index.en.md
│   │   ├── 160_BackupAndRestore
│   │   │   └── index.en.md
│   │   ├── 180_PerformanceTesting
│   │   │   └── index.en.md
│   │   ├── 190_AvailabilityZone_Failover
│   │   │   └── index.en.md
│   │   └── index.en.md
│   ├── 200_module2_eks_fsxl
│   │   ├── 210_SolutionOverview
│   │   │   └── index.en.md
│   │   ├── 220_DeployAmazonFSxLustreCSIDriverToEKS
│   │   │   └── index.en.md
│   │   ├── 230_DeployStorageclassAndPVC
│   │   │   └── index.en.md
│   │   ├── 240_PerformanceTesting
│   │   │   └── index.en.md
│   │   ├── 250_CreateCrossRegionReplicationForS3Buckets
│   │   │   └── index.en.md
│   │   ├── 260_RegionalFailoverAndSwitch
│   │   │   └── index.en.md
│   │   └── index.en.md
│   ├── 300_module3_eks_fsxz
│   │   ├── 310_SolutionOverview
│   │   │   └── index.en.md
│   │   ├── 320_DeployAmazonFSxOpenZFSCSIDriverToEKS
│   │   │   └── index.en.md
│   │   ├── 330_DynamicOpenZFSProvisioning
│   │   │   └── index.en.md
│   │   ├── 340_DynamicVolumeProvisioning
│   │   │   └── index.en.md
│   │   ├── 350_PerformanceTesting
│   │   │   └── index.en.md
│   │   ├── 360_AvailabilityZone_Failover
│   │   │   └── index.en.md
│   │   └── index.en.md
│   ├── index.en.md
│   └── summary
│       └── index.en.md
├── contentspec.yaml
└── static
    ├── FSXWorkshopOnEKS.yaml
    ├── FSxCFN
    │   ├── FSXWorkshopOnEKS.yaml
    │   ├── FSxONTAP.yaml
    │   ├── S3Buckets.yaml
    │   ├── S3ReplicationIAM.yaml
    │   ├── fsx-csi-driver.json
    │   ├── fsxL-SecurityGroup.yaml
    │   ├── fsxZ-SecurityGroup.yaml
    │   └── vpc-subnets.yaml
    ├── aws-logo.png
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
    │   │   ├── filesystem-maz
    │   │   │   ├── fsxZ-storage-claim.yaml
    │   │   │   ├── fsxZ-storage-class.yaml
    │   │   │   └── pod-fsxZ-filesystem.yaml
    │   │   ├── filesystem-saz
    │   │   │   ├── fsxZ-storage-claim.yaml
    │   │   │   ├── fsxZ-storage-class.yaml
    │   │   │   └── pod-fsxZ-filesystem.yaml
    │   │   ├── volume-maz
    │   │   │   ├── fsxZ-volume-claim.yaml
    │   │   │   ├── fsxZ-volume-storage-class.yaml
    │   │   │   └── pod-fsxZ-vol.yaml
    │   │   └── volume-saz
    │   │       ├── fsxZ-volume-claim.yaml
    │   │       ├── fsxZ-volume-storage-class.yaml
    │   │       └── pod-fsxZ-vol.yaml
    │   └── cluster.yaml
    ├── images
    │   ├── < Workshop Images ...>
        (...)
```



### Part 1 : Prerequisite of setting up On-demand Workshop

For this walkthrough, you should have the following prerequisites:

1. An AWS account with necessary permissions to create and manage Amazon VPC, subresources under VPC, Amazon EKS cluster, S3 Bucket, Amazon FSx for NetApp ONTAP, Amazon FSx Lustre, Amazon FSx OpenZFS file system, and CloudFormation stack.

2. The AWS Command Line Interface (AWS CLI) configured in your working environment. For information about installing and configuring the AWS CLI, see Installing or updating the latest version of the AWS CLI. https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

### Part 2 : Clone the Github repository

You can find the CloudFormation template and relevant code in this GitHub repo. Run the following command to clone the repository into your local workstation.

```bash
git clone https://github.com/aws-samples/eks-fsx-workshop.git
```

There are two folders that you need to reference in the following steps, with the “eks” folder containing all manifests files related to the eks cluster resources and “FSxCFN” Cloudformation templates for spinning up the VPC, FSx File Systems, EKS Clusters ..etc.

### Part 3 : Provision workshop resources

#### Upload template to S3
Upload AWS CloudFormation template to your bucket in order to validate template before stack creation. Please replace **YOUR_BUCKET_NAME** in following command.

```bash
ASSET_BUCKET=YOUR_BUCKET_NAME
ASSET_BUCKET_PATH=eks-fsx-workshop
```

```bash
aws s3 sync ./eks-fsx-workshop s3://${ASSET_BUCKET}/${ASSET_BUCKET_PATH}
```


#### Validate template

```bash
aws cloudformation validate-template --template-url https://${ASSET_BUCKET}.s3.amazonaws.com/${ASSET_BUCKET_PATH}/static/FSXWorkshopOnEKS.yaml
```

#### Create stack

Please make sure you replace **YOUR_PREFERED_PRIMARY_REGION** and **YOUR_PREFERED_SECONDARY_REGION** in following command.


```bash
export PRIMARY_REGION=YOUR_PREFERED_PRIMARY_REGION
export SECONDARY_REGION=YOUR_PREFERED_SECONDARY_REGION
STACK_NAME=FSXWorkshopOnEKS
VSINSTANCE_NAME=VSCodeServerForEKS
ASSET_BUCKET_ZIPPATH=""
```

This may take upto 60 mins : 

```bash
aws cloudformation create-stack \
  --stack-name ${STACK_NAME} \
  --template-url https://${ASSET_BUCKET}.s3.amazonaws.com/${ASSET_BUCKET_PATH}/static/FSXWorkshopOnEKS.yaml \
  --region $PRIMARY_REGION \
  --parameters \
  ParameterKey=VSCodeUser,ParameterValue=participant \
  ParameterKey=InstanceName,ParameterValue=${VSINSTANCE_NAME} \
  ParameterKey=InstanceVolumeSize,ParameterValue=100 \
  ParameterKey=InstanceType,ParameterValue=t4g.medium \
  ParameterKey=InstanceOperatingSystem,ParameterValue=AmazonLinux-2023 \
  ParameterKey=HomeFolder,ParameterValue=environment \
  ParameterKey=DevServerPort,ParameterValue=8081 \
  ParameterKey=AssetZipS3Path,ParameterValue=${ASSET_BUCKET_ZIPPATH} \
  ParameterKey=SecondaryRegion,ParameterValue=${SECONDARY_REGION} \
  --disable-rollback \
  --capabilities CAPABILITY_NAMED_IAM

# Wait for stack creation to complete
aws cloudformation wait stack-create-complete --stack-name ${STACK_NAME} --region PRIMARY_REGION
```

### Part 4 : Access your workshop

**Connect to your AWS lab environment via Open source VSCode IDE**

Ref : [code-server](https://github.com/coder/code-server) 

You will be using the Open source VSCode IDE terminal to copy and paste commands that are provided in this workshop. 

::alert[Note: Please use google chrome browser for best user experience. Firefox may experience some issues while copy-paste commands.]{header="Important" type="warning"}

1. Go to Cloud formation console [link](https://console.aws.amazon.com/cloudformation) and select `genaifsxworkshoponeks` stack 
2. Go to Stack **Outputs**
3. Copy Password and click URL
4. Enter copied password in the new tab opened for the URL


![CFN-Output](/static/images/cfn-output.png)

5. Select your VSCode UI theam 

![Select Theme](/static/images/select-theme.png)

6. You can maximize terminal window.

![maximize](/static/images/maximize.png)

### Note: 
When you first time copy-paste a command on VSCode IDE, your browser may ask you to allow permission to see informaiton on clipboard. Please select **"Allow"**.

![allow-clipboard](/static/images/allow-clipboard.png)


- Set cluster variables : 

```bash
export PRIMARY_CLUSTER_NAME=FSx-eks-cluster
export SECONDARY_CLUSTER_NAME=FSx-eks-cluster02
```

- Check if region and cluster names are set correctly

```bash
echo "PRIMARY_REGION        :" $PRIMARY_REGION
echo "SECONDARY_REGION      :" $SECONDARY_REGION
echo "PRIMARY_CLUSTER_NAME  :" $PRIMARY_CLUSTER_NAME
echo "SECONDARY_CLUSTER_NAME:" $SECONDARY_CLUSTER_NAME
```

## Update the kube-config file:
Before you can start running all the Kubernetes commands included in this workshop, you need to update the kube-config file with the proper credentials to access the cluster. To do so, in your VSCode IDE terminal run the below command:

```bash
aws eks update-kubeconfig --name $PRIMARY_CLUSTER_NAME --region $PRIMARY_REGION
```


## Query the Amazon EKS cluster:
Run the command below to see the Kubernetes nodes currently provisioned:

```bash
kubectl get nodes
```

You should see two nodes provisioned (which are the on-demand nodes used by the Kubernetes controllers), such as the output below:


![get-nodes](/static/images/get-nodes.png)


You now have a VSCode IDE Server environment set-up ready to use your Amazon EKS Cluster! You may now proceed with the next step.

::alert[**DO NOT CLICK ON NEXT BELOW**: Please head straight to **[Module 1 - Running Amazon FSx for NetApp ONTAP on Amazon EKS](/100_module1_eks_fsxn/)** for the next action.]{header="Important" type="warning"}

Once you finish with workshop and ready to clean up, visit this page to execute commands in Part 5 Clean up below.

### Part 5 : Clean up
This may take upto 30 mins : 

```bash
aws cloudformation delete-stack --stack-name ${STACK_NAME} --region PRIMARY_REGION

# Wait for stack deletion to complete
aws cloudformation wait stack-delete-complete --stack-name ${STACK_NAME} --region PRIMARY_REGION
```

