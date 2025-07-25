---
title : "On Demand Workshop"
weight : 10
---
-------------------------------------------------------------

:::alert{header="Important" type="warning"}
If you are in **AWS SPONSORED WORKSHOP** instead of self-paced On Demand Workshop, please SKIP this section and move to the next section **[AWS Sponsored Workshop](/020-setup/02-aws-event)**
:::

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
  ParameterKey=Assets,ParameterValue="" \
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
aws cloudformation wait stack-create-complete --stack-name ${STACK_NAME} --region $PRIMARY_REGION
```

### Part 4 : Access your workshop

**Connect to your AWS lab environment via Open source VSCode IDE**

Ref : [code-server](https://github.com/coder/code-server) 

You will be using the Open source VSCode IDE terminal to copy and paste commands that are provided in this workshop. 

### Note: Please use google chrome browser for best user experience. Firefox may experience some issues while copy-paste commands.

1. Go to Cloud formation console [link](https://console.aws.amazon.com/cloudformation) and select `genaifsxworkshoponeks` stack 
2. Go to Stack **Outputs**
3. Copy Password and click URL
4. Enter copied password in the new tab opened for the URL


![CFN-Output](/static/images/cfn-output.png)

5. Select your VSCode UI theam 

![Select Theme](/static/images/select-theme.png)

6. You can maximize terminal window.

![maximize](/static/images/maximize.png)

### Note: When you first time copy-paste a command on VSCode IDE, your browser may ask you to allow permission to see informaiton on clipboard. Please select **"Allow"**.

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

