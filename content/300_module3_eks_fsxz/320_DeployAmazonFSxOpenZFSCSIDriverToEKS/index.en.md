---
title : "Deploy Amazon FSx for OpenZFS CSI Driver"
weight : 320
---
-------------------------------------------------------------

In this section, the following steps will guide you to deploy CSI Driver of Amazon FSx for OpenZFS. As a prerequisite step you will have to create a service account & IAM poilcy with role ARN. Then proceed with the CSI driver deployment for FSx for OpenZFS filesystem.

Below steps to ensure that you are operating from the right AWS region and in the right working directory with the access to the EKS cluster.

::alert[As mentioned in the previous session, you should have run the following commands to be able to run `kubectl` to connect to your EKS cluster. But if you missed it, please run the following commands]

Run these commands to update EKS cluster context to point back to EKS Cluster in current region. 

- Check if region and cluster names are set correctly, if not then follow one of the suitable page for your situation under **[Getting Started ](/020-setup)** to setup these variables. 

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
echo $PRIMARY_REGION
echo $SECONDARY_REGION
echo $PRIMARY_CLUSTER_NAME
echo $SECONDARY_CLUSTER_NAME
:::

- Run the below command to switch your kube config to the FSx-eks-cluster02 in region us-east-2

::code[aws eks update-kubeconfig --name $PRIMARY_CLUSTER_NAME --region $PRIMARY_REGION]{language=bash showLineNumbers=false showCopyAction=true}

- Run kubectl command to confirm your access to the EKS cluster

::code[kubectl get nodes]{language=bash showLineNumbers=false showCopyAction=true}



Go to the right working directory.

::code[cd /home/participant/environment/fsx-workshop-on-eks/static/eks/FSxZ]{language=bash showLineNumbers=false showCopyAction=true}

The below steps will guide you to create a service account, create and attach an IAM policy and deploy the CSI driver for Amazon FSx for OpenZFS.

## Step 1 : Create a Kubernetes service account for the driver and attach the policy to the service account

Copy and run the below command to create the service account and attach the IAM policy created in Step 3. 

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
eksctl create iamserviceaccount \
    --name fsx-openzfs-csi-controller-sa \
    --namespace kube-system \
    --cluster $PRIMARY_CLUSTER_NAME \
    --attach-policy-arn arn:aws:iam::aws:policy/AmazonFSxFullAccess \
    --approve \
    --role-name AmazonEKSFSxOpenZFSCSIDriverFullAccess \
    --region $PRIMARY_REGION
:::

Wait for about 30 to 45 seconds.

::::expand{header="You’ll see several lines of output as the service account is created. The last line of output is similar to the following example line, click to expand"}

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
(...)
2023-10-02 05:24:33 [ℹ]  created serviceaccount "kube-system/fsx-openzfs-csi-controller-sa"
:::
::::

## Step 2 : Deploy the CSI driver of FSx for OpenZFS

Copy and the run the following command to update helm repo.

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
helm repo add aws-fsx-openzfs-csi-driver https://kubernetes-sigs.github.io/aws-fsx-openzfs-csi-driver
helm repo update
:::

Copy and the run the following command to deploy the CSI driver.

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
helm upgrade --install aws-fsx-openzfs-csi-driver \
    --namespace kube-system \
    --set controller.serviceAccount.create=false \
    aws-fsx-openzfs-csi-driver/aws-fsx-openzfs-csi-driver
:::

::::expand{header="You should see the results as below, click to expand"}
You should see output like this: 

:::code[]{language=yaml showLineNumbers=false showCopyAction=false}
Release "aws-fsx-openzfs-csi-driver" does not exist. Installing it now.
NAME: aws-fsx-openzfs-csi-driver
LAST DEPLOYED: Fri Jan 26 04:34:28 2024
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
:::
::::

Verify whether the CSI driver has been installed successfully with the following command.

::code[kubectl get pods -n kube-system -l app.kubernetes.io/part-of=aws-fsx-openzfs-csi-driver]{language=bash showLineNumbers=false showCopyAction=true}

::::expand{header="You should see the results as below, click to expand"}

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
fsx-openzfs-csi-controller-67d9cf6554-bsbpd   5/5     Running   0          2m17s
fsx-openzfs-csi-controller-67d9cf6554-rmx85   5/5     Running   0          2m17s
fsx-openzfs-csi-node-rctch                    3/3     Running   0          2m17s
fsx-openzfs-csi-node-x6hpf                    3/3     Running   0          2m17s
:::

::::

## Summary

In this section you have completed the pre-requisite tasks of creating service account with the right IAM policy and role ARN. Post which you have deployed the CSI driver of FSx for OpenZFS and verified CSI Driver installation. In the next section you will create the storage class and the persistent volume claim (PVC) for FSx for OpenZFS file system. 