---
title : "Amazon EKS Backup and Restore on FSx for NetApp ONTAP"
weight : 160
---
-------------------------------------------------------------

In this section, you will have the opportunities to perform backup and restore of FSx for NetApp ONTAP volume on Amazon EKS.


## Step 1 Create the pvc and pod for our testing

- Go to the right working directory.

::code[cd /home/participant/environment/fsx-workshop-on-eks/static/eks/FSxN/mysql]{language=bash showLineNumbers=false showCopyAction=true}

:::code[]{language=bash showLineNumbers=false showCopyAction=true}
kubectl apply -f pod_backup_restore.yaml
:::

:::code[]{language=bash showLineNumbers=false showCopyAction=true}
kubectl get pvc
:::

PVC is successfully created, please write down the PVC name which you will use in the next step.

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
basic                Bound    pvc-c910187f-582f-44ac-a374-a73232d2b0a5   10Gi       RWX            trident-csi    45m
pvc-backup-restore   Bound    pvc-f7af51ea-aa81-464a-928c-4081739edf67   1Gi        RWO            trident-csi    21m
:::

Note your PVC volume associated with `pvc-backup-restore` for example `pvc-f7af51ea-aa81-464a-928c-4081739edf67` in this case above.

Pod is successfully created.

::code[kubectl get pods]{language=bash showLineNumbers=false showCopyAction=true}

Pod is successfully created.

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
NAME                              READY   STATUS    RESTARTS   AGE
backup-restore-7cbb4ff56c-dl5rj   1/1     Running   0          2m20s
:::

Have a look at the `pod-backup-restore.yaml`, the storage is mounted on `/usr/share/trident-nas/`

:::code[]{language=yaml showLineNumbers=false showCopyAction=false}
...
      volumeMounts:
        - mountPath: "/usr/share/trident-nas/"
          name: backup-restore
:::

## Step 2: Logon to the container to write data for the backup point

Replace pod name from previous output.

::code[kubectl exec -it [pod name] -- bash]{language=bash showLineNumbers=false showCopyAction=true}

Write the backup data to `/usr/share/trident-nas/backup.txt` where the FSx for NetApp Volume is mounted.

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
echo "backup point01" > /usr/share/trident-nas/backup.txt
cat /usr/share/trident-nas/backup.txt
:::

We can see the file is created.

:::code[]{language=bash showLineNumbers=true showCopyAction=false}
root@backup-restore-pod:/# cat /usr/share/trident-nas/backup.txt
backup point01
:::

Exit from the container

::code[exit]{language=bash showLineNumbers=false showCopyAction=true}

There are two ways of backup and restore. One is via the FSx for NetApp ONTAP Snapshot feature, and the other is via the Kubernetes VolumeSnapshot feature. You are welcome to choose either one to work on in this workshop.


::alert[Note: Steps 1 and 2 will be that same for the SnapShot exercise. If you plan to complete both, you may proceed to Step 3 bewlow in this exercise for each option.]


::::tabs{variant="container" groupId="backupandrestore" activeTabId="snapshot"}
:::tab{id="snapshot" label="SnapShot [FSx for NetApp ONTAP method]"}

Amazon FSx for NetApp ONTAP supports NetApp’s SnapShot feature, enabling you to create a clone of the volumes in your file system instantaneously with the click of a button. A Snapshot is a point-in-time, writable copy of its parent volume that shares data blocks with its parent, which means the snapshot consumes no storage for data shared with its parent, taking up minimal incremental space in your file system.


## Step 3: Log on to the FSx for NetApp ONTAP SVM (Storage Virtual Machine) and create snapshot for backup point

- If you are using Cloud9 or Windows, open up a new terminal. If you are using Linux, launch a new Session Manager session or open a new terminal from your computer to SSH into the EC2 instance.

- From your new terminal or new session manager session, SSH into the SVM of FSx for NetApp ONTAP using below command. The IP/Hostname of the SVM as shown in AWS Console, as below. You can use **Management DNS Name** or its corresponding IP address like `198.19.X.X` to use for the `SVMIP` below.

SSH into the SVM of FSx for NetApp ONTAP.

```bash
ssh vsadmin@SVMIP
Password: [your password from secrets manager]
```

- Show the Volume

::code[vol show]{language=bash showLineNumbers=false showCopyAction=true}

You should be able to find the volume ID that matches with your PVC, note down the volume name.

::alert[Note: You can use **Tab** key to auto-complete the ONTAP cli command]

```
SVM1::> vol show
Vserver   Volume       Aggregate    State      Type       Size  Available Used%
--------- ------------ ------------ ---------- ---- ---------- ---------- -----
SVM1      SVM1_root    aggr1        online     RW          1GB    972.2MB    0%
SVM1      trident_pvc_137803cc_8050_49c9_bd22_562f4e5b62be 
                       aggr1        online     RW         30GB    29.98GB    0%
(...)
SVM1      trident_pvc_f7af51ea_aa81_464a_928c_4081739edf67 
                       aggr1        online     RW          1GB     1023MB    0%
8 entries were displayed.
```

