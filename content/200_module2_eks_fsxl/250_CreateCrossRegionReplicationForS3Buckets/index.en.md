---
title : "Create cross region replication for S3 buckets"
weight : 250
---
-------------------------------------------------------------

In this section, you will create a cross region replication configuration for Amazon S3 buckets. These buckets are the data repositories of Amazon FSx for lustre filesystems in their respective regions. In this workshop we are operating between current lab region and us-east-2. This configuration will enable replication of the data between the buckets and available for the Amazon EKS cluster in the other region through the Amazon FSx for lustre persistent storage layer. Let's start..

## In this section we will configure the cross region replication configuration between the Amazon S3 buckets

:::alert{header="Note" type="info"}
For AWS Sponsored Workshop, The S3 bucket for FSx for lustre CSI driver is pre-created as the source with the name as **fsx-lustre-bucket-xxxx** and the destination with the name as **fsx-lustre-bucket-2ndregion-xxxx**
copy the source and the target bucket names in your favorite editor app
:::


### Step 1: Go to the S3 Console page click the link below

Open [S3 console](https://s3.console.aws.amazon.com)


### Step 2: Open bucket **fsx-lustre-bucket-xxxx** and Choose **Management**, scroll down to *Replication rules*, and then choose *Create replication rule*

![S3_01](/static/images/S3_01.png)

### Step 3: Under *Rule name*, enter a name for your rule to help identify the rule later. The name is required and must be unique within the bucket

### Step 4: Under *Status*, see that *Enabled* is selected by default

::alert[For AWS Sponsored Workshop we have already provisioned buckets with **Versioning**, you will not see **Enable Bucket Versioning** option, If you are running On Demand Workshop, Do not forget to **Enable Bucket Versioning**]

![S3_02](/static/images/S3_02.png)

### Step 5: Under *Source bucket* you will select replicate the whole bucket by choosing *Apply to all objects in the bucket*

### Step 6: Under *Destination*, select the bucket where you want Amazon S3 to replicate objects

:::alert{header="Note" type="info"}
For AWS Sponsored Workshop, the bucket is pre-created for you, please select the Bucket Name like **fsx-lustre-bucket-2ndregion-xxxx**. If you are running the self-paced labs, please create the 2nd S3 bucket in another region.
:::

::alert[Please also **Enable Bucket Versioning** for the destination bucket in the same window ]

![S3_03](/static/images/S3_03.png)

### Step 7: Set up an AWS Identity and Access Management (IAM) role that Amazon S3 can assume to replicate objects on your behalf. Choose an existing role that has been pre-created for you, and also select *Replicate objects encrypted with AWS KMS*

::alert[For AWS Sponsored Workshop the Pre-created IAM role starts with **s3-crr**]

IAM role: 

![S3_04](/static/images/S3_04.png)

Select *Replicate objects encrypted with AWS KMS* and select the KMS Key:

![S3_Encryption](/static/images/S3_Encryption.png)


### Step 8: Leave the additional options section as it is and click *save* to save the configuration

Keep all other options default and click Save at the bottom. 

![S3_05](/static/images/S3_05.png)

Now you should see a Pop-up, Here, select "No, do not replicate existing objects." and Submit.

![S3_Existing_Objects](/static/images/S3_06.png)

After this the rule is created, you can see that in your AWS S3 console page.

![S3_06](/static/images/S3_07.png)


:::alert{header="Note" type="info"}
For AWS Sponsored Workshop, The IAM permission for **s3-crr-xxx** is pre-created for you. If you are running the self-paced labs, please create the IAM permission as instructed below.
:::

Let us also look at the IAM permission for the IAM role **s3-crr-xxx**. **DOC-EXAMPLE-BUCKET1** is the source S3 bucket, and the **DOC-EXAMPLE-BUCKET2** is the destination bucket which has been replaced with the two S3 buckets pre-created. You can also check that in your IAM console.

::::expand{header="To verify click and check that the bucket names have changed"}

:::code[]{language=yaml showLineNumbers=false showCopyAction=false}
AWSTemplateFormatVersion: 2010-09-09
Description: "IAM Role for S3 Cross Region Replication"
Parameters:
  IAMRoleName: 
    Type: String
    Description: "IAM Role for S3 Cross Region Replication"
Resources:
  S3CRRRole:
    Type: 'AWS::IAM::Role'
    DeletionPolicy: Delete
    Properties:
      RoleName: !Ref IAMRoleName
      AssumeRolePolicyDocument:
        Statement:
          - Effect: Allow 
            Action:
              - 'sts:AssumeRole'
            Principal:
              Service:
                - s3.amazonaws.com
        Version: '2012-10-17'
  S3CRRRolePolicy:
    Type: 'AWS::IAM::Policy'
    DeletionPolicy: Delete
    Properties:
      PolicyName: !Sub "${IAMRoleName}-Policy"
      Roles:
          - Ref: S3CRRRole
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Action:
              - 's3:GetReplicationConfiguration'
              - 's3:ListBucket'
            Resource:
              - 'arn:aws:s3:::DOC-EXAMPLE-BUCKET1'
          - Effect: Allow
            Action:
              - 's3:GetObjectVersionForReplication'
              - 's3:GetObjectVersionAcl'
              - 's3:GetObjectVersionTagging'
            Resource:
              - 'arn:aws:s3:::DOC-EXAMPLE-BUCKET1/*'
          - Effect: Allow
            Action:
              - 's3:ReplicateObject'
              - 's3:ReplicateDelete'
              - 's3:ReplicateTags'
            Resource: 'arn:aws:s3:::DOC-EXAMPLE-BUCKET2/*'
:::
::::

## Summary

In this section, you have successfully created the cross region replication between two S3 buckets. Next section you will perform data movement from one EKS cluster in your current lab region to another EKS cluster in the `us-east-2` region using Amazon S3 native replication feature. This can help you to build a BCP or regional DR capabilities for the containerized workload deployments. 