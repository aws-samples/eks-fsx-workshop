---
title : "Performance test with FIO and IOping"
weight : 240
---
-------------------------------------------------------------

In this section, you will look at important parameters related to storage performance, IOPS, throughput and latency. You will measure the performance of FSx for Lustre file system provisioned by the FSx For Lustre CSI. You will use [FIO (Flexible I/O)](https://fio.readthedocs.io/en/latest/), the popular storage benchmarking tool, and [IOping](https://github.com/koct9i/ioping), a tool to monitor I/O latency in real time, to test the performances on FSx for Lustre drive from the EKS pod. Let's start..

Please capture the output of the performance numbers in your favorite app editor. Sample of the output format is shown below.

:::alert{header="Important" type="info"}
Amazon FSx for Lustre file system is built for high performance workloads. In this workshop you will be focusing on throughput performance stats (e.g. Megabytes per second).
:::

|         Scenario          | Average throughput (read) | Average throughput (write) | Average latency |
|:-------------------------:|:-------------------------:|:--------------------------:|:---------------:|
| FSx for Lustre Filesystem |                           |                            |                 |

### Step 1: Deploy the yaml file to provision the pod and the 10 GB storage on FSx for Lustre

Go to the right working directory.

::code[cd /home/participant/environment/fsx-workshop-on-eks/static/eks/FSxL]{language=bash showLineNumbers=false showCopyAction=true}

Write down the availability zone of the FSx for Lustre with below output of the command

::code[aws ec2 describe-subnets --subnet-id $SUBNET_ID --region $PRIMARY_REGION | jq .Subnets[0].AvailabilityZone]{language=bash showLineNumbers=false showCopyAction=true}

::alert[Note: In Cloud9 you can follow the same instructions or you could edit the file directly from Cloud9 browser]

Edit the **pod_performance.yaml**

::code[vi pod_performance.yaml]{language=bash showLineNumbers=false showCopyAction=true}

::alert[Below steps are very important for your performance test, as it makes sure that you deploy the pod in the same availability zone as your FSx for Lustre file system.]{header="Important" type="info"}

Press “i” to go into edit mode

Uncomment by removing `#` in the last two lines starting with **nodeSelector** and **topology.kubernetes.io/zone**, as the below screenshot. 

Replace the `us-east-2c` with the availability zone you noted from the previous step, as the below screenshot.

Press ESC and type `:wq` then press enter. 

![FSXlperf03](/static/images/fsxl_perf_03.png)

Copy and run the below command to provision the pod

::code[kubectl apply -f pod_performance.yaml]{language=bash showLineNumbers=false showCopyAction=true}

:::alert{header="Important" type="info"}
Check the status of the pod, wait until it is up and running, normally it takes upto 1 minute. If your pod remains in pending for longer and pod events shows message like below, this means that FSx Lustre filesystem is yet to be created. 

PVC `fsx-lustre-claim` should be in pending status, wait for upto 15 mins. Pod should transision to running once PVC `fsx-lustre-claim` is in bound status.

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  25s   default-scheduler  0/2 nodes are available: 2 pod has unbound immediate PersistentVolumeClaims. preemption: 0/2 nodes are available: 2 Preemption is not helpful for scheduling.
:::


Copy and run the below command to check the status of the pod. It should be in the running status

::code[kubectl get pods]{language=bash showLineNumbers=false showCopyAction=true}

### Step 2: Log in to the container and perform FIO and IOping testing

The below steps is to install the FIO and IOping utilities. Post which you will run the performance load testing and observe the IOPS and throughput

1. Login to the container with the below command

::code[kubectl exec -it fsxl-performance  -- bash]{language=bash showLineNumbers=false showCopyAction=true}

2. The below command to Install FIO and IOping

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
apt-get update
apt-get install fio ioping -y
:::

3. Follow the below instructions of FIO command to perform load testing, and write down the IOPS and Throughput

::alert[Note: In the below FIO test, you are running a simulation of a load test using 1MB large block size to test throughput using our small containers environment, with 50% read/write mix, with a random read-write pattern using 4 concurrent jobs.]{header="Important" type="info"}


:::code[]{language=bash showLineNumbers=true showCopyAction=true}
mkdir -p /data/performance
cd /data/performance
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=fiotest --filename=testfio8gb --bs=1MB --iodepth=64 --size=8G --readwrite=randrw --rwmixread=50 --numjobs=4 --group_reporting --runtime=30
:::

::alert[Note: Look at the section of the output as highlighted from the below screenshot from your `fio` test output. This screenshot is only for reference purpose and not to indicate the actual performance stats.]{header="Important" type="info"}

![FIO01](/static/images/fio_01.png)

1. Run the below command of **IOping** to test the latency

::code[ioping -c 30 .]{language=bash showLineNumbers=false showCopyAction=true}

- Write down the latency number, delete the file, and exit the container

![Diagram](/static/images/Ioping-test-FSxL.png)

5. Post completion of the above tests. Run the below command to remove the file testfio8gb

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
rm -f /data/performance/testfio8gb
exit
:::

## Performance Summary

The specific amount of throughput and IOPS that your workload can drive on your FSx for Lustre file system depends on the throughput capacity, storage capacity configuration of your file system, and the nature of your workload. For more information on the Amazon FSx for Lustre performance kindly check this link [Amazon FSx for Lustre performance chart](https://docs.aws.amazon.com/fsx/latest/LustreGuide/performance.html)

## Summary

You have successfully completed this section on performance testing of Amazon FSx for Lustre filesystem using FIO and IOping tool. You observed the different performance metrics with EKS pod running the load on the FSx for Lustre with high throughput and sub-millisecond latency. In the Next section you will create a cross region replication of your S3 buckets. These buckets are the data repositories of the Amazon FSx for Lustre filesystems in the module.
