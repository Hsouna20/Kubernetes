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
