# Deploy Kubernetes Cluster on Ubuntu 20.04 using Kubeadm

In this documentation we will learn how to set up a Kubernetes cluster on Ubuntu usign Kubeadm

### What is Kubeadm
 Kubeadm is a tool used to build Kubernetes (K8s) clusters. Kubeadm performs the actions necessary to get a minimum viable cluster up and running quickly. By design, it cares only about bootstrapping, not about provisioning machines (underlying worker and master nodes).
## Prepare the environments
The following Steps must be applied to each node (both master nodes and worker nodes)
#### Disable the Swap Memory
The Kubernetes requires that you disable the swap memory in the host system because the kubernetes scheduler determines the best available node on which to deploy newly created pods. If memory swapping is allowed to occur on a host system, this can lead to performance and stability issues within Kubernetes

You can disable the swap memory by deleting or commenting the swap entry in `/etc/fstab` manually or using the `sed` command

         sudo swapoff -a && sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab`

This command disbales the swap memory and comments out the swap entry in `/etc/fstab`
#### Configure or Disable the firewall
When running Kubernetes in an environment with strict network boundaries, such as on-premises datacenter with physical network firewalls or Virtual Networks in Public Cloud, it is useful to be aware of the ports and protocols used by Kubernetes components.

The ports used by Master Node:

| Protocol  | Direction     | Port Range    |  Purpose 
| -------   | ------------- | ------------- | -------
| TCP       | Inbound       | 6443          | Kubernetes API server
| TCP       | Inbound       | 2379-2380     | etcd server client API
| TCP       | Inbound       | 10250         | Kubelet API
| TCP       | Inbound       | 10259         | kube-scheduler
| TCP       | Inbound       | 10257         | kube-controller-manager

The ports used by Worker Nodes: 

| Protocol  | Direction     | Port Range    |  Purpose 
| -------   | ------------- | ------------- | -------
| TCP       | Inbound       | 10250         | Kubelet API
| TCP       | Inbound       | 30000-32767   | NodePort Services

You can either disable the firewall or allow the ports on each node.
###### Method 1: Add firewall rules to allow the ports used by the Kubernetes nodes
Allow the ports used by the master node:
```bash
     sudo ufw allow 6443/tcp
     sudo ufw allow 2379:2380/tcp
     sudo ufw allow 10250/tcp
     sudo ufw allow 10259/tcp
     sudo ufw allow 10257/tcp
````
Allow the ports used by the worker nodes:
```bash
     sudo ufw allow 10250/tcp
     sudo ufw allow 30000:32767/tcp
```
###### Method 2: Disable the firewall
``` bash
sudo systemctl stop ufw
sudo systemctl disable ufw

```
#### Configure the local IP tables to see the Bridged Traffic
These steps are necessary for setting up the networking components required for Kubernetes. Kubernetes uses bridged networking to manage communication between different pods and nodes in the cluster. The bridge configurations allow the kernel to handle the network traffic appropriately and facilitate communication between different components of the Kubernetes cluster .
