---
title : "Amazon FSx for OpenZFS Filesystem Dynamic Provisioning"
weight : 330
---
-------------------------------------------------------------

In this section you will define the storageclass variables and create the storageclass for Amazon FSx for OpenZFS. Later you will create the persistent volume claim (PVC) and deploy the storage class. Observe the FSx for OpenZFS file system is being auto provisioned. 


### Step 1: Prerequisite - setting the required environmental variable

Copy and paste the following lines in the CLI.


:::code[]{language=bash showLineNumbers=true showCopyAction=true}
PRIMARY_CLUSTER_NAME="FSx-eks-cluster"
VPC_ID=$(aws eks describe-cluster --name $PRIMARY_CLUSTER_NAME --region $PRIMARY_REGION --query "cluster.resourcesVpcConfig.vpcId" --output text)
SUBNET_ID_1=$(aws ec2 describe-subnets --filters Name=vpc-id,Values=$VPC_ID --region $PRIMARY_REGION --query "Subnets[?AvailabilityZone=='${PRIMARY_REGION}a'].SubnetId | [0]" --output text)
SUBNET_ID_2=$(aws ec2 describe-subnets --filters Name=vpc-id,Values=$VPC_ID --region $PRIMARY_REGION --query "Subnets[?AvailabilityZone=='${PRIMARY_REGION}b'].SubnetId | [0]" --output text)
ROUTE_TABLE_1=$(aws ec2 describe-route-tables --filters "Name=vpc-id,Values=$VPC_ID" --query "RouteTables[?Associations[*].Main != 'true'] && RouteTables[].Associations[?SubnetId=='${SUBNET_ID_1}'].RouteTableId" --region $PRIMARY_REGION --output text)
ROUTE_TABLE_2=$(aws ec2 describe-route-tables --filters "Name=vpc-id,Values=$VPC_ID" --query "RouteTables[?Associations[*].Main != 'true'] && RouteTables[].Associations[?SubnetId=='${SUBNET_ID_2}'].RouteTableId" --region $PRIMARY_REGION --output text )
SECURITY_GROUP_ID=$(aws cloudformation describe-stacks --stack-name fsxZ-SecurityGroup --region $PRIMARY_REGION --query "Stacks[0].Outputs[0].OutputValue" --output text)
FSX_KEY=$(aws kms list-aliases --region $PRIMARY_REGION --query 'Aliases[?AliasName==`alias/aws/fsx`].TargetKeyId' --output text)
:::