- Create the FSx for NetApp ONTAP volume Snapshot for Backup Point, replace the "[volume-name]" with your volume name which is PVC volume associated with `pvc-backup-restore`.

::alert[The volume name should be matching with your PVC name which is mentioned above]

::code[snap create -volume [volume-name] -snapshot backup_point]{language=bash showLineNumbers=false showCopyAction=true}



- Show the snapshot created

::code[snap show -volume [volume-name]]{language=bash showLineNumbers=false showCopyAction=true}

You should see something as below:

```
SVM1::> snap show -volume trident_pvc_f7af51ea_aa81_464a_928c_4081739edf67
                                                                 ---Blocks---
Vserver  Volume   Snapshot                                  Size Total% Used%
-------- -------- ------------------------------------- -------- ------ -----
SVM1     trident_pvc_f7af51ea_aa81_464a_928c_4081739edf67
                  backup_point                             188KB     0%   46%
```

## Step 4: Delete the /usr/share/trident-nas/backup.txt from the container

- Now open another terminal, here we exec back to your container that you logged onto earlier, and delete the /usr/share/trident-nas/backup.txt created before. Replace pod name from the first command output. 


```bash
kubectl get pods
```

```bash
kubectl exec -it [pod name] -- bash
rm -f /usr/share/trident-nas/backup.txt
```

Check backup.txt is deleted

```bash
ls -ltr /usr/share/trident-nas/          
```

You should see output like : 

```      
total 0
```

## Step 5. Restore the snapshot backup_point from SVM

- Go to the FSx for NetApp ONTAP SVM SSH session, and restore the volume from the snapshot created.

::code[snap restore -volume [volume-name] -snapshot backup_point]{language=bash showLineNumbers=false showCopyAction=true}

- Type Y in both prompts, the volume has been restored.

```
SVM1::> snap restore -volume [volume-name] -snapshot backup_point

Warning: Quota rules currently enforced on volume "[volume-name]" might change
        during this operation. If the currently enforced quota rules are different from those in Snapshot copy
        "backup_point", you might have to resize or reinitialize quotas on this volume after this operation. 
Do you want to continue? {y|n}: y

Warning: Export policies currently enforced on the qtrees of volume
        "[volume-name]" will not change during this operation. If the currently enforced export policies are different from those in Snapshot copy "backup_point", reassign
        the export policies of the qtrees on this volume after this operation.
Do you want to continue? {y|n}: y
```

## Step 6. Validate the file restored in the container immediately

- Go back to your container that you logged on, and validate /usr/share/trident-nas/backup.txt is restored

```bash
ls -ltr /usr/share/trident-nas/backup.txt
cat /usr/share/trident-nas/backup.txt 
```

You should see the file is successfully restored with the correct contents.

```
root@backup-restore-pod:/# ls -ltr /usr/share/trident-nas/backup.txt
-rw-r--r-- 1 root root 15 Nov 19 05:53 /usr/share/trident-nas/backup.txt
root@backup-restore-pod:/# cat /usr/share/trident-nas/backup.txt 
backup point01
```

Type `exit` in both your terminal sessions to exit the POD client and also the SVM SSH session

::code[exit]{language=bash showLineNumbers=false showCopyAction=true}


:::

:::tab{id="volumesnapshot" label="VolumeSnapshot [Kubernetes native method]"}
As an alternative option to using the ONTAP CLI to create a Snapshot for backup and recovery of data in a pod, kubectl can be used to create a volumesnapshot as a native Kubernetes primitive that accesses the ONTAP API directly to snapshot the filesystem via the Trident CSI Driver.


## Step 3: Apply the Volume Snapshot CRDs and Volume Snapshot Controller

