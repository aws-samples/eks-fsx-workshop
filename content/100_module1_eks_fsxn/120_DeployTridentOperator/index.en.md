---
title : "Deploy Trident Operator"
weight : 120
---
-------------------------------------------------------------

In this section, you are going to download and install the FSx for NetApp ONTAP CSI driver Trident operator on your Amazon EKS cluster.

There are three ways to deploy the Trident (`HELM`,  `Trident Operator` and [tridentctl](https://netapp-trident.readthedocs.io/en/stable-v18.07/reference/tridentctl.html)). Please refer to the [Trident Official Document](https://netapp-trident.readthedocs.io/en/latest/kubernetes/fsx.html) for the instruction. Helm is a Kubernetes deployment tool for automating creation, packaging, configuration, and deployment of applications and services to Kubernetes clusters. For more information, please refer to the [official document](https://helm.sh/docs/). In this workshop, we will use `HELM` method to deploy the 21.07 version.

You can follow the below steps using either of the method (Linux, Windows or Cloud9)

Perform the following steps listed to deploy Trident Operator by using Helm. 

::alert[As mentioned in the previous session, you should have run the following commands to be able to run `kubectl` to connect to your EKS cluster. But if you missed it, please run the following commands]

::::expand{header="Confirm your access to the EKS cluster, Only run these commands if you have not yet done so in the previous steps."}

- Check if region and cluster names are set correctly, if not then follow one of the suitable page for your situation under **[Getting Started ](/020-setup)** to setup these variables. 


:::code[]{language=bash showLineNumbers=true showCopyAction=true}
echo $PRIMARY_REGION
echo $SECONDARY_REGION
echo $PRIMARY_CLUSTER_NAME
echo $SECONDARY_CLUSTER_NAME
:::

- Next update kubeconfig file to point it to EKS cluster in that region.

::code[aws eks update-kubeconfig --name $PRIMARY_CLUSTER_NAME --region $PRIMARY_REGION]{language=bash showLineNumbers=false showCopyAction=true}

- Run kubectl command to confirm your access to the EKS cluster

::code[kubectl get nodes]{language=bash showLineNumbers=false showCopyAction=true}

::::

## 1. Create Namespace called “trident”

::code[kubectl create ns trident]{language=bash showLineNumbers=false showCopyAction=true}

## 2. Download the installer bundle

Download the Trident v24.10.0 (or latest version) installer bundle from the [Trident Github](https://github.com/netapp/trident/releases) page. The installer bundle includes the Helm chart in the /helm directory.

### Download the file

::code[curl -L -o trident-installer-25.06.0.tar.gz https://github.com/NetApp/trident/releases/download/v25.06.0/trident-installer-25.06.0.tar.gz]{language=bash showLineNumbers=false showCopyAction=true}

### Extract the tar file downloaded

::code[tar -xvf ./trident-installer-25.06.0.tar.gz]{language=bash showLineNumbers=false showCopyAction=true}

## 3. Use the helm install command and specify a name for your deployment

Go to the trident folder downloaded from last step

:::code{language=bash showLineNumbers=false showCopyAction=true}
cd trident-installer/helm
ls -ltr
:::


Install via the helm command to the trident namespace (Match your file name as per your output above)
::code[helm upgrade --install trident -n trident trident-operator-100.2506.0.tgz]{language=bash showLineNumbers=false showCopyAction=true}


::::expand{header="You should see similar output like below, click to expand:"}

:::code{language=yaml showLineNumbers=false showCopyAction=false}
NAME: trident
LAST DEPLOYED: Wed Sep 27 06:38:46 2023
NAMESPACE: trident
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing trident-operator, which will deploy and manage NetApp's Trident CSI
storage provisioner for Kubernetes.

Your release is named 'trident' and is installed into the 'trident' namespace.
Please note that there must be only one instance of Trident (and trident-operator) in a Kubernetes cluster.

To configure Trident to manage storage resources, you will need a copy of tridentctl, which is
available in pre-packaged Trident releases.  You may find all Trident releases and source code
online at https://github.com/NetApp/trident.

To learn more about the release, try:

  $ helm status trident
  $ helm get all trident
:::

::::

## 4. Check the status of the trident operator

::code[kubectl get pods -n trident]{language=bash showLineNumbers=false showCopyAction=true}

::::expand{header="You should see the results as below, click to expand"}

:::code[]{language=bash showLineNumbers=false showCopyAction=false}
NAME                                  READY   STATUS    RESTARTS   AGE
trident-controller-5587878776-2rldk   6/6     Running   0          36s
trident-node-linux-8t6b8              2/2     Running   0          36s
trident-node-linux-lt6rj              2/2     Running   0          36s
trident-operator-78d545d599-25qhl     1/1     Running   0          64s
:::
::::

- Check the helm status

::code[helm status trident -n trident]{language=bash showLineNumbers=false showCopyAction=true}


::::expand{header="You should see the results as below, click to expand"}

:::code{language=yaml showLineNumbers=false showCopyAction=false}
NAME: trident
LAST DEPLOYED: Wed Sep 27 06:38:46 2023
NAMESPACE: trident
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing trident-operator, which will deploy and manage NetApp's Trident CSI
storage provisioner for Kubernetes.

Your release is named 'trident' and is installed into the 'trident' namespace.
Please note that there must be only one instance of Trident (and trident-operator) in a Kubernetes cluster.

To configure Trident to manage storage resources, you will need a copy of tridentctl, which is
available in pre-packaged Trident releases.  You may find all Trident releases and source code
online at https://github.com/NetApp/trident.

To learn more about the release, try:

  $ helm status trident
  $ helm get all trident
:::

::::

## Summary

By the end of this section, you have successfully downloaded and installed the trident operator. Next section you will install the Trident CSI driver and storage class, then provision the persistent volume (PV) which will automatically provision the volume in FSx for NetApp ONTAP SVM.
