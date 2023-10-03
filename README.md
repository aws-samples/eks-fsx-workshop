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

For this walkthrough, you should have the following prerequisites:

1. An AWS account with necessary permissions to create and manage Amazon VPC, Amazon EKS cluster, Amazon FSx for NetApp ONTAP file system, and CloudFormation stack.

2. You will need : 
  - EKS 1.22 and later
  
 On your laptop: 
  - eksctl: https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html
  - kubectl: https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html
  - Helm 3: https://docs.aws.amazon.com/eks/latest/userguide/helm.html


3. The AWS Command Line Interface (AWS CLI) configured in your working environment. For information about installing and configuring the AWS CLI, see Installing or updating the latest version of the AWS CLI. https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

## 1. Clone the Github repository

You can find the CloudFormation template and relevant code in this GitHub repo. Run the following command to clone the repository into your local workstation.

```bash
git clone https://github.com/aws-samples/eks-fsx-workshop.git
```

There are two folders that you need to reference in the following steps, with the “eks” folder containing all manifests files related to the eks cluster resources and “FSxCFN” Cloudformation templates for spinning up the VPC environment and FSx File System.

Set region variables, Replace `<region name>` with your lab primary region name : 

```bash
export REGION_1=<region name>
export REGION_2=us-east-2
```

Set cluster variables : 

```bash
export CLUSTER_NAME_1=FSx-eks-cluster
export CLUSTER_NAME_2=FSx-eks-cluster02
```


## 2. Create a VPC environment for Amazon EKS and FSx (Optional)

Create a new VPC with two private subnets and two public subnets using CloudFormation. This step is optional, and an existing VPC can be reused for the Amazon EKS cluster and the FSx file system.

Launch the CloudFormation stack to set up the network environment for both FSx and EKS cluster:

```bash
cd eks-fsx-workshop/FSxCFN
aws cloudformation create-stack --stack-name FSX-EKS-VPC --template-body file://./vpc-subnets.yaml --region $REGION_1
```

  ![VPCCFN](/images/VPCCFN.png)

## 3. Create an Amazon EKS cluster

In this walkthrough, we are going to create the EKS cluster with a managed node group that contains two worker nodes residing across the two private subnets created in step 2. In the `cluster.yaml` file, substitute the VPC ID and subnet IDs based on the output of the CloudFormation stack launched in step 2.

1. Current working directory is 

```bash
cd ../eks
```
      
2.  Edit the file cluster.yaml and replace the region, vpcid, private subnet and public subnet


![VPCREGION](/images/vpcregion.png) 

If you are using your own existing VPC and subnets then copy the following CLI command in a notepad to modify it to set values.
```bash
VPCID=<your vpc id>
PUBLIC_SUBNET_1=<your public subnet 1>
PUBLIC_SUBNET_1=<your public subnet 2>
PRIVATE_SUBNET_1=<your private subnet 1>
PRIVATE_SUBNET_2=<your private subnet 2>
PUBLIC_ROUTETABLE=<your public route table>
PRIVATE_ROUTETABLE=<your private route table>
```

Else get values from the cloud formation stack created in Step 2 above : 
```bash
VPCID=$(aws cloudformation describe-stacks --stack-name FSX-EKS-VPC --region $REGION_1 --query "Stacks[0].Outputs[?OutputKey=='VPCId'].OutputValue" --output text)
VPCCIDR=$(aws cloudformation describe-stacks --stack-name FSX-EKS-VPC --region $REGION_1 --query "Stacks[0].Outputs[?OutputKey=='VpcCidrBlock'].OutputValue" --output text)
PUBLIC_SUBNET_1=$(aws cloudformation describe-stacks --stack-name FSX-EKS-VPC --region $REGION_1 --query "Stacks[0].Outputs[?OutputKey=='PublicSubnet1'].OutputValue" --output text)
PUBLIC_SUBNET_2=$(aws cloudformation describe-stacks --stack-name FSX-EKS-VPC --region $REGION_1 --query "Stacks[0].Outputs[?OutputKey=='PublicSubnet2'].OutputValue" --output text)
PRIVATE_SUBNET_1=$(aws cloudformation describe-stacks --stack-name FSX-EKS-VPC --region $REGION_1 --query "Stacks[0].Outputs[?OutputKey=='PrivateSubnet1'].OutputValue" --output text)
PRIVATE_SUBNET_2=$(aws cloudformation describe-stacks --stack-name FSX-EKS-VPC --region $REGION_1 --query "Stacks[0].Outputs[?OutputKey=='PrivateSubnet2'].OutputValue" --output text)
PUBLIC_ROUTETABLE=$(aws cloudformation describe-stacks --stack-name FSX-EKS-VPC --region $REGION_1 --query "Stacks[0].Outputs[?OutputKey=='PublicRouteTable'].OutputValue" --output text)
PRIVATE_ROUTETABLE=$(aws cloudformation describe-stacks --stack-name FSX-EKS-VPC --region $REGION_1 --query "Stacks[0].Outputs[?OutputKey=='PrivateRouteTable'].OutputValue" --output text)
```

