# Deploying Drupal with Kubernetes

This guide will walk you through the steps to deploy Drupal using Kubernetes. We will use the official Drupal image and PostgreSQL as the database.

## Prerequisites

Before you begin, make sure you have the following:

- Kubernetes cluster up and running
- `kubectl` command-line tool installed and configured to access your cluster
- Docker installed on your local machine (for building custom images, if required)

## Step 1: Deploy PostgreSQL

First, let's deploy the PostgreSQL database for Drupal to use:

```bash
kubectl apply -f postgres-deployment.yaml
kubectl apply -f postgres-service.yaml
```

## Step 2: Deploy Drupal
Next, let's deploy the Drupal application:
```bash
kubectl apply -f drupal-deployment.yaml
kubectl apply -f drupal-service.yaml
```

## Step 3: Access Drupal
By default, the Drupal service is exposed externally as a LoadBalancer type. we can find the external IP by running:
```bash
kubectl get services drupal-service
```
Access Drupal using the external IP and port displayed in the output.
