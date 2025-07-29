---
title : "Deploy the stateful application"
weight : 140
---
-------------------------------------------------------------

In this section, you will deploy a highly available MySQL cluster onto the Kubernetes cluster using a Kubernetes [Statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/). Kubernetes statefulset ensures the original PersistentVolume (PV) is mounted on the same pod identity when it’s rescheduled again to retain data integrity and consistency.

This application is a replicated MySQL database. The example topology has a single primary server for read-write and multiple read-only replicas, using asynchronous row-based replication. In this setup, all writes and updates must take place on the primary (source) server. Reads, however, may take place on one or more read replicas. For more information about the application replication configuration, please refer to [MySQL Official document](https://dev.mysql.com/doc/refman/8.0/en/replication.html).

::alert[Note: This is not a production configuration. MySQL settings remain on insecure defaults to keep the focus on general patterns for running stateful applications in Kubernetes.]

## 1.Create ConfigMap

Here, you use Kubernetes [ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/) to separate configurations and pods. In this example, you apply a ConfigMap named `mysql`. When the Primary and Secondary pods gets deployed, they read the corresponding configurations.


- Go to the right working directory.

::code[cd /home/participant/environment/fsx-workshop-on-eks/static/eks/FSxN/mysql]{language=bash showLineNumbers=false showCopyAction=true}



- Create a namespace for MySQL and the ConfigMap for MySQL

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
kubectl create ns mysql
kubectl create -f mysql-configmap.yaml -n mysql
:::


This ConfigMap provides `*.cnf` overrides that let you independently control configuration on the primary MySQL server and its replicas. In this case, you want the primary server to be able to serve replication logs to replicas and you want replicas to reject any writes that don't come via replication.

There's nothing special about the ConfigMap itself that causes different portions to apply to different Pods. Each Pod decides which portion to look at as it's initializing, based on information provided by the StatefulSet controller.

## 2.Create Kubernetes Services

Kubernetes [Service](https://kubernetes.io/docs/concepts/services-networking/service/) defines a logical set of pods and a policy by which to access them. StatefulSet currently requires a headless service to control the domain of its pods, directly reaching each pod with stable DNS entries. By specifying “None” for the clusterIP, you can create a headless service.

- Create mysql headless service.

::code[kubectl apply -f mysql-service.yaml]{language=bash showLineNumbers=false showCopyAction=true}

- Verify mysql headless service is created successfully.

::code[kubectl get service -n mysql]{language=bash showLineNumbers=false showCopyAction=true}

## 3.Deploy StatefulSet for MySQL

Next, you need to deploy the StatefulSet for MySQL. You may find that the mysql pod contains two init containers (`init-mysql` and `clone-mysql`) and two app containers (`mysql` and `xtrabackup`), and the pod will be bound to the persistent volume provided by FSx for NetApp ONTAP volumes via Trident CSI in the PersistentVolumeClaim.

:::alert{header="Note" type="info"}
For more information on how the statefulset sets up the MySQL clusters (1 primary server and multiple read-only replicas), please refer to the Kubernetes official document [Understanding stateful Pod initialization](https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/#understanding-stateful-pod-initialization)
:::

- Create the MySQL Statefulset

::code[kubectl apply -f mysql-statefulset.yaml]{language=bash showLineNumbers=false showCopyAction=true}

- Make sure 3 MySQL pods are running, then check the volume bounding

::alert[Kindly wait for 60 to 120 secs to get the pods in running status]{header="Note" type="info"}

::code[kubectl get pod -l app=mysql -n mysql]{language=bash showLineNumbers=false showCopyAction=true}

::::expand{header="You should see the results as below, click to expand"}

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
NAME      READY   STATUS    RESTARTS   AGE
mysql-0   2/2     Running   0          2m21s
mysql-1   2/2     Running   0          89s
mysql-2   2/2     Running   0          42s
:::

::::

- Let's have a look at the `mysql-statefulset.yaml`, particularly the storage volume part.

:::code[]{language=yaml showLineNumbers=false showCopyAction=false}
...
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteMany"]
      storageClassName: trident-csi
      resources:
        requests:
          storage: 30Gi
:::

You can see that the storage volume of 30GB has been deployed with Storage Class `trident-csi`. Thus, you will expect 30GB of volume will be provisioned on FSx for NetApp ONTAP SVM.

- You can confirm `data-mysql-0`, `data-mysql-1` and `data-mysql-2` have persistent volumes mounted.

::code[kubectl get pv -n mysql]{language=bash showLineNumbers=false showCopyAction=true}

::::expand{header="You should see the results as below, click to expand"}

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                STORAGECLASS   REASON   AGE
pvc-137803cc-8050-49c9-bd22-562f4e5b62be   30Gi       RWX            Retain           Bound    mysql/data-mysql-2   trident-csi             2m14s
pvc-6117eeea-14f1-44fd-ad61-ff6f19ca0608   30Gi       RWX            Retain           Bound    mysql/data-mysql-0   trident-csi             4m
pvc-c910187f-582f-44ac-a374-a73232d2b0a5   10Gi       RWX            Retain           Bound    default/basic        trident-csi             9m19s
pvc-e6b502c3-4fb3-498c-b229-d182b6a0453c   30Gi       RWX            Retain           Bound    mysql/data-mysql-1   trident-csi             3m6s
:::

::::

- Let’s pay attention that mapping between the pod and PersistentVolume, and open up notepad per your choice to write down the mapping and PVC name.

`data-mysql-0` → `pvc-6117eeea-14f1-44fd-ad61-ff6f19ca0608`

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

Last login time: 10/3/2023 05:30:11
SVM1::> vol show
Vserver   Volume       Aggregate    State      Type       Size  Available Used%
--------- ------------ ------------ ---------- ---- ---------- ---------- -----
SVM1      SVM1_root    aggr1        online     RW          1GB    972.4MB    0%
SVM1      trident_pvc_137803cc_8050_49c9_bd22_562f4e5b62be
                       aggr1        online     RW         30GB    29.98GB    0%
SVM1      trident_pvc_6117eeea_14f1_44fd_ad61_ff6f19ca0608
                       aggr1        online     RW         30GB    29.97GB    0%
SVM1      trident_pvc_c910187f_582f_44ac_a374_a73232d2b0a5
                       aggr1        online     RW         10GB    10.00GB    0%
SVM1      trident_pvc_e6b502c3_4fb3_498c_b229_d182b6a0453c
                       aggr1        online     RW         30GB    29.98GB    0%
5 entries were displayed.
:::

- You can see the Volume name matches with your pvc name

::::

## Summary

Congratulations! In this section, you have successfully deployed the MySQL clusters with PV mounted to the pod, and volume automatically provisioned by Trident CSI on your FSx for NetApp ONTAP SVM. In the Next section, you are going to perform read/write operations on the MySQL Database and scale your MySQL database.
