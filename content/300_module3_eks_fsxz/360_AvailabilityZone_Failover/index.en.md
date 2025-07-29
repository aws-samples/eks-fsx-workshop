---
title : "Multi-AZ FSx for OpenZFS file system fail over across Availability Zones - Optional"
weight : 190
---
-------------------------------------------------------------

Business-critical production workloads normally require high availability across Availability Zones. Availability Zones consist of one or more discrete data centers, each with redundant power, networking, and connectivity, housed in separate facilities.  

In this section, you will simulate an AZ failure while constantly performing a FIO write operation for 45 minutes to the multi-AZ FSx for OpenZFS file system. During the process you can monitor if the write requests are interrupted during the Availability Zone failover.

In **[Last Section](/100_module1_eks_fsxn/180_PerformanceTesting)**, you deployed an pod named `task-pv-pod-saz`, we will logon to the container

### 1. Log in to the container and use FIO to write to the FSx for OpenZFS volume for 45 minutes

::code[kubectl exec -it task-pv-pod-saz -- bash]{language=bash showLineNumbers=false showCopyAction=true}

- Use FIO command to continue write for 45 minutes.

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
cd /usr/share/trident-nas/
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=fiotest --filename=testfio --bs=4k --iodepth=64 --size=8G --readwrite=randrw --rwmixread=75 --time_based=1 --runtime=2700s
:::

### 2. Navigate back to FSx for OpenZFS console, take notes of the network interface for both Preferred Subnet and Standby Subnet.

- Navigate to the [FSx console](https://console.aws.amazon.com/fsx/)
- Ensure you are in the current operating region of the lab
- From the left hand pane select File Systems → Then select the *file system ID* of your file system → Select the Network & Security tab
- As shown below, take note of the *“Network Interface”* value shown for the Preferred Subnet and also the Standby Subnet, in a notepad file.  During failover, you will see the switching between the preferred subnet ENI to Standby subnet ENI.

![Diagram](/static/images/190-1.png)

### 3. Click the Route table that associated to the FSx for OpenZFS file system and confirm traffic is being routed to the network interface for the the preferred subnet

- on the Network & Security tab, click on the routable associated to your FSxInstance

:::alert{header="Important" type="info"}
Notice that traffic to the 198.19.255.x address of your FSx Instance is being routed through the ENI that is associated with the “Preferred Subnet“ in your FSx configuration (which you noted down earlier) Multi-AZ FSx Instance failover, the traffic will automatically be routed to the ENI associated with the standby subnet, and then back to the ENI associated with the preferred subnet when the FSx Instances fails back.
:::

![Diagram](/static/images/190-2.png)

As shown in the image the traffic is routed through the preferred subnet ENI
 
Lets go ahead now and simulate a failover and failback by performing an FSx online throughput capacity upgrade

### 4. Modify FSx for OpenZFS filesystem throughput Capacity to 256MB/s

![Diagram](/static/images/190-3.png)

- On the FSx Console, In the *Summary* section select the *Update* button associated with the *Throughput Capacity* value
- Select 256 MB/s and *Update*


### 5. Check the route table again, now we see that the traffic is routed to the network interface for the standby subnet

![Diagram](/static/images/190-5.png)

You can see update in progress on the console : 

![Diagram](/static/images/190-4.png)

Also you can monitor the status of FSx Instance configuration updates using the → update tab

![Diagram](/static/images/fsx-status-update.png)

### 6. Review the FIO command output after Availability Zone failover and fail back. You can see that FIO tool keeps writing the file system without any errors. Failover and fail back are transparent to the file system writing test

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
root@task-pv-pod:/usr/share/trident-nas/# fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=fiotest --filename=testfio --bs=4k --iodepth=64 --size=8G --readwrite=randrw --rwmixread=75 --time_based=1 --runtime=2700s
Starting 1 process
fiotest: Laying out IO file (1 file / 8192MiB)
Jobs: 1 (f=1): [m(1)][100.0%][r=158MiB/s,w=53=53.1MiB/s][r=40.4k,w=13.6k IOPS][eta 00m:00s]
fiotest: (groupid=0,jobs=1): err=0: pid=730: Sun Nov 14 15:44:28 2022
...
:::

## Summary

Congratulations! you have successfully completed this section. You are able to simulate FSx for OpenZFS system failover and fail back, and you could verify that the file system failover did not cause any IO errors. This showcases the FSx for OpenZFS Multi-AZ capabilities on Amazon EKS. Amazon FSx for OpenZFS in a multi-AZ deployment type provides storage layer replication and data is available for the hosts even during a failover scenario.
