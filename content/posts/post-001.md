---
title: "Creating a Kubernetes cluster on AWS using kops"
date: 2022-08-15T17:47:46+05:30
draft: false
---

This article will help you to set up a Kubernetes cluster on AWS using kops. A command-line utility that helps you create your own master and worker nodes on AWS.

The Major difference from using the eksctl is — eksctl uses the EKS managed service that manages the master nodes for you, but here you manage the master nodes as well.

## **Requirements**

* a free-tier AWS account on which your user has the necessary permissions on IAM.

* an access key for the above mention account which can be downloaded from AWS Console

* AWS CLI 2 configured on your machine using AWS configure — kops will use this to connect to AWS Console

* Install kops on your machine

## **Getting Started**

*kops* much like terraform maintains a state for your cluster, the state is stored remotely in this case, an S3 bucket is needed. Follow the snippet below to create a bucket that will hold your kops state. A single bucket can hold multiple states and states of multiple clusters.

    REGION="us-east-1"
    BUCKET_NAME="sanjaybalaji-kops-store"

    # create a bucket in a specified region
    aws s3api create-bucket --bucket ${BUCKET_NAME} --region=${REGION}

    # enable bucket versioning so that your changes are not lost
    aws s3api put-bucket-versioning --bucket ${BUCKET_NAME} --region=${REGION} --versioning-configuration Status=Enabled

    # validate that bucket is created successfully
    aws s3 ls

## **Creating the cluster config**

Once you are done with step 1, run the following to create your cluster configuration

    # this can be used to avoid passing --state to all kops commands
    export KOPS_STATE_STORE=s3://${BUCKET_NAME}

    # this command creates a cluster configuration of a particular version

    kops create cluster --name sanjaybalaji.k8s.local --zones eu-west-1a --master-size t2.micro --node-size t2.micro --master-count 1 --node-count 2 --kubernetes-version 1.18.0 --api-loadbalancer-class network

This command only creates the cluster config, it does not provide your cluster.

The os, size, and count can be altered, you can spread the cluster across zones by passing in a CSV of other az’s — note that every zone needs one master minimum.
The API load balancer class is set to Classic by default, but that might cause an authentication issue with certain versions of Kubernetes, so we are using the Network type load balancer.

## **Check and Deploy your cluster**

To review your cluster configuration, you can run the following

    kops edit cluster --name sanjaybalaji.k8s.local

This will give you a configuration that is editable. Once you are sure of your config, you can move on to deployment

    kops update cluster --name sanjaybalaji.k8s.local --yes

This will take a few minutes to provision the cluster by creating the VCN, subnets, nodes, security groups etc.

## **Validate and Test your cluster**

To validate your cluster you can run the following command, which will make sure that your cluster is in the expected state.

    kops validate cluster — name sanjaybalaji.k8s.local — wait 10m
> In case your validate fails even after 10–15 mins with an Unathorized error, run this command kops export kubecfg --admin and then re-run the validate command.

To try a test run, you can create an nginx pod — and access [http://localhost:8080](http://localhost:8080) and you should see an Nginx welcome page.

    kubectl run nginx --image nginx

    kubectl port-forward nginx 8080:80

## **Cleanup**

Finally, To delete your cluster and all of it’s supporting resources, simply run the following command

    kops delete cluster --name sanjaybalaji.k8s.local --yes

For the S3 bucket, you would have to clean it and delete it from the AWS Console GUI.
