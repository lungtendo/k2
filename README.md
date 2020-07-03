# k2
_Setup a two-node Kubernetes cluster_.

This repository contains Ansible plabooks reqiured to bootstrap a two-node Kubernetes cluster - one master and one worker node.
It also contains `Vagrantfile` configured for `libvirt` provider to two create the required nodes.

> The playbooks were tested on Ubuntu 18.04 servers. It is recommended to use the same OS version to avoid issues.

# Steps to Bootstrap the Cluster

## 1. Provision Nodes
Provision two nodes - one master and one worker node. 

Minimun system requirement for master node is 2 CPU cores and 2GB Memory. Resources for worker node can be configured as required.

Provision a user with sudo privilege and enable key-based SSH authentication for the user on both nodes. Also enable passwordless
sudo for this user for the duration of cluster setup. 

## 2. Update Ansible Inventory
The `hosts.ini` is a sample inventory for this setup. Update this file with proper IP addresses for your Kubernetes nodes.
The `master` and `cluster` entries are set to the same IP address. `cluster` can be assigned the address of `kube-apiserver` load balancer
if more control plane nodes are added in the future.

You can optionally replace `localdomain` with your own domain name. Make sure to update the playbooks with correct host names if you 
change the host name entries in the inventory file.

If the remote user name (on the nodes) is different from your local user name (on the Ansible control node) make sure to update the inventory 
file with the remote user name. Example:
```
master.localdomain ansible_host=192.168.33.110 ansible_user=username
worker.localdomain ansible_host=192.168.33.120 ansible_user=username
```

## 3. Update /etc/hosts on all nodes
```
ansible-playbook -i hosts.ini playbooks/etc-hosts.yaml
```

Ruuning the playbook will add entries such as the following to `/etc/hosts` on all nodes.
```
# BEGIN ANSIBLE MANAGED BLOCK
    192.168.33.110       master.localdomain      master
    192.168.33.120       worker.localdomain      worker
    192.168.33.110       cluster.localdomain      cluster
# END ANSIBLE MANAGED BLOCK
```

>NOTE: When running the playbooks, if the remote user is not configured for passwordless sudo add the option `-K` or `--ask-become-pass`
>to all `ansible-playbook` commands.

## 4. Install Dependencies

```
ansible-playbook -i hosts.ini playbooks/kube-dependencies.yaml
```
> This playbook refers the hosts by group name `nodes`.

## 4. Bootstrap the Master Node
Before running the playbook below, replace `MASTER_IP_ADDRESS` with the master node's address.
Also replace occurances of `USERNAME` with your remote user name - lines 18,27,29,30,34.

```
ansible-playbook -i hosts.ini playbooks/master.yaml
```

If you have replaced `localdomain` in the inventory file with your own domain name, 
make sure it is updated in this playbook as well - lines 1 and 7.

Once completed successfully, login to the master node and run `kubectl get nodes` to verify.
> OUTPUT:
```
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   47m   v1.18.4
```

If, for some reason, the cluster initialization fails, login to the master node, and run
1. `sudo kubeadm reset`
2. `sudo rm /root/cluster_initialized.txt`
3. Run the playbook again after fixing the cause of the issue.

## 5. Add Worker Node to the Cluster
Before running the playbook, make sure the host/domain name occurrances in the playbook are correct - lines 1,13,17.

```
ansible-playbook -i hosts.ini playbooks/worker.yaml
```

On master node, run `kubectl get nodes` again to verify.
>OUTPUT:
```
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   4m   v1.18.4
worker   Ready    <none>   1m   v1.18.4
```

## 6. Tests
Run some tests to insure your cluster is working.

### Deployment
```
kubectl create deploy nginx --image=nginx
```

### Service
```
kubectl expose deploy nginx --port 80 --target-port 80
```

### ReplicaSet
```
kubctl scale deploy nginx --replicas=2
```

## 7. Cleanup
```
kubectl delete deploy,svc -l app=nginx
```

## 8. Further
For setting up Ingress and Load Balancer follow the instructions at the following links:

- [Ingress](https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal)
- [Load Balancer](https://metallb.universe.tf/installation/#installation-by-manifest)

The load balancer needs to be configured futher before it is functional.
Check the [example lyer 2 configuration](https://metallb.universe.tf/configuration/#layer-2-configuration).
