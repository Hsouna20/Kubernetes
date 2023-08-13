Laravel App Deployment Guide

This guide provides step-by-step instructions on how to deploy a Laravel app using Docker and Kubernetes.
## Step 1: Create a Dockerfile

- Create a Dockerfile .
- Use the official PHP image as the base image.
- Install system dependencies and PHP extensions required by Laravel.
- Install Composer to manage PHP dependencies.
- Create a new Laravel app in the Docker container.
- Set the working directory to the Laravel app directory.
- Expose the default Laravel development server port.
- Start the Laravel development server.

## Step 2: Build and Push the Docker Image

- Build the Docker image using the command: 
    ```bash
     docker build -t your-dockerhub-username/laravel-app:latest .
    ```
- Log in to Docker Hub using 
    ```bash
     docker login
    ```
- Push the image to Docker Hub with 
    ```bash 
     docker push your-dockerhub-username/laravel-app:latest
    ```

## Step 3: Deploy the Laravel App to Kubernetes

- Create a Kubernetes deployment YAML file  for the Laravel app.
- Define the desired number of replicas.
- Specify the container image from Docker Hub.
- Expose the required ports and set environment variables as needed.
- Apply the deployment to Kubernetes using 
    ```bash 
     kubectl apply -f laravel-deployment.yaml
    ```

## Step 4: Access the Laravel App in the Browser
- Use kubectl get svc to find the service information.
- Note down the EXTERNAL-IP and PORT(S) values.
- In The web browser, visit http://EXTERNAL-IP:PORT to access the Laravel app.

Now the Laravel app should be deployed and accessible in the browser.