```bash
sed -i'' -e "s/ap-southeast-2/$REGION_1/g" cluster.yaml
sed -i'' -e "s/vpc-id/$VPCID/g" cluster.yaml
sed -i'' -e "s/public-subnet-1/$PUBLIC_SUBNET_1/g" cluster.yaml
sed -i'' -e "s/public-subnet-2/$PUBLIC_SUBNET_2/g" cluster.yaml
sed -i'' -e "s/private-subnet1/$PRIVATE_SUBNET_1/g" cluster.yaml
sed -i'' -e "s/private-subnet2/$PRIVATE_SUBNET_2/g" cluster.yaml
```

Verify updated values : 

```bash
cat cluster.yaml
```

- Create the EKS cluster by running the following command:
```bash
eksctl create cluster -f ./cluster.yaml
```

- Create EKS Cluster in 2nd region for testing cross region desaster recovery and OpenZFS

```bash
eksctl create cluster --name $CLUSTER_NAME_2 --region $REGION_2 --nodes=2 --instance-types=c5.2xlarge
```

The above EKS Cluster creation will complete approximately in 30 Mins. 


## 4. Create an Amazon FSx for NetApp ONTAP file system

The following steps to create the Amazon FSx for Netapp ONTAP Filesystem. If you want to skip and move to Amazon FSx for Lustre then go to to step 5

Run the following CLI command to create the Amazon FSx for NetApp ONTAP file system. (Note that you need to modify the parameters based on your VPC environment created as above.)

1. Copy the following CLI command in a notepad to modify it to define your password
   
```bash
aws cloudformation create-stack \
  --stack-name EKS-FSXONTAP \
  --template-body file://./FSxONTAP.yaml \
  --region $REGION_1 \
  --parameters \
  ParameterKey=Subnet1ID,ParameterValue=$PRIVATE_SUBNET_1 \
  ParameterKey=Subnet2ID,ParameterValue=$PRIVATE_SUBNET_2 \
  ParameterKey=myVpc,ParameterValue=$VPCID \
  ParameterKey=FSxONTAPRouteTable,ParameterValue=$PUBLIC_ROUTETABLE,$PRIVATE_ROUTETABLE \
  ParameterKey=FileSystemName,ParameterValue=EKS-myFSxONTAP \
  ParameterKey=ThroughputCapacity,ParameterValue=128 \
  ParameterKey=FSxAllowedCIDR,ParameterValue=$VPCCIDR \
  ParameterKey=FsxAdminPassword,ParameterValue=[Define password] \
  ParameterKey=SvmAdminPassword,ParameterValue=[Define password] \
  --capabilities CAPABILITY_NAMED_IAM
```

2. Change the region that you are operating

3. Enter the values that was captured earlier during VPC creation as show in the below screenshot
  
   ![VPCCFN](/images/VPCCFN.png)

4. Throughput capacity input is 128MB/s. However if you wish to increase it the supported values are 256, 512, 1024 and 2048. 

5. Default storage capacity for this deployment type is 1024


This CloudFormation stack will take approximately 45 minutes to complete; feel free to move to step 4 while waiting for the file system to be deployed.

After the completion of the deployment, we can verify in the following screenshot that the FSx NetApp ONTAP file system and Storage Virtual Machine (SVM) are created.
![FSX01](/images/fsx01.png)

Take a look at the details of the FSx for NetApp ONTAP file system. We can see that the file system has a primary subnet and a standby subnet.

![ONTAP04](/images/ontap4.png)

SVM is also created.

![ONTAP05](/images/ontap5.png)


## 5. Create needful resources for Amazon FSx for Lustre file system


- Creating S3 bucket for FSx Luster in primary region

