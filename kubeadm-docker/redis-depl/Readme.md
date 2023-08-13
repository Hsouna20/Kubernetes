# Deploying Redis in Kubernetes

This guide explains how to deploy the official Redis image in a Kubernetes cluster. Redis is a popular in-memory data store and message broker.

## Prerequisites

- Kubernetes cluster up and running.
- `kubectl` command-line tool installed and configured to access your cluster.

## Step 1: Create a Redis Deployment

To deploy Redis, we'll use a Kubernetes Deployment to manage the Redis Pods.  
To apply the Deployment, execute the following command:

```bash
    kubectl apply -f redis-deployment.yaml
```
we can check the status of the Deployment and Pods using 
```bash 
    kubectl get deployment 
    kubectl get pods
```
## Step 2: Create a Redis Service

Next, we'll create a Kubernetes Service to expose the Redis Pods within the cluster. 
To apply the Service, run the following command:
```bash
kubectl apply -f redis-service.yaml
```
we can check the status of the Service using 
```bash
    kubectl get service
```
