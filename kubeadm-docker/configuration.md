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
##### 1- Enable the bridged traffic

bash
```
lsmod | grep br_netfilter
sudo modprobe br_netfilter
```
##### 2- Add "br_netfilter" to the modules load configuration file

bash
```
echo "br_netfilter" | sudo tee /etc/modules-load.d/k8s.conf
```
##### 3- Configure bridge-nf-call settings 
 
bash
```
echo "net.bridge.bridge-nf-call-ip6tables = 1" | sudo tee /etc/sysctl.d/k8s.conf
echo "net.bridge.bridge-nf-call-iptables = 1" | sudo tee -a /etc/sysctl.d/k8s.conf
sudo sysctl --system
```
#### Installing Docker Engine
Kubernetes requires you to install a container runtime to work correctly.There are many available options like containerd, CRI-O, Docker etc

By default, Kubernetes uses the Container Runtime Interface (CRI) to interface with your chosen container runtime.If you don't specify a runtime, kubeadm automatically tries to detect an installed container runtime by scanning through a list of known endpoints.

You must install the Docker Engine on each node! 

##### 1- Set up the repository 
```bash
     sudo apt update
     sudo apt install ca-certificates curl gnupg lsb-release
```
##### 2- Add Docker's official GPG key
```bash
     sudo mkdir -p /etc/apt/keyrings
     curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
##### 3- Add the stable repository using the following command:
```bash
         echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
##### 4- Install the docker container
```bash
     sudo apt update && sudo apt install docker-ce docker-ce-cli containerd.io -y
``` 

##### 5- Make sure that the docker will work on system startup
```bash
     sudo systemctl enable --now docker 
```
##### 6- Configuring Cgroup Driver:  
The Cgroup Driver must be configured to let the kubelet process work correctly
```bash
         cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```
##### 7- Restart the docker service to make sure the new configuration is applied
```bash
     sudo systemctl daemon-reload && sudo systemctl restart docker
```
#### Installing kubernetes (kubeadm, kubelet, and kubectl):

``` bash
# Install the following dependency required by Kubernetes on each node
     sudo apt install apt-transport-https

# Create the Kubernetes repository configuration:
      curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list


# Update the apt package index and install kubeadm, kubelet, and kubeclt
     sudo apt update && sudo apt install -y kubelet=1.23.1-00 kubectl=1.23.1-00 kubeadm=1.23.1-00
```

## Initializing the control-plane node
At this point, we have 3 nodes with docker, `kubeadm`, `kubelet`, and `kubectl` installed. Now we must initialize the Kubernetes master, which will manage the whole cluster and the pods running within the cluster `kubeadm init` by specifiy the address of the master node and the ipv4 address pool of the pods 

```bash
     sudo kubeadm init --apiserver-advertise-address=<master-ip-address> --pod-network-cidr=192.168.0.0/16
```
You should wait a few minutes until the initialization is completed. The first initialization will take a lot of time if your connexion speed is slow (pull the images of the cluster components)

#### Configuring kubectl 
As known, the `kubectl` is a command line tool for performing actions on your cluster. So we must to configure `kubectl`. Run the following command from your master node:
``` bash
    mkdir -p $HOME/.kube
     sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
     sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
##### 1- Install the Tigera Calico operator and custom resource definitions.

``` bash 
     kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
```
The Tigera Operator is a Kubernetes operator which manages the lifecycle of a Calico or Calico Enterprise installation on Kubernetes. Its goal is to make installation, upgrades, and ongoing lifecycle management of Calico and Calico Enterprise as simple and reliable as possible.

##### 2- Install Calico by creating the necessary custom resource

``` bash 
     kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml
```

Before you can use the cluster, you must wait for the pods required by Calico to be downloaded. You must wait until you find all the pods running and ready! 
``` bash
     kubectl get pods --all-namespaces
NAMESPACE          NAME                                       READY   STATUS    RESTARTS       AGE
calico-apiserver   calico-apiserver-5989576d6d-5nw7n          1/1     Running   1 (4min ago)    4min
calico-apiserver   calico-apiserver-5989576d6d-h677h          1/1     Running   1 (4min ago)    4min
calico-system      calico-kube-controllers-69cfd64db4-9hnh5   1/1     Running   1 (4min ago)    4min
calico-system      calico-node-lshdl                          1/1     Running   1 (4min ago)    4min
calico-system      calico-typha-76dd7c96d7-88826              1/1     Running   1 (4min ago)    4min
kube-system        coredns-64897985d-jkpwh                    1/1     Running   1 (4min ago)    4min
kube-system        coredns-64897985d-zk9wx                    1/1     Running   1 (4min ago)    4min
kube-system        etcd-master                                1/1     Running   1 (4min ago)    4min
kube-system        kube-apiserver-master                      1/1     Running   1 (4min ago)    4min
kube-system        kube-controller-manager-master             1/1     Running   1 (4min ago)    4min
kube-system        kube-proxy-4nf4q                           1/1     Running   1 (4min ago)    4min
kube-system        kube-scheduler-master                      1/1     Running   1 (4min ago)    4min
tigera-operator    tigera-operator-7d8c9d4f67-j5b2g           1/1     Running   2 (103s ago)    4min
```
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
-master            Ready      control-plane,master   40m5s  v1.23.1
-worker1           Ready      <none>                 3m7s   v1.23.1
-worker2           Ready      <none>                 2m3s   v1.23.1
```



References:
Kubernetes Documentation: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

Calico Documentation: Install Calico Networking for on-premises deployments :https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises

Docker Documentation: Install Docker Engine on Ubuntu : https://docs.docker.com/engine/install/ubuntu/