For more information about what rule is required for the FSx OpenZFS Security Group, please refer to the [official document](https://docs.aws.amazon.com/fsx/latest/OpenZFSGuide/limit-access-security-groups.html).

### Step 2: Define the storageclass

In the following steps you will be using the environment variables which was created in the previous section.

Run the below command to verify values for these variables. 

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
echo $SUBNET_ID_1
echo $SUBNET_ID_2
echo $SECURITY_GROUP_ID
echo $FSX_KEY
echo $ROUTE_TABLE_1
echo $ROUTE_TABLE_2
:::

Go to the right working directory.

::code[cd /home/participant/environment/fsx-workshop-on-eks/static/eks/FSxZ/filesystem-maz]{language=bash showLineNumbers=false showCopyAction=true}

We are going to replace `SUBNET_ID_1`,`SUBNET_ID_2`, `FSX_KEY` and `SECURITY_GROUP_ID`, `ROUTE_TABLE` in `fsxZ-storage-class.yaml` next.

::code[cat fsxZ-storage-class.yaml]{language=bash showLineNumbers=false showCopyAction=true}

::::expand{header="Click to expand the section to see output of the fsxZ-storage-class.yaml file"}
:::code[]{language=yaml showLineNumbers=false showCopyAction=false}
#fsxZ-storage-class.yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fsx-openzfs-sc-maz
provisioner: fsx.openzfs.csi.aws.com
parameters:
  ResourceType: "filesystem" 
  DeploymentType: '"MULTI_AZ_1"' 
  ThroughputCapacity: '160' 
  SubnetIds: '["SUBNET_ID_1","SUBNET_ID_2"]'  
  PreferredSubnetId: '"SUBNET_ID_1"'
  RouteTableIds: '["ROUTE_TABLE"]'
  SkipFinalBackupOnDeletion: 'true' 
  OptionsOnDeletion: '["DELETE_CHILD_VOLUMES_AND_SNAPSHOTS"]' 
  KmsKeyId: '"FSX_KEY"'    
  AutomaticBackupRetentionDays: '1'
  CopyTagsToBackups: 'false'
  CopyTagsToVolumes: 'false'
  DailyAutomaticBackupStartTime: '"00:00"'
  DiskIopsConfiguration: '{"Iops": 3600, "Mode": "USER_PROVISIONED"}'
  RootVolumeConfiguration: '{"CopyTagsToSnapshots": false, "DataCompressionType": "NONE", "NfsExports": [{"ClientConfigurations": [{"Clients": "*", "Options": ["rw","crossmnt"]}]}], "ReadOnly": false, "RecordSizeKiB": 128, "UserAndGroupQuotas": [{"Type": "USER", "Id": 1, "StorageCapacityQuotaGiB": 10}]}'
  WeeklyMaintenanceStartTime: '"7:09:00"'
  SecurityGroupIds: '["SECURITY_GROUP_ID"]'     
  Tags: '[{"Key": "OPENZFS", "Value": "OPENZFS"}]'
reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:
  - nfsvers=4.1
  - rsize=1048576
  - wsize=1048576
  - timeo=600
:::
::::

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
sed -i'' -e "s/SUBNET_ID_1/$SUBNET_ID_1/g" fsxZ-storage-class.yaml
sed -i'' -e "s/SUBNET_ID_2/$SUBNET_ID_2/g" fsxZ-storage-class.yaml
sed -i'' -e "s/SECURITY_GROUP_ID/$SECURITY_GROUP_ID/g" fsxZ-storage-class.yaml
sed -i'' -e "s/FSX_KEY/$FSX_KEY/g" fsxZ-storage-class.yaml
sed -i'' -e "s/ROUTE_TABLE/${ROUTE_TABLE_1}/g" fsxZ-storage-class.yaml
:::


Verify if all the input is saved by running the below command.

::code[cat fsxZ-storage-class.yaml]{language=bash showLineNumbers=false showCopyAction=true}
 
::::expand{header="Click to expand the section to understand the settings defined in the fsxZ-storage-class.yaml file"}

* **SubnetIds** – The subnet ID that the Amazon FSx for OpenZFS file system should be created in. Amazon FSx for OpenZFS is not supported in all Availability Zones. Open the Amazon FSx for OpenZFS console at `https://console.aws.amazon.com/fsx/` to confirm that the subnet that you want to use is in a supported Availability Zone. The subnet can include your nodes, or can be a different subnet or VPC. If the subnet that you specify is not the same subnet that you have nodes in, then your VPCs must be connected, and you must ensure that you have the necessary ports open in your security groups.
* **SecurityGroupIds** – The security group ID for your nodes.
* **KmsKeyId** - Amazon managed KMS key used with OpenZFS.
* **RouteTableIds** - Route table used by OpenZFS
* **PreferredSubnetId** - Preferred Subnet Id of OpenZFS

Check other Parameters detail [here](https://github.com/kubernetes-sigs/aws-fsx-openzfs-csi-driver/blob/main/docs/parameters.md#file-system-parameters) and [API reference](https://docs.aws.amazon.com/fsx/latest/APIReference/API_CreateFileSystemOpenZFSConfiguration.html#API_CreateFileSystemOpenZFSConfiguration_Contents)

::::


### Step 3: Create the storageclass

Copy and run the below command to apply the defined settings from the step 1. This will create the storageclass.

::code[kubectl apply -f fsxZ-storage-class.yaml]{language=bash showLineNumbers=false showCopyAction=true}

Copy and run the below command to verify the storageclass is created.

::code[kubectl get sc]{language=bash showLineNumbers=false showCopyAction=true}

::::expand{header="You should see the results as below, click to expand"}

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
NAME                 PROVISIONER               RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
fsx-openzfs-sc-maz   fsx.openzfs.csi.aws.com   Delete          Immediate              true                   11m
...
:::

::::

### Step 4. Create the persistent volume claim (PVC)

In this step you will create the persistent volume claim for the defined storageclass.

1. Run the below command to see the persistent volume claim definition.

::code[cat fsxZ-storage-claim.yaml]{language=bash showLineNumbers=false showCopyAction=true}

**Sample output**
:::code[]{language=yaml showLineNumbers=false showCopyAction=false}
#fsxZ-storage-claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: fsx-openzfs-pvc-maz
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: fsx-openzfs-sc-maz
  resources:
    requests:
      storage: 1200Gi
:::

::alert[Observe the `fsxZ-storage-claim.yaml`, which is referring to the `fsx-openzfs-sc` storage class. You will use this file to provision the FSx for OpenZFS storage. you are configuring the 1200Gi PVC.]

2. Deploy the pvc.

Copy and run the below command to create the pvc.

::code[kubectl apply -f fsxZ-storage-claim.yaml]{language=bash showLineNumbers=false showCopyAction=true}


::alert["This process could take anywhere between to 5 to 10 minutes to completed. While you are waiting, you could quickly look at the status of the file system]

3. To check the status of the pvc from the cli with the below command.

::code[kubectl describe pvc/fsx-openzfs-pvc-maz]{language=bash showLineNumbers=false showCopyAction=true}

Note : Wait until you see last message like `Successfully provisioned volume ...` as below. This may take anywhere between to 5 to 10 minutes to complete.

::::expand{header="If you see the below output, it means that your volume creation is not expected to have issues. Otherwise, there might be problems that you need to troubleshoot, click to expand"}

:::code[]{language=yaml showLineNumbers=false showCopyAction=false}
Name:          fsx-openzfs-pvc-maz
Namespace:     default
StorageClass:  fsx-openzfs-sc-maz
Status:        Bound
Volume:        pvc-67cbe565-a1a2-462c-8320-9877c81ff5be
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: fsx.openzfs.csi.aws.com
               volume.kubernetes.io/storage-provisioner: fsx.openzfs.csi.aws.com
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1200Gi
Access Modes:  RWX
VolumeMode:    Filesystem
Used By:       fsx-openzfs-app-maz
Events:
  Type     Reason                 Age                    From                                                                                                      Message
  ----     ------                 ----                   ----                                                                                                      -------
  Warning  ProvisioningFailed     10m                    persistentvolume-controller                                                                               storageclass.storage.k8s.io "fsx-openzfs-sc-maz" not found
  Warning  ProvisioningFailed     4m50s                  fsx.openzfs.csi.aws.com_fsx-openzfs-csi-controller-7cc689b9b7-2fhm6_7ef8b8a6-1b2f-4ed9-b7de-b114722287b9  failed to provision volume with StorageClass "fsx-openzfs-sc-maz": rpc error: code = DeadlineExceeded desc = context deadline exceeded
  Warning  ProvisioningFailed     4m49s (x2 over 4m50s)  fsx.openzfs.csi.aws.com_fsx-openzfs-csi-controller-7cc689b9b7-2fhm6_7ef8b8a6-1b2f-4ed9-b7de-b114722287b9  failed to provision volume with StorageClass "fsx-openzfs-sc-maz": rpc error: code = Aborted desc = CreateVolume: An operation with the given volume pvc-67cbe565-a1a2-462c-8320-9877c81ff5be already exists
  Normal   Provisioning           4m45s (x4 over 9m50s)  fsx.openzfs.csi.aws.com_fsx-openzfs-csi-controller-7cc689b9b7-2fhm6_7ef8b8a6-1b2f-4ed9-b7de-b114722287b9  External provisioner is provisioning volume for claim "default/fsx-openzfs-pvc-maz"
  Normal   ExternalProvisioning   4m5s (x25 over 9m50s)  persistentvolume-controller                                                                               waiting for a volume to be created, either by external provisioner "fsx.openzfs.csi.aws.com" or manually created by system administrator
  Normal   ProvisioningSucceeded  59s                    fsx.openzfs.csi.aws.com_fsx-openzfs-csi-controller-7cc689b9b7-2fhm6_7ef8b8a6-1b2f-4ed9-b7de-b114722287b9  Successfully provisioned volume pvc-67cbe565-a1a2-462c-8320-9877c81ff5be
:::

::::

4. You can also check the status of the Amazon FSx for OpenZFS filesystem from the AWS Console. Click the [link](https://console.aws.amazon.com/fsx/)

### Step 5: Confirm that the file system is provisioned

Copy and run the below command to check the status of the pvc to confirm the status is "Bound".

::code[kubectl get pvc]{language=bash showLineNumbers=false showCopyAction=true}

**Sample output**

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
NAME                       STATUS     VOLUME                                   CAPACITY   ACCESS MODES   STORAGECLASS         AGE
fsx-openzfs-pvc-maz        Bound    pvc-67cbe565-a1a2-462c-8320-9877c81ff5be   1200Gi     RWX            fsx-openzfs-sc-maz   16m
...
:::

::alert[The STATUS may show as Pending for anywhere between to 5 to 10 minutes, before changing to Bound. Don’t continue with the next step until the STATUS is Bound.]{header="Note:" type="info"}

Once it is successfully created, you can see a new FSx for OpenZFS is displayed from [AWS FSx console](https://console.aws.amazon.com/fsx/) as below screenshot.

![FSxZ_01](/static/images/FSxZ_01.png)

### Step 6: Create pod

Copy and run the below command to apply and create the pod.

::code[kubectl apply -f pod-fsxZ-filesystem.yaml]{language=bash showLineNumbers=false showCopyAction=true}

**Sample output**
::code[pod/fsx-openzfs-app created]{language=bash showLineNumbers=false showCopyAction=false}

Check pod status to see it running.

::code[kubectl get pod]{language=bash showLineNumbers=false showCopyAction=true}

**Sample output**
:::code[]{language=bash showLineNumbers=false showCopyAction=false}
NAME              READY   STATUS    RESTARTS   AGE
fsx-openzfs-app   1/1     Running   0          48s
:::

## Summary

In this section you have successfully completed defining the storage class for Amazon FSx for OpenZFS file system with the deployment type. Created the storageclass and created the persistent volume claim which deployed the FSx for OpenZFS filesystem with auto provisioning of volume. We also created pod to mound our OpenZFS inside pod. In the next section you will continue with the static provisioning.
