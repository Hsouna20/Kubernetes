this file shows how to automate the cofiguration of a k8s cluster using ansible .
### 1- Generate ssh-key :
### What is ssh : 
SSH (Secure Shell) is a cryptographic network protocol used for secure remote access to systems over an unsecured network. It provides a secure way to log into and manage remote servers and devices. SSH encrypts the data transmitted between the client and the server, making it much more secure compared to traditional remote access methods like Telnet.
Ansible uses ssh to communicate between nodes .
To install SSH and generate an SSH key pair for Ansible on Ubuntu, follow these steps:
### Install ansible :
```bash 
sudo apt update
sudo apt install openssh-client openssh-server
```
### Generating ssh key :
Now we need to generate an SSH key pair. The key pair consists of a public key (used to authenticate with remote servers) and a private key (kept securely on your local machine).
```bash
ssh-keygen -t rsa -b 4096 
```
The ssh-keygen command will we prompt  for a location to save the keys. The default location is usually fine, but we can choose a custom location if needed.
### Copy Public Key to Remote Hosts:
Once the key pair is generated, we will need to copy the public key to the remote hosts we want to manage using Ansible
```bash
ssh-copy-id username@remote_host
```
Replace "username" and "remote_host" with the appropriate values.
This will copy your public key to the remote host's ~/.ssh/authorized_keys file, allowing we to authenticate without a password.
### 2- Install Ansible
### what is ansible : 
Ansible is an open-source automation tool that simplifies the process of configuring, managing, and deploying software applications and infrastructure. It's part of the DevOps toolchain and is widely used for automating repetitive tasks, orchestrating complex workflows, and maintaining consistent configurations across multiple servers or devices. 
### install ansible: 
```bash
sudo apt update
sudo apt install ansible
```

### Create an inventory file :
An inventory file in Ansible is where you define the hosts and groups you want to manage. You can create the inventory file at /etc/ansible/hosts or in the user's home directory.
To create an inventory file in the home directory. In the terminal, run:
```bash
sudo nano inventory.ini 
```
Add the hosts' IP addresses or hostnames in the file, like this:
```bash
[web_servers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[db_servers]
db1 ansible_host=192.168.1.20
```
### Test the communication :
We can use the ping module to test communication with your hosts.
```bash
ansible -i inventory.ini all -m ping
```
If everything is set up correctly, we should see output like this:
```bash
web1 | SUCCESS => {
    "ansible_ping": "pong"
}
web2 | SUCCESS => {
    "ansible_ping": "pong"
}
db1 | SUCCESS => {
    "ansible_ping": "pong"
}
```
in some case we should modify the /etc/sudoers file to make hosts use sudo command without errors .
###open the /etc/sudoers file :
```bash 
sudo visudo 
```
or 
```bash
sudo ano /etc/sudoers
```
### Add Ansible user permissions:
Find the section that defines user privileges. We should see lines that look like this:
```bash
root    ALL=(ALL:ALL) ALL
%sudo   ALL=(ALL:ALL) ALL
```
Add a line below the %sudo line to grant permissions to the Ansible user without requiring a password:
```bash
ansible_user ALL=(ALL:ALL) NOPASSWD:ALL
```
Replace 'ansible_user' with the actual username of the Ansible user. This line allows the Ansible user to execute any command with sudo privileges without being prompted for a password.
