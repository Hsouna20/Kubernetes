# Deploying MongoDB with Kubernetes

This repository contains files and instructions to deploy MongoDB as a service using Kubernetes and access it.

## Prerequisites

1. A running Kubernetes cluster.
2. `kubectl` configured to interact with the cluster.

## Deployment

## 1. Create the MongoDB deployment YAML file (mongo-deployment.yaml):
Apply the MongoDB deployment to your cluster:
```bash
    kubectl apply -f mongo-deployment.yaml
```
## 2-Create the MongoDB service YAML file (mongo-service.yaml):
Apply the MongoDB service to your cluster:
```bash
    kubectl apply -f mongo-service.yaml
```
## 3.Accessing MongoDB

Check the status of the MongoDB deployment and service:
```bash
    kubectl get pods
    kubectl get services
```
Get the external IP address of one of your Kubernetes nodes:

```bash

kubectl get nodes -o wide
```
 Access MongoDB using the NodePort and external IP:

```bash

http://EXTERNAL_IP:NODE_PORT

```
