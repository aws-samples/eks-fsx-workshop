---
title : "Performance test with FIO and IOping"
weight : 180
---
-------------------------------------------------------------

In this section, you will look at important parameters related to storage performance, IOPS, throughput and latency. You will measure the performance of FSx for NetApp ONTAP file system provisioned by the Trident CSI in this example. You will use [FIO (Flexible I/O)](https://fio.readthedocs.io/en/latest/), the storage benchmarking tool, and [IOping](https://github.com/koct9i/ioping), a tool to monitor I/O latency in real time, to test the performances on FSx for NetApp ONTAP drive from the EKS pod.

Please fill in the performance of the same Availability Zone and different Availability Zones figures as you obtain them in this workshop.

|          Scenario           | Average IOPS (read) | Average IOPS (write) | Average throughput (read) | Average throughput (write) | Average latency |
|:---------------------------:|:-------------------:|:--------------------:|:-------------------------:|:--------------------------:|:---------------:|
|   Same Availability Zone    |                     |                      |                           |                            |                 |
| Different Availability Zone |                     |                      |                           |                            |                 |

## EKS pod and FSx for NetApp ONTAP in the same Availability Zone

As the FSx for NetApp ONTAP file system preferred subnet is in `ap-southeast-2a` in this test, you will deploy the EKS pod in the same Availability Zone to check the performance data.



:::alert{header="Important" type="info"}
Note: The preferred subnet Availability Zone - AZ changes based on the current lab region
:::

### Update the yaml file with your region subnet AZ with same AZ

- Go to the right working directory.
  
::code[cd /home/participant/environment/fsx-workshop-on-eks/static/eks/FSxN]{language=bash showLineNumbers=false showCopyAction=true}

- Update preferred Availability Zone
::code[vi pod_performance_same_AZ.yaml]{language=bash showLineNumbers=false showCopyAction=true}

* Press “i” to go into edit mode
* replace the topology.kubernetes.io/zone: `us-west-2a` to `ap-southeast-2a` as shown in the image. (If you are running this workshop in any other region then replace AZ accordingly)
* Press ESC and type `:wq` then press enter.

![saz-pod-az](/static/images/saz-pod-az.png)

### Deploy the yaml file to provision the pod and the 10 GB storage on FSx for NetApp ONTAP

::code[kubectl apply -f pod_performance_same_AZ.yaml]{language=bash showLineNumbers=false showCopyAction=true}

Wait for 1 minute

### Log in to the container and perform FIO and IOping testing

::code[kubectl exec -it task-pv-pod-saz  -- bash]{language=bash showLineNumbers=false showCopyAction=true}

- Install FIO and IOping, then perform the FIO command to perform load testing, and write down the IOPS and Throughput

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
apt-get update
apt-get install fio ioping -y
:::

:::alert{header="Important" type="info"}
Note: In the below FIO test, we are running a simulation of a load test using 4KB small block size, with 50% read/write mix, with a random read-write pattern using 4 concurrent jobs
:::

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
cd /usr/share/trident-nas/
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=fiotest --filename=testfio --bs=4k --iodepth=64 --size=8G --readwrite=randrw --rwmixread=50 --numjobs=4 --group_reporting --runtime=30
:::

::::alert{header="Important" type="info"}

Note: Look at the section of the output as highlighted from the below screenshot from your `fio` test output. This screenshot is only for reference purpose and not to indicate the actual performance stats.
::::

![FIO02](/static/images/fio_02.png)

- Use **IOping** to test the latency

::code[ioping -c 30 .]{language=bash showLineNumbers=false showCopyAction=true}

![Ioping-test-FSx](/static/images/Ioping-test-FSx.png)

- Write down the latency number across `Same Availability Zone` row in above table and exit the container

::code[exit]{language=bash showLineNumbers=false showCopyAction=true}

## EKS pod and FSx NetApp ONTAP in the different Availability Zone

Let’s do the test where the pod sits in a different Availability Zone of the storage

### Update the yaml file with your region subnet of different AZ
::code[vi pod_performance_different_AZ.yaml]{language=bash showLineNumbers=false showCopyAction=true}

* Press “i” to go into edit mode
* replace the topology.kubernetes.io/zone: `<lab region >` to ap-southeast-2b as shown in the image
* Press ESC and type `:wq` then press enter.

![pod-diff-az](/static/images/pod-diff-az.png)


### 1. Deploy the yaml file to provision the pod and the 10 GB storage on FSx NetApp ONTAP

::code[kubectl apply -f pod_performance_different_AZ.yaml]{language=bash showLineNumbers=false showCopyAction=true}

Wait for 1 minute

### 2.Log on to the container and perform FIO and IOping testing

::code[kubectl exec -it task-pv-pod-daz  -- bash]{language=bash showLineNumbers=false showCopyAction=true}

- Install FIO and IOping, then perform the FIO command to perform load testing, and write down the IOPS and Throughput

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
apt-get update
apt-get install fio ioping -y
:::

:::alert{header="Important" type="info"}
Note: In the below FIO test, we are running a simulation of a load test using 4KB small block size, with 50% read/write mix, with a random read-write pattern using 4 concurrent jobs
:::

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
cd /usr/share/trident-nas/
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=fiotest --filename=testfio --bs=4k --iodepth=64 --size=8G --readwrite=randrw --rwmixread=50 --numjobs=4 --group_reporting --runtime=30
:::

- Use **IOping** to test the latency

::code[ioping -c 30 .]{language=bash showLineNumbers=false showCopyAction=true}

- Write down the latency number under `Different Availability Zone` row in above table same as before and exit the container

::code[exit]{language=bash showLineNumbers=false showCopyAction=true}

## Performance Summary

The specific amount of throughput and IOPS that your workload can drive on your FSxONTAP file system depends on the throughput capacity, storage capacity configuration of your file system, and the nature of your workload. In this example, you provisioned 1024 GB as storage capacity and 128 MB/s as throughput.

![](/static/images/FSXOnTap.png)
## Summary

You have successfully completed this section of performance testing using FIO and IOping tool. From the result, the FSx for NetApp ONTAP storage has achieved sub-millisecond latency performances on Amazon EKS. In the next section, you will perform a failover test on FSx for NetApp ONTAP to understand Amazon FSx for Netapp ONTAP high availability feature.
