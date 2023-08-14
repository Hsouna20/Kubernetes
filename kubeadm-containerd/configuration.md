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
### Forwarding IPv4 and letting iptables see bridged 
``` bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```
``` bash
sudo modprobe overlay
sudo modprobe br_netfilter
```
### Sysctl params required by setup, params persist across reboots
``` bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
```
###Apply sysctl params without reboot
```bash
sudo sysctl --system
```
### Verify that the br_netfilter, overlay modules are loaded by running below instructions
``` bash
lsmod | grep br_netfilter
lsmod | grep overlay
```
=> Verify system variables(net.bridge.bridge-nf-call-iptables, net.bridge.bridge-nf-call-ip6tables, net.ipv4.ip_forward) are set to 1 :
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
#### Installing Docker Engine
### 1- Uninstall docker (if exists):
```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```
### 2- Update apt packages :
```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release
```
### 3-Add Docker’s official GPG key:
``` bash
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
### 4-Set up the repository:
```baash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
### 5- install the containerd package:
```bash
sudo apt-get update
sudo apt-get install containerd.io
```
### 6- Verify that containerd service is running:
```bash
systemctl status containerd
```
### 7- configure "systemd" as a cgroup for containerd:
=> Open containerd configuration file located at: /etc/containerd/config.toml
sudo vim /etc/containerd/config.toml
=> Delete the entire content of the file and replace it with :
```bash
 [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```
### 8- Restart the containerd service and Check the status of containerd service: 
```bash
sudo systemctl restart containerd
systemctl status containerd
```
#### Installing kubernetes (kubeadm, kubelet, and kubectl):

``` bash
# Install the following dependency required by Kubernetes on each node
     sudo apt install apt-transport-https

# Create the Kubernetes repository configuration:
      curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list


# Update the apt package index and install kubeadm, kubelet, and kubeclt
     sudo apt update && sudo apt install -y kubelet kubectl kubeadm
```

### Initializing the control-plane node
At this point, we have 3 nodes with docker, `kubeadm`, `kubelet`, and `kubectl` installed. Now we must initialize the Kubernetes master, which will manage the whole cluster and the pods running within the cluster `kubeadm init` by specifiy the address of the master node and the ipv4 address pool of the pods 

```bash
     sudo kubeadm init --apiserver-advertise-address=<master-ip-address> --pod-network-cidr=10.244.0.0/16
```
You should wait a few minutes until the initialization is completed. The first initialization will take a lot of time if your connexion speed is slow (pull the images of the cluster components)

#### Configuring kubectl 
As known, the `kubectl` is a command line tool for performing actions on your cluster. So we must to configure `kubectl`. Run the following command from your master node:
``` bash
    mkdir -p $HOME/.kube
     sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
     sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### Deploy a weave-net daemonset in the controlplane node:
```bash
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```
=> verify that weave-net daemonset is deployed in the controlplane node:
```bash
kubectl get ds -n kube-system | grep weave-net
```
==>You should see an output contains weave-net.
```bash
kubectl get pods -n kube-system | grep weave-net
```
==>You should see an output contains a list of all available pods in kube-system namespace including  weave-net.
# Now we should configure weave-net addon to use the 10.244.0.0/16 range address when creating pods. For that we need to Edit weave-net manifest file in the controlplane:
``` bash
kubectl edit daemonset weave-net -n kube-system
```
=> Look for the environment variables for the "weave" container.
=> Add a new environment variable named "IPALLOC_RANGE".Set it's value to "10.244.0.0/16"
=> Save the file. This should delete the existing two pods for the weave-net daemonset and create a brand new two.

## Join the worker nodes
Now our cluster is ready to work! let's join the worker nodes to this cluster by getting the token from the master node 
``` bash
     sudo kubeadm token create --print-join-command
kubeadm join .....
```
Now let's move to the worker node and run the following command given by `kubeadm token create`

``` bash 
     sudo kubeadm join <master-ip-address>:....
``` 
The output must be similar to the following 
``` bash
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
W0623 12:45:07.940655   23651 utils.go:69] The recommended value for "resolvConf" in "KubeletConfiguration" is: /run/systemd/resolve/resolv.conf; the provided value is: /run/systemd/resolve/resolv.conf
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

```
Now let's Check the cluster by running `kubectl get nodes` command on the master node.

``` bash
     kubectl get nodes

NAME              STATUS     ROLES                  AGE    VERSION
-master            Ready      control-plane,master   40m5s  v1.27.4
-worker1           Ready      <none>                 3m7s   v1.27.4
-worker2           Ready      <none>                 2m3s   v1.27.4
```



References:
Kubernetes Documentation: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

weaveworks official documentation at "https://www.weave.works/docs/net/latest/kubernete

Install Containerd on Ubuntu : Docker official documentation at "https://docs.docker.com/engine/install/ubuntu/".

