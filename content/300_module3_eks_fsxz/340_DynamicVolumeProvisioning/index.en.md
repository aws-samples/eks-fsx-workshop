---
title : "Amazon FSx for OpenZFS Volume Dynamic Provisioning"
weight : 340
---
-------------------------------------------------------------

In this section you will define the storageclass variables and create the storageclass for Amazon FSx for OpenZFS Volume. Later you will create the persistent volume claim (PVC) and deploy the storage class. Observe the FSx for OpenZFS Volume is being auto provisioned. Notice here, we already have provisioned Amazon FSx for OpenZFS filesystem in previos section, here we are creating dyncamic volume which will be attached with our Amazon FSx for OpenZFS filesystem.


### Step 1: Prerequisite - setting the required environmental variable

Copy and paste the following lines in the CLI.

:::code[]{language=bash showLineNumbers=false showCopyAction=true}
FSX_VOLUME=$(aws fsx describe-volumes --query 'Volumes[?Name==`fsx`].VolumeId' --region $PRIMARY_REGION --output text)
:::

### Step 2: Define the storageclass

In the following steps you will be using the environment variables which was created in the previous section.

1. Run the below command to obtain the values for these variables. Copy these variables in your favorite editor app for later use in this module.

::code[echo $FSX_VOLUME]{language=bash showLineNumbers=false showCopyAction=true}

2. Go to the right working directory.

::code[cd /home/participant/environment/fsx-workshop-on-eks/static/eks/FSxZ/volume-maz]{language=bash showLineNumbers=false showCopyAction=true}

::code[cat fsxZ-volume-storage-class.yaml]{language=bash showLineNumbers=false showCopyAction=true}

We are going to replace `FSX_VOLUME` in `fsxZ-volume-storage-class.yaml` next.

::::expand{header="Click to expand the section to understand the settings defined in the fsxZ-volume-storage-class.yaml file"}
:::code[]{language=bash showLineNumbers=false showCopyAction=false}
#fsxZ-volume-storage-class.yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fsx-openzfs-vol-sc-maz
provisioner: fsx.openzfs.csi.aws.com
parameters:
  ResourceType: "volume" 
  ParentVolumeId: '"FSX_VOLUME"' 
  CopyTagsToSnapshots: 'false'
  DataCompressionType: '"NONE"'
  NfsExports: '[{"ClientConfigurations": [{"Clients": "*", "Options": ["rw","crossmnt"]}]}]'
  ReadOnly: 'false'
  RecordSizeKiB: '128'
  StorageCapacityReservationGiB: '-1'
  StorageCapacityQuotaGiB: '-1'
  UserAndGroupQuotas: '[{"Type": "USER", "Id": 1, "StorageCapacityQuotaGiB": 10}]'
  Tags: '[{"Key": "OPENZFS", "Value": "OPENZFS"}]'
  OptionsOnDeletion: '["DELETE_CHILD_VOLUMES_AND_SNAPSHOTS"]'
reclaimPolicy: Delete
allowVolumeExpansion: false #Not supported for OpenZFS volumes
mountOptions:
  - nfsvers=4.1
  - rsize=1048576
  - wsize=1048576
  - timeo=600
:::
::::

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
sed -i'' -e "s/FSX_VOLUME/$FSX_VOLUME/g" fsxZ-volume-storage-class.yaml
:::

Verify if all the input is saved by running the below command.

::code[cat fsxZ-volume-storage-class.yaml]{language=bash showLineNumbers=false showCopyAction=true}
 
::::expand{header="Click to expand the section to understand the settings defined in the fsxZ-volume-storage-class.yaml file"}

* **ParentVolumeId** – The ID of the volume to use as the parent volume of the volume that you are creating. Here Parent volume ID is our root volume for Amazon FSx for OpenZFS filesystem. 

* **StorageCapacityReservationGiB** - Specifies the amount of storage in gibibytes (GiB) to reserve from the parent volume. Setting StorageCapacityReservationGiB guarantees that the specified amount of storage space on the parent volume will always be available for the volume. You can't reserve more storage than the parent volume has. To not specify a storage capacity reservation, set this to 0 or -1.

* **StorageCapacityQuotaGiB** - Sets the maximum storage size in gibibytes (GiB) for the volume. You can specify a quota that is larger than the storage on the parent volume. A volume quota limits the amount of storage that the volume can consume to the configured amount, but does not guarantee the space will be available on the parent volume. To guarantee quota space, you must also set StorageCapacityReservationGiB. To not specify a storage capacity quota, set this to -1.