- Apply the CRDs (Custom Resources) for the Snapshot Controller. Custom resources are extensions of the Kubernetes API, for more information please refer to the [official page](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/release-5.0/client/config/crd/snapshot.storage.k8s.io_volumesnapshotclasses.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/release-5.0/client/config/crd/snapshot.storage.k8s.io_volumesnapshotcontents.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/release-5.0/client/config/crd/snapshot.storage.k8s.io_volumesnapshots.yaml
```

- Deploy the Snapshot Controller Pods

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/release-5.0/deploy/kubernetes/snapshot-controller/rbac-snapshot-controller.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/release-5.0/deploy/kubernetes/snapshot-controller/setup-snapshot-controller.yaml
```

- View the Snapshot Controller Pods

::code[kubectl get pods -n kube-system |grep snapshot-controller]{language=bash showLineNumbers=false showCopyAction=true}

You should see something like below:

```
NAME                                   READY   STATUS    RESTARTS   AGE
snapshot-controller-75fd799dc8-t2wzl   1/1     Running   0          121m
snapshot-controller-75fd799dc8-tg7km   1/1     Running   0          121m
```

## Step 4: Create the Volume Snapshot Class

```bash
kubectl apply -f snapshot-class.yaml
kubectl get VolumeSnapshotClass 
```

View the Volume Snapshot Class, should be something like below:

```
NAME                    DRIVER                  DELETIONPOLICY   AGE
trident-snapshotclass   csi.trident.netapp.io   Delete           91m
```

## Step 5: Create a Snapshot of the existing PVC "backup-restore"

::code[kubectl apply -f snapshot.yaml]{language=bash showLineNumbers=false showCopyAction=true}

View the new Snapshot

::code[kubectl get vs]{language=bash showLineNumbers=false showCopyAction=true}

You can see something as below

```
NAME         READYTOUSE   SOURCEPVC   SOURCESNAPSHOTCONTENT   RESTORESIZE   SNAPSHOTCLASS           SNAPSHOTCONTENT                                    CREATIONTIME   AGE
basic-snap   true         backup-restore                              328Ki         trident-snapshotclass   snapcontent-4615c2f3-2668-40a0-b1c4-ed554f2a2990   95m            95m
```

## Step 6: Delete the /usr/share/trident-nas/backup.txt from the container

- Log back to your container

::code[kubectl get pod]{language=bash showLineNumbers=false showCopyAction=true}

::code[kubectl exec -it [pod name] -- bash]{language=bash showLineNumbers=false showCopyAction=true}

- Delete the /usr/share/trident-nas/backup.txt created earlier.

```bash
rm -f /usr/share/trident-nas/backup.txt
ls -ltr /usr/share/trident-nas/
```

The backup.txt is deleted

```
root@backup-restore-pod:/# ls -ltr /usr/share/trident-nas/          
total 0
```

Exit from the container

::code[exit]{language=bash showLineNumbers=false showCopyAction=true}

## Step 7: Create a new PVC from the Snapshot

Perform the following steps to create the new PVC from the Snapshot

```bash
kubectl apply -f pvc-from-snap.yaml
kubectl get pvc
```

You should see the output like below:

```
NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
basic                Bound    pvc-c910187f-582f-44ac-a374-a73232d2b0a5   10Gi       RWX            trident-csi    60m
pvc-backup-restore   Bound    pvc-f7af51ea-aa81-464a-928c-4081739edf67   1Gi        RWO            trident-csi    36m
pvc-from-snap        Bound    pvc-e8d5b39f-1af5-457d-9676-58fe1444106b   1Gi        RWO            trident-csi    90s
```

## Step 8: Mount the new PVC created from the Snapshot to the container

::alert[Note: In Cloud9 you can follow the same instructions or you could edit the file directly from Cloud9 browser]

::code[vi pod_backup_restore.yaml]{language=bash showLineNumbers=false showCopyAction=true}

- Select “i” to go into edit mode
- Replace the **pvc-backup-restore** with **pvc-from-snap** ONLY in the last line of the file (not the beginning of the file, see screenshot below.)
- select ESC and type `:wq` then press enter.

![PVCRestore](/static/images/pvc_restore.png)

Deploy the new pod

::code[kubectl apply -f pod_backup_restore.yaml]{language=bash showLineNumbers=false showCopyAction=true}

You can see that a new pod is deployed replacing the previous pod from the output from the previous command, but PV (Persistent Volume) remains unchanged.

```
persistentvolumeclaim/pvc-backup-restore unchanged
deployment.apps/backup-restore configured
```

Verify the new pod being created, and write down the new pod name

::code[kubectl get pods]{language=bash showLineNumbers=false showCopyAction=true}

You are expected to see the output as below

```
NAME                              READY   STATUS    RESTARTS   AGE
backup-restore-YYYYYYY  1/1     Running   0          33s
```

## Step 9. Validate the file restored in the container immediately

Logon to the new container

::code[kubectl exec -it [pod name] -- bash]{language=bash showLineNumbers=false showCopyAction=true}

Run the below command in container

```bash
ls -ltr /usr/share/trident-nas/backup.txt
cat /usr/share/trident-nas/backup.txt 
```

You should see the file is successfully restored with the correct contents.

```
root@backup-restore-pod:/# ls -ltr /usr/share/trident-nas/backup.txt
-rw-r--r-- 1 root root 15 Nov 19 05:53 /usr/share/trident-nas/backup.txt
root@backup-restore-pod:/# cat /usr/share/trident-nas/backup.txt 
backup point01
```

Type `exit` in your terminal sessions to exit the POD client.

::code[exit]{language=bash showLineNumbers=false showCopyAction=true}

:::
::::

## Summary

In this section, you have successfully performed backup and restore of the data on the FSx for NetApp ONTAP SVM volume using ONTAP method Or using Kubernetes native method. In the next section, you will run performance tests with FSx for NetApp ONTAP on Amazon EKS when the Kubernetes pod is within the same Availability Zone and in a different Availability Zone.
