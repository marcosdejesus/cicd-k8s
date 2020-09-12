CI/CD pipeline for a simple web application using CloudFormation, Jenkins and Kubernetes
===

## Table of Contents

[TOC]

## Configuration steps

To deploy this project we will need an EC2 Instance with Jenkins, AWS kubectl and tidy installed. All the commands will be executed inside this instance.

### Creating the kubernetes cluster

We will create the cluster and the node group using cloudformation templates.

1. Creating the network infrastructure and the EKS Cluster
```console
user@instance:~$ aws cloudformation create-stack --stack-name infraCluster --template-body file://infrastructure/createNetworkCluster.yaml --capabilities CAPABILITY_IAM
```
2. After the cluster stack creation completes, create the node group for the cluster
```console
user@instance:~$ aws cloudformation create-stack --stack-name NodeGroup --template-body file://infrastructure/createNodeGroup.yaml --capabilities CAPABILITY_IAM
```

### Configuring kubectl

1. Get the kubeclt configuration
```console
user@instance:~$ aws eks --region us-west-2 update-kubeconfig --name devCluster
```
2. Copy the config to the jenkins user directory
```console
user@instance:~$ sudo cp ~/.kube/config ~jenkins/.kube/
user@instance:~$ sudo chown -R jenkins: ~jenkins/.kube/

```

## Deployment method

The deployment method used for this project is the Blue/Green Deployment.

The follow diagram explains how it works:
![Diagram](/images/diagram.png)
