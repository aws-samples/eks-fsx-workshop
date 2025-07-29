---
title: 'AWS Sponsored Workshop'
chapter: false
weight: 10
---
## Login into the AWS Console

::alert[Before you proceed log off from any previous AWS consoles and close all the web browser. Start a new browser]{header="Important" type="warning"}

::alert[If you are currently logged in to an AWS Account, you can logout using this [link](https://console.aws.amazon.com/console/logout!doLogout).]{type="warning"}


1. From your local workstation, open a web browser to the lab access URL provided for the workshop
    
    - Click on the [link](https://catalog.us-east-1.prod.workshops.aws/join) to be be taken to the sign in page
    
    ![Workshop Studio](/static/images/signin_page.png)

    - Click on the Email one-time password(OTP) and enter your email address to recieve the OTP

    - Enter the One-time email 9 digits passcode and click sign in

    ![Workshop Studio](/static/images/One_time_passcode.png)
    
    - You will be redirected to Join event page,  Eneter the event access code and click on **Next**

    ![Workshop Studio](/static/images/Start_page_join.png)

    - You will then be taken to the Review & Join page, click on the check box to agree the terms and condition. Click on the Join Event

    ![Workshop Studio](/static/images/review_join.png)

    - You will be taken to the workshop instructions page with all details

    - on the left bottom of the page you will find the AWS account access information.

    ![Workshop Studio](/static/images/account_access.png)

    - Click **Open AWS Console**


::alert[Ask Your Operator for the region to use.]


Please select the appropriate region in the top right corner.


## **Connect to your AWS lab environment via Open source VSCode IDE**

Ref : [code-server](https://github.com/coder/code-server) 

You will be using the Open source VSCode IDE terminal to copy and paste commands that are provided in this workshop. 

::alert[Note: Please use google chrome browser for best user experience. Firefox may experience some issues while copy-paste commands.]{header="Important" type="warning"}

1. Go to Cloud formation console [link](https://console.aws.amazon.com/cloudformation) and select `genaifsxworkshoponeks` stack 
2. Go to Stack **Outputs**
3. Copy Password and click URL
4. Enter copied password in the new tab opened for the URL


![CFN-Output](/static/images/cfn-output.png)

5. Select your VSCode UI theam 

![Select Theme](/static/images/select-theme.png)

6. You can maximize terminal window.

![maximize](/static/images/maximize.png)

:::alert{header="Note" type="info"}
When you first time copy-paste a command on VSCode IDE, your browser may ask you to allow permission to see informaiton on clipboard. Please select **"Allow"**.

![allow-clipboard](/static/images/allow-clipboard.png)
:::

- Replace `<region name>` with your lab region name as shared by your workshop operator. 

::code[export PRIMARY_REGION=$AWS_REGION]{language=bash showLineNumbers=false showCopyAction=true}

::code[export SECONDARY_REGION=us-east-2]{language=bash showLineNumbers=false showCopyAction=true}

- Set cluster variables : 

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
export PRIMARY_CLUSTER_NAME=FSx-eks-cluster
export SECONDARY_CLUSTER_NAME=FSx-eks-cluster02
:::


- Check if region and cluster names are set correctly

:::code[]{language=bash showLineNumbers=true showCopyAction=true}
echo "PRIMARY_REGION        :" $PRIMARY_REGION
echo "SECONDARY_REGION      :" $SECONDARY_REGION
echo "PRIMARY_CLUSTER_NAME  :" $PRIMARY_CLUSTER_NAME
echo "SECONDARY_CLUSTER_NAME:" $SECONDARY_CLUSTER_NAME
:::

## Update the kube-config file:
Before you can start running all the Kubernetes commands included in this workshop, you need to update the kube-config file with the proper credentials to access the cluster. To do so, in your VSCode IDE terminal run the below command:

::code[aws eks update-kubeconfig --name $PRIMARY_CLUSTER_NAME --region $PRIMARY_REGION]{language=bash showLineNumbers=false showCopyAction=true}


## Query the Amazon EKS cluster:
Run the command below to see the Kubernetes nodes currently provisioned:

::code[kubectl get nodes]{language=bash showLineNumbers=false showCopyAction=true}

You should see two nodes provisioned (which are the on-demand nodes used by the Kubernetes controllers), such as the output below:


![get-nodes](/static/images/get-nodes.png)


You now have a VSCode IDE Server environment set-up ready to use your Amazon EKS Cluster! You may now proceed with the next step.