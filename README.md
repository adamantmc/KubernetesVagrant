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

### Example deployment of a sample NGINX

An example deployment of an NGINX service can be found in `deployments/nginx.yml`.

This deployment describes a single pod, with a single replica, which contains a container of the `nginx:latest` image.

We can create this deployment by running:
```
kubectl apply -f deployments/nginx.yml
```

To expose our deployment to the outside world, we need to create a service of `NodePort` type. This will result in NGINX's port 80 being accessible via a random external port on **any node of the cluster**.

The service can be created as such:
```
kubectl expose deployment nginx-sample-deployment --type=NodePort
```

After the service is created, we can find out the random port assigned by running:
```
kubectl get service nginx-sample-deployment
```

The `PORT(S)` field follows the `INTERNAL_PORT:EXTERNAL_PORT/PROTOCOL` structure. 

Let's assume that the command above returned the following:
```
NAME                      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-sample-deployment   NodePort   10.102.11.170   <none>        80:31797/TCP   2m57s
```

Then we can access our app via any NODE_IP:31797. Each node in the cluster will proxy to the node that is running the container. This means that we can access the app via any of the following:

1. 192.168.56.10:31797
2. 192.168.56.11:31797
3. 192.168.56.12:31797


### Example deployment using a private Docker registry

An example deployment file of a simple API can be found in `https://github.com/adamantmc/OnlineMovieStoreAPI/blob/master/kubernetes/deployments/online_movie_store_fastapi.yml`. 

This deployment uses an image stored in Docker Hub. It uses a Kubernetes Docker Secret to authenticate.

To create the secret, run the following:

```
kubectl create secret docker-registry SECRET_NAME --docker-server=https://index.docker.io/v2/ --docker-username USERNAME --docker-password PASSWORD --docker-email EMAIL
```

If the `SECRET_NAME` is passed under `imagePullSecrets` in the deployment, the docker credentials provided will be used to authenticate with the registry.
