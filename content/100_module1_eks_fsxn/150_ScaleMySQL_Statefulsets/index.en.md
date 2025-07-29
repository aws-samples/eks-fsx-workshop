---
title : "Send Database Traffic and Scale the Stateful Applications"
weight : 150
---
-------------------------------------------------------------

In this section, you will perform read/write database operations and observe how the storage and application can scale in/out seamlessly.

## Install MySQL Client and Create New Database and Table

### 1. Create MYSQL client for MYSQL

In order to perform some database operations, you need to perform some actions from the MySQL client.

- Go to the right working directory.

::code[cd /home/participant/environment/fsx-workshop-on-eks/static/eks/FSxN/mysql]{language=bash showLineNumbers=false showCopyAction=true}


- Deploy the MySQL client
::code[kubectl apply -f mysql-client.yaml]{language=bash showLineNumbers=false showCopyAction=true}

- Verify the pod getting deployed

::code[kubectl get pods -n mysql]{language=bash showLineNumbers=false showCopyAction=true}

::::expand{header="You should see the results as below, click to expand"}

- Note that you have 3 MySQL replicas (1 primary read-write, 2 read-only replicas)

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
NAME           READY   STATUS    RESTARTS   AGE
mysql-0        2/2     Running   0          161m
mysql-1        2/2     Running   0          160m
mysql-2        2/2     Running   0          160m
mysql-client   1/1     Running   0          25h
:::

::::

### 2. Logon to the MySQL client pod

::code[kubectl exec -it mysql-client -n mysql -- sh]{language=bash showLineNumbers=false showCopyAction=true}

### 3. Create the database

You can send test queries to the primary MySQL server (hostname mysql-0.mysql) on the MySQL Client.

:::code[]{language=sql showLineNumbers=true showCopyAction=true}
mysql -h mysql-0.mysql <<EOF
CREATE DATABASE erp;
CREATE TABLE erp.Persons ( ID int, FirstName varchar(255),Lastname varchar(255));
INSERT INTO erp.Persons (ID, FirstName, LastName) values (1234 , "John" , "Doe");
commit;
EOF
:::

### 4. Query the table just created

Use the hostname `mysql-read` to send test queries to any server that reports being Ready. Alternatively, you can send the read-query to primary MySQL server (hostname `mysql-0.mysql`), or read-only replicas (`mysql-1.mysql`, `mysql-2.mysql`).

::code[mysql -h mysql-0.mysql -e "SELECT * FROM erp.Persons;"]{language=sql showLineNumbers=false showCopyAction=true}

::alert[Feel free to replace `mysql-0.mysql` to `mysql-1.mysql` or `mysql-2.mysql`]

::::expand{header="You should see the results as below, click to expand"}

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
+------+-----------+----------+
| ID   | FirstName | Lastname |
+------+-----------+----------+
| 1234 | John      | Doe      |
+------+-----------+----------+
:::

::::

### 5. Demonstrate MySQL-Read Service distributes connections across servers

- To demonstrate that the mysql-read Service distributes connections across servers, you can run SELECT @@server_id in a loop:

::code[while sleep 1; do mysql -h mysql-read -e 'SELECT @@server_id,NOW()'; done]{language=bash showLineNumbers=false showCopyAction=true}


::::expand{header="You should see the reported @@server_id change randomly, because a different endpoint might be selected upon each connection attempt, click to expand"}

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         100 | 2023-09-27 07:54:42 |
+-------------+---------------------+
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         100 | 2023-09-27 07:54:43 |
+-------------+---------------------+
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         102 | 2023-09-27 07:54:44 |
+-------------+---------------------+
^C
:::

::::

You can press Ctrl+C when you want to stop the loop, but it's useful to keep it running in another window so you can see the effects.

Type "exit" to exit the container

::code[exit]{language=bash showLineNumbers=false showCopyAction=true}

## Scale out and scale in your MySQL applications

- When you use MySQL replication, you can scale your read query capacity by adding replicas. For a StatefulSet, you can achieve this with a single command (On another terminal window if possible while keeping the previous @@server_id loop in the previous terminal window):

::code[kubectl scale statefulset mysql -n mysql --replicas=5]{language=bash showLineNumbers=false showCopyAction=true}

- Verify how many pods are running of MySQL

::code[kubectl get pods -n mysql]{language=bash showLineNumbers=false showCopyAction=true}

- Once they're up, you should see server IDs 103 and 104 start appearing in the SELECT @@server_id loop output.

- You will also find out that PVCs and storage volumes are created instantly.

::code[kubectl get pvc -n mysql -l app=mysql]{language=bash showLineNumbers=false showCopyAction=true}

- Scaling back down is also seamless

::code[kubectl scale statefulset mysql -n mysql --replicas=3]{language=bash showLineNumbers=false showCopyAction=true}

::alert[Note: Although scaling up creates new PersistentVolumeClaims automatically, scaling down does not automatically delete these PVCs.This gives you the choice to keep those initialized PVCs around to make scaling back up quicker, or to extract data before deleting them.]

- You can see this by running:

Pods have been terminated or being terminated

::code[kubectl get pods -n mysql]{language=bash showLineNumbers=false showCopyAction=true}

Repeat it for a few times until you see only 3 pods are running

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
NAME           READY   STATUS    RESTARTS   AGE
mysql-0        2/2     Running   0          169m
mysql-1        2/2     Running   0          169m
mysql-2        2/2     Running   0          168m
mysql-client   1/1     Running   0          25h
:::

Run this commands to see how many PVCs still exists even after pods have been successfully destroyed.

::code[kubectl get pvc -l app=mysql -n mysql]{language=bash showLineNumbers=false showCopyAction=true}

::::expand{header="It shows that all 5 PVCs still exist, despite having scaled the StatefulSet down to 3, click to expand"}

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-mysql-0   Bound    pvc-fe74fc64-fcc3-44bb-8304-6dd2f3153255   30Gi       RWX            trident-csi    25h
data-mysql-1   Bound    pvc-7c69aab2-525a-425b-a784-0d3481259c77   30Gi       RWX            trident-csi    25h
data-mysql-2   Bound    pvc-6e88990d-87c2-4af4-a41a-e4f26215ee7a   30Gi       RWX            trident-csi    24h
data-mysql-3   Bound    pvc-0a1778ee-f86e-447a-b46c-8a6216c82380   30Gi       RWX            trident-csi    24h
data-mysql-4   Bound    pvc-d27610ba-c68f-4e73-8fd1-effbef59447d   30Gi       RWX            trident-csi    6m14s
:::

::::


## Summary

In this section, you managed to create database and table in the MySQL database, perform database read across multiple read-only replicas, and scale out and scale in the database. In addition, you also see how the storage volumes works in the scale out and scale in scenarios. Next section, you will get a chance to look at the backup and restore of the persistent volumes using either NetApp FlexClone feature (ONTAP Method) or Kubernetes VolumeSnapshot (Kubernetes Method) to backup and restore data.
