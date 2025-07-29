---
title : "Replicate the data between the regions for Amazon EKS cluster"
weight : 260
---
-------------------------------------------------------------

In this section, you will be performing a cross region replication of data between the Amazon S3 bucket. Which will enable the data movement from one EKS cluster to the other EKS cluster in `us-east-2`. To complete this section you will deploy the pod in the current region and sync the data between the regions. Deploy the pod to read the replicated data from the persistent storage layer of Amazon FSx for Lustre file system running in `us-east-2`. Let's start

### Step 1: Deploy the pod for the experiment in the EKS cluster in your current region

::::expand{header="Confirm your access to the EKS cluster, Only run these commands if you have not yet done so in the previous steps."}

- Check if region and cluster names are set correctly, if not then follow one of the suitable page for your situation under **[Getting Started ](/020-setup)** to setup these variables. 

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
echo $PRIMARY_REGION
echo $SECONDARY_REGION
echo $PRIMARY_CLUSTER_NAME
echo $SECONDARY_CLUSTER_NAME
:::

- Next update kubeconfig file to point it to EKS cluster in that region.

::code[aws eks update-kubeconfig --name $PRIMARY_CLUSTER_NAME --region $PRIMARY_REGION]{language=bash showLineNumbers=false showCopyAction=true}

- Run kubectl command to confirm your access to the EKS cluster

::code[kubectl get nodes]{language=bash showLineNumbers=false showCopyAction=true}

::::


Go to the right working directory.

::code[cd /home/participant/environment/fsx-workshop-on-eks/static/eks/FSxL]{language=bash showLineNumbers=false showCopyAction=true}

Copy and run the below command to deploy the `pod.yaml`

::code[kubectl apply -f pod.yaml]{language=bash showLineNumbers=false showCopyAction=true}


Let us have a look at the `pod.yaml`. You can see that the pod mounts the PVC created in the previous section, and write the `date` command to `/data/out.txt` every 5 seconds.

:::code[]{language=yaml showLineNumbers=false showCopyAction=false}
apiVersion: v1
kind: Pod
metadata:
  name: fsx-app
spec:
  containers:
  - name: app
    image: amazonlinux:2
    command: ["bash"]
    securityContext:
      privileged: true
    args: ["-c", "while true; do echo $(date -u) >> /data/out.txt; sleep 5; done"]
    lifecycle:
      postStart:
        exec:
          command: ["amazon-linux-extras", "install", "lustre2.10", "-y"]
    volumeMounts:
    - name: persistent-storage
      mountPath: /data
  volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: fsx-lustre-claim
:::

::alert[This example uses lifecycle hook to install lustre client for demonstration purpose, a normal approach will be building a container image with lustre client]

### Step 2: Login to the container and manually sync the changes to S3 bucket

Copy and run the below command to login to the container

::code[kubectl exec -it fsx-app -- bash]{language=bash showLineNumbers=false showCopyAction=true}

Archive the data from the FSx for Lustre file system into the linked S3 bucket. Run the below commands to manually export the file to the linked S3 bucket

::alert[lfs is a helper utility to administrate lustre cluster]

::code[lfs hsm_archive /data/out.txt]{language=bash showLineNumbers=false showCopyAction=true}

Type "exit" to exit the container
::code[exit]{language=bash showLineNumbers=false showCopyAction=true}

:::alert{header="Note:" type="info"}
New created files won't be synced back to S3 automatically. In order to sync files to s3ExportPath, you need to install lustre client in your container image and manually run following command to force sync up using `lfs hsm_archive`. And the container should run in `privileged` mode with `CAP_SYS_ADMIN capability`.**
:::

### Step 3: Check both the Source S3 bucket and the Destination S3 Bucket

Copy and run the below command from the cli to Look for the s3 bucket name

::code[aws s3 ls]{language=bash showLineNumbers=false showCopyAction=true}

Copy and run the below command to Check the out.txt are in both s3 buckets. 

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
aws s3 ls s3://$S3_BUCKET/export/
aws s3 ls s3://$S3_BUCKET_2NDREGION/export/
:::

You should be able to see that both S3 buckets have the `out.txt` file

::alert[This could take 1-2 minutes for both buckets to show ]

### Step 4: Check the file is synced in the new EKS cluster in 2nd region 

Run the below command to switch your kube config to the EKS Cluster in region 2nd Region (i.e. `us-east-2`)

::code[aws eks update-kubeconfig --name $SECONDARY_CLUSTER_NAME --region $SECONDARY_REGION]{language=bash showLineNumbers=false showCopyAction=true}

Check your nodes are running on the cluster

::code[kubectl get nodes]{language=bash showLineNumbers=false showCopyAction=true}

### Step 5: Validate the PVC

:::alert{header="Note" type="info"}
For AWS Sponsored Workshop, the second region PVC **fsx-lustre-claim** is pre-created for you
:::

Run the below command to check the pvc status

::code[kubectl get pvc]{language=bash showLineNumbers=false showCopyAction=true}

::::expand{header="You should see the results as below, click to expand"}

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
fsx-lustre-claim   Bound    pvc-CCCCCC----02468c18769e   1200Gi     RWX            fsx-lustre-sc         7m37s
:::

::::

You can see the new filesystem is displayed from AWS FSx console

::alert[This should be in the **us-east-2** region in the AWS Console]

![FSxL_01](/static/images/FSxL_01.png)

### Step 6: Now let's deploy the pod

Copy and run the below command to deploy the pod.

::code[kubectl apply -f pod.yaml]{language=bash showLineNumbers=false showCopyAction=true}

### Step 7. Logon to the container to see the `out.txt` is it available for use.

Run the below command to Logon to the container.

::code[kubectl exec -it fsx-app -- bash]{language=bash showLineNumbers=false showCopyAction=true}

Run the below command to verify that the file is available and all the contents are correct. 

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
ls -ltr /data/export/
tail -f /data/export/out.txt
:::

Type `ctrl+c` to exit the "tail" command

Type "exit" to exit the container
::code[exit]{language=bash showLineNumbers=false showCopyAction=true}

## Summary

In this section, you manged to validate the cross region replication of data available for two different EKS clusters in less than a minute. Congratulations, You have sucessfully completed this module.