Check other Parameters detail [here](https://github.com/kubernetes-sigs/aws-fsx-openzfs-csi-driver/blob/main/docs/parameters.md#file-system-parameters) and [API reference](https://docs.aws.amazon.com/fsx/latest/APIReference/API_CreateOpenZFSVolumeConfiguration.html#API_CreateOpenZFSVolumeConfiguration_Contents)

::::


### Step 3: Create the storageclass

Copy and run the below command to apply the defined settings from the step 1. This will create the storageclass.

::code[kubectl apply -f fsxZ-volume-storage-class.yaml]{language=bash showLineNumbers=false showCopyAction=true}

Copy and run the below command to verify the storageclass is created.

::code[kubectl get sc]{language=bash showLineNumbers=false showCopyAction=true}

::::expand{header="You should see the results as below, click to expand"}

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
NAME                      PROVISIONER               RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
fsx-openzfs-vol-sc-maz    fsx.openzfs.csi.aws.com   Delete          Immediate              false                   5m
...
:::

::::

### Step 4. Create the persistent volume claim (PVC)

In this step you will create the persistent volume claim for the defined storageclass.

1. Run the below command to see the persistent volume claim definition.

::code[cat fsxZ-volume-claim.yaml]{language=bash showLineNumbers=false showCopyAction=true}


**Sample output**
:::code[]{language=yaml showLineNumbers=false showCopyAction=false}
#fsxZ-volume-claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: fsx-openzfs-vol-pvc-maz
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: fsx-openzfs-vol-sc-maz
  resources:
    requests:
      # When dynamically provisioning FSx for OpenZFS volumes, the requested storage value is ignored.
      # Instead, the CSI driver uses the storageCapacityReservation and storageCapacityQuota parameters to
      # configure the storage properties of the volume. These values are defined in the storage class.
      # The value below MUST be explicitly set to 1Gi, otherwise the creation will fail.
      storage: 1Gi
:::

::::alert[Observe the `fsxZ-volume-claim.yaml`, which is referring to the `fsx-openzfs-vol-sc-maz` storage class. You will use this file to provision the FSx for OpenZFS storage. you are configuring the 64Gi PVC.]
::::

2. Deploy the pvc.

Copy and run the below command to create pvc.

::code[kubectl apply -f fsxZ-volume-claim.yaml]{language=bash showLineNumbers=false showCopyAction=true}

::::alert["This process could take up to 5 mins minutes to completed. While you are waiting, you could quickly look at the status of the file system]
::::

3. To check the status of the pvc from the cli with the below command.

::code[kubectl describe pvc/fsx-openzfs-vol-pvc-maz]{language=bash showLineNumbers=false showCopyAction=true}

Note : Wait until you see last message like `Successfully provisioned volume ...` as below. This may take upto 2 mins.

::::expand{header="If you see the below output, it means that your volume creation is not expected to have issues. Otherwise, there might be problems that you need to troubleshoot, click to expand"}

:::code[]{language=yaml showLineNumbers=false showCopyAction=false}
Name:          fsx-openzfs-vol-pvc-maz
Namespace:     default
StorageClass:  fsx-openzfs-vol-sc-maz
Status:        Bound
Volume:        pvc-b073830a-398d-4b97-bbe7-620d50568656
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: fsx.openzfs.csi.aws.com
               volume.kubernetes.io/storage-provisioner: fsx.openzfs.csi.aws.com
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Gi
Access Modes:  RWX
VolumeMode:    Filesystem
Used By:       <none>
Events:        <none>
:::

::::

4. You can also check the status of the Amazon FSx for OpenZFS filesystem from the AWS Console. Click the [link](https://console.aws.amazon.com/fsx/volumes?#volumes)

### Step 5: Confirm that the file system is provisioned

Copy and run the below command to check the status of the pvc to confirm the status is "Bound".

::code[kubectl get pvc]{language=bash showLineNumbers=false showCopyAction=true}

**Sample output**

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
NAME                      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS              AGE
fsx-openzfs-vol-pvc-maz   Bound    pvc-06f82ea0-0755-4d11-8b3e-c3afc16b2237   1Gi        RWX            fsx-openzfs-vol-sc-maz   3m39s
...
:::

:::alert{header="Note:" type="info"}
The STATUS may show as Pending for up to 5 minutes, before changing to Bound. Don’t continue with the next step until the STATUS is Bound.
:::

Once it is successfully created, you can see a new volume for FSx for OpenZFS is displayed from [AWS FSx console](https://console.aws.amazon.com/fsx/) as below screenshot.

![FSxZ_01](/static/images/FSxZVol_01.png)


### Step 6: Create pod

Copy and run the below command to apply and create the pod.

::code[kubectl apply -f pod-fsxZ-vol.yaml]{language=bash showLineNumbers=false showCopyAction=true}

Check pod status to see it running.

::code[kubectl get pod]{language=bash showLineNumbers=false showCopyAction=true}

**Sample output**
:::code[]{language=bash showLineNumbers=false showCopyAction=false}
NAME                                     READY   STATUS    RESTARTS      AGE
fsx-openzfs-vol-app-maz                  1/1     Running   0             3s
...
:::

## Summary

In this section you have successfully completed defining the storage class for Amazon FSx for OpenZFS Volume. Created the storageclass and created the persistent volume claim which deployed the FSx for OpenZFS Volume with auto provisioning of Volume. We also created pod to mount our OpenZFS inside pod. In the next section you will continue with the performance testing.
