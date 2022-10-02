# Kubernetes deployment in Vagrant VMs

This repo contains Ansible scripts that provision a Kubernetes cluster in Vagrant VMs.

## Prerequisites

Vagrant is needed to provision the VMs and Ansible is needed to run the scripts that provision the Kubernetes cluster.

Both the Vagrantfile and the Ansible scripts assume that there is a directory named `keys` in the root of the repository which contains a private (`id_rsa`)
and a public key (`id_rsa.pub`). You can generate a keypair by running:

```
ssh-keygen -t rsa -b 4096
```

## Starting the Vagrant VMs

The Vagrant VMs can be started by running `vagrant up` in the same directory as the `Vagrantfile`. The Vagrantfile defines 3 VMs - one master and 2 workers.

The VMs use the `192.168.56.x` network and more specifically the following IPs:

1. master  - 192.168.56.10
2. worker1 - 192.168.56.11
3. worker2 - 192.168.56.12

## Running Ansible

After the VMs have started, the Ansible scripts must run to provision the Kubernetes cluster. The scripts can be run as such (from the `ansible` dir):

```
ansible-playbook kubernetes_node_setup.yml -i inventory.ini --become
```

## Using the Cluster

Once the Ansible scripts are executed, the Kubernetes cluster is ready to use. For a fast setup, copy the `/etc/kubeconfig/admin.conf` file from `master` locally
and set the `KUBECONFIG` env var to point to it. 

The versions of kubeadm, kubelet and kubectl user are all `1.25.2-00`.
