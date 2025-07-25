---
title : "Provision Trident Volumes and Storage Class"
weight : 130
---
-------------------------------------------------------------

In this section, you are going to create the Trident Backend, storage class and then create a PVC (Persistent Volume Claim) to provision your storage volume in Amazon EKS cluster.

## 1. Retrieve Kubernetes Secrets of the Storage Virtual Machine (SVM) username and password

::alert[The SVM username and its admin password has been created and saved in the AWS Secrets Manager. Please retrieve password from the AWS Secrets Manager.]

Navigate to the [AWS Secrets Manager console](https://console.aws.amazon.com/secretsmanager/listsecrets), Click on the value of *-SVMAdminPassword.

![Secrets_Manager_Screenshot](/static/images/Secrets_01.png)

::alert[For the Self-Pace lab, the secrets name will be `SVMAdminPassword`]


* Retrieve the secret password from secret manager
  * Scroll down to secret value section
  * Select `Retireve secret value` button
    ![Secret01](/static/images/Secret_03.png)
  * copy the "Secret value" against the "Secret key" password
    ![Secret01](/static/images/Secret_02.png)
  * Make a note of the **password** in a **Notepad/Text Editor** for later use

- Navigate to the **svm_secret.yaml** file stored in the **eks/FSxN** folder of **fsx-workshop-on-eks** folder in your environment.


::code[cd /home/participant/environment/fsx-workshop-on-eks/static/eks/FSxN]{language=bash showLineNumbers=false showCopyAction=true}


::alert[Note: In Cloud9 you can follow the same instructions or you could edit the file directly from Cloud9 browser]

::code[vi svm_secret.yaml]{language=bash showLineNumbers=false showCopyAction=true}

- Update the password in the file from the credentials you noted from the AWS Secrets Manager

* press “i” to go into edit mode
* Replace the field `password` with the credentials you retrieved from Secret Manager
* Press ESC and type `:wq` then press enter.


![Secrets01](/static/images/Secrets_01.png)


- Create the Kubernetes secrets.

::code[kubectl apply -f svm_secret.yaml]{language=bash showLineNumbers=false showCopyAction=true}

- Check the Secret has been created.

::code[kubectl get secrets -n trident | grep backend-fsx-ontap-nas-secret]{language=bash showLineNumbers=false showCopyAction=true}

::::expand{header="You should see the results as below, click to expand"}

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
backend-fsx-ontap-nas-secret    Opaque                                2      9s
:::

::::

## 2. Create the Trident Backend

When configuring the backend configuration for Trident, there are five drivers available for FSx for NetApp ONTAP backends as shown in the following table. These drivers are differentiated by the protocol being used and how the volumes are provisioned on the storage system. Refer to the [Trident documentation](https://netapp-trident.readthedocs.io/en/latest/dag/kubernetes/integrating_trident.html) for more details when considering which one to use based on your application requirements.

| Driver              | Description                                                                                          | Compatible with FSxONTAP? |
|---------------------|------------------------------------------------------------------------------------------------------|:-------------------------:|
| ontap-nas           | Each PV provisioned is a full ONTAP FlexVolume                                                       |            Yes            |
| ontap-nas-economy   | Each PV provisioned is a qtree, with a configurable number of qtrees per FlexVolume (default is 200) |            Yes            |
| ontap-nas-flexgroup | Each PV provisioned as a full ONTAP FlexGroup, and all aggregates assigned to a SVM are used         |            Yes            |
| ontap-san           | Each PV provisioned is a LUN within its own FlexVolume.                                              |            Yes            |
| ontap-san-economy   | Each PV provisioned is a LUN, with a configurable number of LUNs per FlexVolume (default is 100)     |            Yes            |

In this workshop, you will choose the `ontap-nas` driver that supports multi-attach, meaning the volume provisioned could be mounted by multiple pods simultaneously; and this allows the volume to be re-mounted by rescheduled pod of a StatefulSet due to a network partition, worker node failure or unexpected node draining in the Kubernetes data plane. Go back to your terminal where we are operating under `fsx-workshop-on-eks/eks/FSxN` folder, you will see `backend-ontap-nas.yaml` file. 




::alert[`Management IP Address` and `NFS IP Address` can be found via the Amazon FSx Console as shown in the below highlighted as `Management IP Address` which are likely to be the same.]

- Click on [AWS FSx Console](https://console.aws.amazon.com/fsx/), 
- Kindly ensure that the region selected in the AWS Console is your lab region name
- From the left hand pane select ONTAP → Storage Virtual machines → click on your SVM name
- Navigate to the Endpoints section 
- We need to update the **managementLIF** and **dataLIF** in `backend-ontap-nas.yaml`  file with the **Management IP Address** and **NFS IP Address** shown in below screenshot.

![SVM01](/static/images/SVM_01.png)

::code[vi backend-ontap-nas.yaml]{language=bash showLineNumbers=false showCopyAction=true}

* Press “i” to go into edit mode
* change the “managementLIF” with the IP Address copied from console above
* change the “DataLIF” with the IP Address copied from console above
* Press ESC and type `:wq` then press enter.


::::expand{header="Below is an explanation on relevant backend configuration parameters click to expand"}


| Parameter     | Description                | Remarks                                                              |
|---------------|----------------------------|----------------------------------------------------------------------|
| backendName   | Custom name for the storage backend                                                              ||
| managementLIF | IP address or FQDN of a cluster or SVM management LIF                                            ||
| dataLIF       | IP address of protocol LIF | When choosing ontap-san driver for backend, dataLIF could be skipped |
| SVM           | Storage virtual machine to use                                                                   ||

::::

- Create the Trident Backend

::code[kubectl apply -f backend-ontap-nas.yaml]{language=bash showLineNumbers=false showCopyAction=true}

- Verify the Trident Backend is deployed successfully.

::code[kubectl get tbc -n trident]{language=bash showLineNumbers=false showCopyAction=true}

::::expand{header="You should see the results as below, click to expand"}

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
NAME                   BACKEND NAME  BACKEND UUID                          PHASE  STATUS
backend-fsx-ontap-nas  fsx-ontap     6329459a-55e9-4606-881d-f83e34f558db  Bound  Success
:::

::::

## 3. Create Storage Class

The storage class yaml manifest is located as `storage-class-csi-nas.yaml`（also in the same `/fsx-workshop-on-eks/eks/FSxN` folder). Note that the `provisioner` is pointing to the CSI driver Trident created before.

::::expand{header="If you are interested see the file, click below"}

:::code[]{language=yaml showLineNumbers=true showCopyAction=false}
# storage-class-csi-nas.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: trident-csi
provisioner: csi.trident.netapp.io
parameters:
  backendType: "ontap-nas"
  fsType: "ext4"
allowVolumeExpansion: True
reclaimPolicy: Retain
:::

::::

- Deploy the storage class

::code[kubectl apply -f storage-class-csi-nas.yaml]{language=bash showLineNumbers=false showCopyAction=true}

- Check the status of the storage class

::code[kubectl get sc]{language=bash showLineNumbers=false showCopyAction=true}

::::expand{header="You should see the results as below, click to expand"}

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
gp2 (default)   kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   false                  4h58m
trident-csi     csi.trident.netapp.io   Retain          Immediate              true                   5s
:::

::::

## 4.Create Persistent Volume Claim (PVC)

Now you are creating a PVC of 10GB from `Storage Class` called `trident-csi` created in the previous step. The PVC is located as `pvc-trident.yaml` in the same `/fsx-workshop-on-eks/eks/FSxN` folder.

::::expand{header="If you are interested see the file, click below"}

:::code[]{language=yaml showLineNumbers=true showCopyAction=false}
# pvc-trident.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: basic
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: trident-csi
:::

::::


::code[kubectl create -f pvc-trident.yaml]{language=bash showLineNumbers=false showCopyAction=true}

- Verify that the persistent volume is created successfully and the PersistentClaim status is “Bound”

::code[kubectl get pv,pvc]{language=bash showLineNumbers=false showCopyAction=true}

::::expand{header="You should see the results as below, click to expand"}

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM           STORAGECLASS   REASON   AGE
persistentvolume/pvc-6ee33e4c-5356-40b5-9f2d-50a10fcbb0ba   10Gi       RWX            Retain           Bound    default/basic   trident-csi             68s

NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/basic   Bound    pvc-6ee33e4c-5356-40b5-9f2d-50a10fcbb0ba   10Gi       RWX            trident-csi    70s
:::

::::

- As the Trident CSI driver supports dynamic provisioning, after the PVC is submitted, PV will be automatically created and bounded, the external storage system (FSx for NetApp ONTAP) also automatically provisions the volume which matches the name of the PVC.
- **Optional:** When you navigate back to FSx for NetApp ONTAP console (after 30-45 minutes), select “Volumes” of your file system, then you can see the corresponding volume has been created.

::alert[Although it takes approximately 45 minutes to shown in the AWS console, the FSx for NetApp ONTAP volume is created almost instantly after PVC is submitted.]

![Volume01](/static/images/volume01.png)
![Volume02](/static/images/volume02.png)

::::expand{header="Given the volume takes some time to show up on the AWS Console, if you are interested in finding it from the ONTAP-cli, please expand the session."}

- Run the following SSH command and change the Management-IP-Address with the Management-IP-Address they noted from the FSxN console for the Storage Virtual Machine (SVM).
  * ssh vsadmin@`Management-IP-Address`
  * Enter the password you noted down for the SVM
  * type `vol show`
  * type `exit` to quit the ssh session with the FSxN SVM instance (you need to run this other wise they will remain logged into the ssh session and have issues on the next module)

![SVM01](/static/images/SVM_01.png)

You will expect to see the following output

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
ssh vsadmin@SVMIP
Password:

SVM1 > vol show
Vserver   Volume       Aggregate    State      Type       Size  Available Used%
--------- ------------ ------------ ---------- ---- ---------- ---------- -----
SVM1      trident_pvc_23d6b9f3_17ce_4845_a433_56b27a996435
                       aggr1        online     RW         10GB    9.98GB    0%
SVM1      
:::

- You can see the Volume name matches with your pvc name

::::

### Summary

In this section, You have sucessfully configured the trident operator and it enabled you to provision Kubernetes Persistent Volume Claim successfully. In the next section, you will deploy a stateful application that runs on Amazon EKS and have the Persistent Volume provisioned by Trident.