```bash
random=`tr -dc A-Za-z0-9 </dev/urandom | head -c 13 ; echo ''`
s3_bucket_name="fsx-luster-bucket-${random}"
s3_2nd_bucket_name="fsx-luster-bucket-2ndregion-${random}"
```


```bash
aws cloudformation create-stack \
  --stack-name ${s3_bucket_name} \
  --region $REGION_1 \
  --template-body file://./S3Buckets.yaml \
  --parameters \
  ParameterKey=S3BucketName,ParameterValue=${s3_bucket_name} \
  --capabilities CAPABILITY_NAMED_IAM 
```

- Creating S3 bucket in 2nd region 
  
```bash
aws cloudformation create-stack \
  --stack-name ${s3_2nd_bucket_name} \
  --region $REGION_2 \
  --template-body file://./S3Buckets.yaml \
  --parameters \
  ParameterKey=S3BucketName,ParameterValue=${s3_2nd_bucket_name} \
  --capabilities CAPABILITY_NAMED_IAM 
```


- Creating IAM Role and Policy for S3 Cross Region Replication
  
```bash
iam_role_name="s3-crr-${random}"
sed -i "s/DOC-EXAMPLE-BUCKET1/$s3_bucket_name/g" S3ReplicationIAM.yaml
sed -i "s/DOC-EXAMPLE-BUCKET2/$s3_2nd_bucket_name/g" S3ReplicationIAM.yaml  
```

```bash
aws cloudformation create-stack \
  --stack-name ${iam_role_name} \
  --region $REGION_1 \
  --template-body file://./S3ReplicationIAM.yaml \
  --parameters ParameterKey=IAMRoleName,ParameterValue=${iam_role_name} \
  --capabilities CAPABILITY_NAMED_IAM
```

- Creating Security Group for FSx for Lustre

```bash
sed -i "s/myVpc/$VPCID/g" fsxL-SecurityGroup.yaml
```

```bash
aws cloudformation create-stack \
  --stack-name FSxL-SecurityGroup-01 \
  --region $REGION_1 \
  --template-body file://./fsxL-SecurityGroup.yaml \
  --parameters ParameterKey=SecurityGroupName,ParameterValue=FSxLSecurityGroup01 \
  --capabilities CAPABILITY_NAMED_IAM 
```

```bash
aws cloudformation create-stack \
  --stack-name FSxL-SecurityGroup-02 \
  --region $REGION_2 \
  --template-body file://./fsxL-SecurityGroup.yaml \
  --parameters ParameterKey=SecurityGroupName,ParameterValue=FSxLSecurityGroup02 \
  --capabilities CAPABILITY_NAMED_IAM 
```

## 6. Create needful resources for Amazon FSx for OpenZFS 

Create Security Group for OpenZFS
              
`sed -i "s/myVpc/$VPCID/g" fsxZ-SecurityGroup.yaml`

```bash
aws cloudformation create-stack \
  --stack-name fsxZ-SecurityGroup \
  --region $REGION_1 \
  --template-body file://./fsxZ-SecurityGroup.yaml \
  --parameters ParameterKey=SecurityGroupName,ParameterValue=FSxOSecurityGroup \
  --capabilities CAPABILITY_NAMED_IAM 
```


## Clean up
Run these commands one by one in given sequence : 

```bash
eksctl delete nodegroup --region $REGION_2  --cluster $CLUSTER_NAME_2 --name $(eksctl get nodegroup --cluster $CLUSTER_NAME_2 --region $REGION_2 --output json | jq -r .[].Name)
eksctl delete cluster --name=$CLUSTER_NAME_2 --region $REGION_2 
eksctl delete nodegroup --cluster $CLUSTER_NAME_1 --name $(eksctl get nodegroup --cluster $CLUSTER_NAME_1 --output json | jq -r .[].Name)
eksctl delete cluster --name=$CLUSTER_NAME_1 
aws cloudformation delete-stack --stack-name FSxL-SecurityGroup-02 --region $REGION_2
aws cloudformation delete-stack --stack-name FSxL-SecurityGroup-01
aws cloudformation delete-stack --stack-name fsxZ-SecurityGroup
aws cloudformation delete-stack --stack-name ${iam_role_name}
aws cloudformation delete-stack --stack-name ${s3_2nd_bucket_name} --region $REGION_2
aws cloudformation delete-stack --stack-name ${s3_bucket_name}
aws cloudformation delete-stack --stack-name EKS-FSXONTAP
aws cloudformation delete-stack --stack-name FSX-EKS-VPC
```


