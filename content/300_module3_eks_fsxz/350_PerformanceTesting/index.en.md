---
title : "Performance test with FIO and IOping"
weight : 340
---
-------------------------------------------------------------

In this section, you will look at important parameters related to storage performance, IOPS, throughput and latency. You will measure the performance of FSx for OpenZFS file system and volume provisioned by the FSx for OpenZFS CSI. You will use [FIO (Flexible I/O)](https://fio.readthedocs.io/en/latest/), the popular storage benchmarking tool, and [IOping](https://github.com/koct9i/ioping), a tool to monitor I/O latency in real time, to test the performances on FSx for OpenZFS drive from the EKS pod. Let's start..

Please capture the output of the performance numbers in your favorite app editor. Sample of the output format is shown below.

:::alert{header="Important" type="info"}
Amazon FSx for OpenZFS file system is built for high performance workloads. In this workshop you will be focusing on throughput performance stats (e.g. Megabytes per second).
:::

|          Scenario          | Average IOPS (read) | Average IOPS (write) | Average throughput (read) | Average throughput (write) | Average latency |
|:--------------------------:|:-------------------:|:--------------------:|:-------------------------:|:--------------------------:|:---------------:|
| FSx for OpenZFS Filesystem |                     |                      |                           |                            |                 |
|   FSx for OpenZFS Volume   |                     |                      |                           |                            |                 |


### Step 1: Here we are going to reuse pods deployed in previous section

Copy and run the below command to check the status of the pod. It should be in the running status

::code[kubectl get pods]{language=bash showLineNumbers=false showCopyAction=true}

### Step 2: Log in to the container and perform FIO and IOping testing for FSx for OpenZFS Filesystem

The below steps is to install the FIO and IOping utilities. Post which you will run the performance load testing and observe the IOPS and throughput

1. Login to the container with the below command

::code[kubectl exec -it fsx-openzfs-app-maz  -- bash]{language=bash showLineNumbers=false showCopyAction=true}

2. The below command to Install FIO and IOping

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
apt-get update
apt-get install fio ioping -y
:::

3. Follow the below instructions of FIO command to perform load testing, and write down the IOPS and Throughput

:::alert{header="Important" type="info"}
Note: In the below FIO test, you are running a simulation of a load test using 1MB large block size to test throughput using our small containers environment, with 50% read/write mix, with a random read-write pattern using 4 concurrent jobs.
:::

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
mkdir -p /data/performance
cd /data/performance
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=fiotest --filename=testfio --bs=4k --iodepth=64 --size=8G --readwrite=randrw --rwmixread=50 --numjobs=4 --group_reporting --runtime=30
:::

::::alert{header="Important" type="info"}
Note: Look at the section of the output as highlighted from the below screenshot from your `fio` test output. This screenshot is only for reference purpose and not to indicate the actual performance stats.
::::

![FIO03](/static/images/fio_03.png)

4. Run the below command of **IOping** to test the latency

::code[ioping -c 30 .]{language=bash showLineNumbers=false showCopyAction=true}

- Write down the latency number, delete the file, and exit the container

![Diagram](/static/images/Ioping-test-FSxZf.png)

5. Post completion of the above tests. Run the below command to remove the file testfio

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
rm -f /data/performance/testfio
exit
:::

### Step 3: Log in to the container and perform FIO and IOping testing for FSx for OpenZFS Volume

The below steps is to install the FIO and IOping utilities. Post which you will run the performance load testing and observe the IOPS and throughput

1. Login to the container with the below command

::code[kubectl exec -it fsx-openzfs-vol-app-maz -- bash]{language=bash showLineNumbers=false showCopyAction=true}

2. The below command to Install FIO and IOping

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
apt-get update
apt-get install fio ioping -y
:::

3. Follow the below instructions of FIO command to perform load testing, and write down the IOPS and Throughput

:::alert{header="Important" type="info"}
Note: In the below FIO test, you are running a simulation of a load test using 1MB large block size to test throughput using our small containers environment, with 50% read/write mix, with a random read-write pattern using 4 concurrent jobs.
:::

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
mkdir -p /data/performance
cd /data/performance
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=fiotest --filename=testfio --bs=4k --iodepth=64 --size=8G --readwrite=randrw --rwmixread=50 --numjobs=4 --group_reporting --runtime=30
:::

::::alert{header="Important" type="info"}
Note: Look at the section of the output as highlighted from the below screenshot from your `fio` test output. This screenshot is only for reference purpose and not to indicate the actual performance stats.
::::

![FIO04](/static/images/fio_04.png)


4. Run the below command of **IOping** to test the latency

::code[ioping -c 30 .]{language=bash showLineNumbers=false showCopyAction=true}

- Write down the latency number, delete the file, and exit the container

![Diagram](/static/images/Ioping-test-FSxZf-vol.png)

5. Post completion of the above tests. Run the below command to remove the file testfio

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
rm -f /data/performance/testfio
exit
:::

## Performance Summary

The specific amount of throughput and IOPS that your workload can drive on your FSx for OpenZFS file system depends on the throughput capacity, storage capacity configuration of your file system, and the nature of your workload. For more information on the Amazon FSx for OpenZFS performance kindly check this link [Amazon FSx for OpenZFS performance chart](https://docs.aws.amazon.com/fsx/latest/OpenZFSGuide/performance.html)

## Summary

You have successfully completed this section on performance testing of Amazon FSx for OpenZFS filesystem and volume using FIO and IOping tool. You observed the different performance metrics with EKS pod running the load on the FSx for OpenZFS with high throughput and sub-millisecond latency. 
