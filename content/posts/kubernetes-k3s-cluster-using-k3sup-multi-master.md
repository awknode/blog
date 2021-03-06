---
title: "Kubernetes K3s Cluster Using K3sup Multi Master"
date: 2020-06-08T13:01:40+02:00
draft: false
toc: false
description: summary of blogpost
author: loeken
summary: in this part we are going to setup a 3 master node k3s cluster inside virtual box
images:
tags:
  - untagged
---

# getting started information

## Terminology

- kubernetes: Full blown container orchestration tool
- minicube: a 1 node kubernetes cluster running inside a vm ( good for local testing )
- k3s a lightweight alternative to Kubernetes  with a lot of unneeded code removed
- k3sup a small extra tool that helps you getting your k3s cluster going quickly 

## Why k3s what is the difference to kubernetes

{{<  youtube_lazy ytid="-HchRyqNtkU" yttitle="k3s under the hood" >}}

<style type="text/css">
.flex { 
    display: flex; 
    justify-content: center; 
    align-items: center;
}
</style>

## Test Cluster layout

<div class="flex">

![TestCluster Layout](/media/img/k3s_testlab_01.png)

</div>

[Download Image Markup](/media/imgmarkup/k3s_testlab_01.py)

## prepare 3 virtual machines

I used virtualbox to create 3 vms ( debian 10 netinstaller ) i gave each 4GB of ram and a 8GB virtual disk



the ips of these test vms are in the 172.16.137.0/24 subnet
- k3s-01: 172.16.137.43
- k3s-02: 172.16.137.44
- k3s-03: 172.16.137.45


these are the dependencies k3sup needs in order to run. I created a user called loeken which i ll be using in this tutorial.
```
sudo apt install curl sudo
```

sudo expects to be configured to not use a password we are doing this by editing the /etc/sudoers config
```
%sudo	ALL=(ALL:ALL) NOPASSWD:ALL
```

now we add the user to the sudo group
```
usermod -a -G sudo loeken
```

we also transfer our id_rsa.pub onto the 3 vms so we can login. k3sup uses id_rsa.pub/id_rsa it seems 
( it did not support my kr ssh agent forwarding )
```
ssh-copy-id loeken@172.16.137.43
ssh-copy-id loeken@172.16.137.44 
ssh-copy-id loeken@172.16.137.45 
```

## installing k3sup locally on my workstation:

```
curl -sLS https://get.k3sup.dev | sh
sudo install k3sup /usr/local/bin/

k3sup --help
```


## installation of k3s using k3sup
creating the cluster on the first node. this will create a kubeconfig file in the home folder of the user you are running this command from, i ran this from my workstation.
not the added extra --bind-address and the advertise address params which tells the server to not bind on the ip of the primary but only on the secondary interface
and also to advertise this address to others to be used for communication.

we will be going with version v1.19.1-rc2+k3s1 as it does not have this dqlite crap that keeps on breaking but uses etcd
```
k3sup install --k3s-version v1.19.1-rc2+k3s1 \
      --ip 172.16.137.43 \
      --user loeken \
      --cluster \
      --k3s-extra-args '--no-deploy=traefik --bind-address=172.16.137.43 --advertise-address=172.16.137.43 --node-ip=172.16.137.43 --node-external-ip 1.2.3.4'
Running: k3sup install
Public IP: 172.16.137.43
ssh -i /home/loeken/.ssh/id_rsa -p 22 loeken@172.16.137.43
ssh: curl -sLS https://get.k3s.io | INSTALL_K3S_EXEC='server --cluster-init --tls-san 172.16.137.43 --bind-address=172.16.137.43 --advertise-address=172.16.137.43 --node-ip=172.16.137.4' INSTALL_K3S_VERSION='v1.17.2+k3s1' sh -

[INFO]  Using v1.17.2+k3s1 as release
[INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v1.17.2+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v1.17.2+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s
Result: [INFO]  Using v1.17.2+k3s1 as release
[INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v1.17.2+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v1.17.2+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
[INFO]  systemd: Starting k3s
 Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.

ssh: sudo cat /etc/rancher/k3s/k3s.yaml

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJXRENCL3FBREFnRUNBZ0VBTUFvR0NDcUdTTTQ5QkFNQ01DTXhJVEFmQmdOVkJBTU1HR3N6Y3kxelpYSjIKWlhJdFkyRkFNVFU1TVRJNE1qVTVOekFlRncweU1EQTJNRFF4TkRVMk16ZGFGdzB6TURBMk1ESXhORFUyTXpkYQpNQ014SVRBZkJnTlZCQU1NR0dzemN5MXpaWEoyWlhJdFkyRkFNVFU1TVRJNE1qVTVOekJaTUJNR0J5cUdTTTQ5CkFnRUdDQ3FHU000OUF3RUhBMElBQkFoWFlkWW5ZWjVCcVUwS3JhdmpkZVNIQXFJcVNoKzRnT1N0cDFrOUVNQUMKS1RLbyt6RjNoQmZ3UGF4VzRZOHF1Q2hjdDNPZVBPekVvdzAwanFmT0t1MmpJekFoTUE0R0ExVWREd0VCL3dRRQpBd0lDcERBUEJnTlZIUk1CQWY4RUJUQURBUUgvTUFvR0NDcUdTTTQ5QkFNQ0Ewa0FNRVlDSVFDL3hYeCthQm5pCmZzUk9kMG53dkczaGlaWURlcmJYK3A1MmgzNVI5QUpYWGdJaEFMcWxkZVZMVXlRR1R3Z1JVY01TYTE0enF1ekQKaUdxc2JQZkViUVZpbHpxRQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://172.16.137.4:6443
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    password: afd3bd64ed2aef936b904d13dee80739
    username: admin
Result: apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJXRENCL3FBREFnRUNBZ0VBTUFvR0NDcUdTTTQ5QkFNQ01DTXhJVEFmQmdOVkJBTU1HR3N6Y3kxelpYSjIKWlhJdFkyRkFNVFU1TVRJNE1qVTVOekFlRncweU1EQTJNRFF4TkRVMk16ZGFGdzB6TURBMk1ESXhORFUyTXpkYQpNQ014SVRBZkJnTlZCQU1NR0dzemN5MXpaWEoyWlhJdFkyRkFNVFU1TVRJNE1qVTVOekJaTUJNR0J5cUdTTTQ5CkFnRUdDQ3FHU000OUF3RUhBMElBQkFoWFlkWW5ZWjVCcVUwS3JhdmpkZVNIQXFJcVNoKzRnT1N0cDFrOUVNQUMKS1RLbyt6RjNoQmZ3UGF4VzRZOHF1Q2hjdDNPZVBPekVvdzAwanFmT0t1MmpJekFoTUE0R0ExVWREd0VCL3dRRQpBd0lDcERBUEJnTlZIUk1CQWY4RUJUQURBUUgvTUFvR0NDcUdTTTQ5QkFNQ0Ewa0FNRVlDSVFDL3hYeCthQm5pCmZzUk9kMG53dkczaGlaWURlcmJYK3A1MmgzNVI5QUpYWGdJaEFMcWxkZVZMVXlRR1R3Z1JVY01TYTE0enF1ekQKaUdxc2JQZkViUVZpbHpxRQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://172.16.137.4:6443
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    password: afd3bd64ed2aef936b904d13dee80739
    username: admin
 
Saving file to: /home/loeken/kubeconfig

# Test your cluster with:
export KUBECONFIG=/home/loeken/kubeconfig
kubectl get node -o wide
```

so we are doing what we are told:
```
export KUBECONFIG=/home/loeken/kubernetes/kubeconfig                                                                                                                                                                                                                 0.01   12:18  
kubectl get node -o wide
NAME     STATUS   ROLES    AGE     VERSION        INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                       KERNEL-VERSION   CONTAINER-RUNTloeken
k3s-01   Ready    master   7m13s   v1.17.2+k3s1   94.23.161.209   <none>        Debian GNU/Linux 10 (buster)   4.19.0-9-amd64   containerd://1.3.3-k3s1
```

this will set the environment variable but this won't remain after reboot so i ll make it persistent via:
```
echo "export KUBECONFIG=/home/loeken/kubernetes/kubeconfig" >> .zshrc
```

notice that im using the zsh shell if you are not using zsh you might want to use the ~/.bashrc or the ~/.profile file instead

joining in the second node
```
k3sup join --k3s-version v1.19.1-rc2+k3s1 \
      --ip 172.16.137.44 \
      --server-ip 172.16.137.43 \
      --user loeken \
      --server \
      --k3s-extra-args '--no-deploy=traefik --bind-address=172.16.137.44 --advertise-address=172.16.137.44 --node-ip=172.16.137.44 --node-external-ip=1.2.3.4'  

Running: k3sup join
Server IP: 172.16.137.43
K10f239b8838d7e8c915c4824cd6c00f4b3a4ec62a8cd30f2549be38c1a2cb55d23::server:e0c44f337986756a0adb640f74505f23
[INFO]  Using v1.19.1-rc2+k3s1 as release
[INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v1.19.1-rc2+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v1.19.1-rc2+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s
Logs: Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
Output: [INFO]  Using v1.19.1-rc2+k3s1 as release
[INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v1.19.1-rc2+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v1.19.1-rc2+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
[INFO]  systemd: Starting k3s

```

three times the charm:
```
k3sup join --k3s-version v1.19.1-rc2+k3s1 \
      --ip 172.16.137.45 \
      --server-ip 172.16.137.43 \
      --user loeken \
      --server \
      --k3s-extra-args '--no-deploy=traefik --bind-address=172.16.137.45 --advertise-address=172.16.137.45 --node-ip=172.16.137.45 --node-external-ip=1.2.3.4'  

Running: k3sup join
Server IP: 172.16.137.43
K10f239b8838d7e8c915c4824cd6c00f4b3a4ec62a8cd30f2549be38c1a2cb55d23::server:e0c44f337986756a0adb640f74505f23
[INFO]  Using v1.19.1-rc2+k3s1 as release
[INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v1.19.1-rc2+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v1.19.1-rc2+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s
Logs: Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
Output: [INFO]  Using v1.19.1-rc2+k3s1 as release
[INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v1.19.1-rc2+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v1.19.1-rc2+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
[INFO]  systemd: Starting k3s

```

## verifcation of installation
```
kubectl get node -o wide
NAME     STATUS   ROLES         AGE     VERSION            INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                       KERNEL-VERSION    CONTAINER-RUNTIME
k3s-01   Ready    etcd,master   5m19s   v1.19.1-rc2+k3s1   172.16.137.43   1.2.3.4       Debian GNU/Linux 10 (buster)   4.19.0-11-amd64   containerd://1.4.0-k3s1
k3s-02   Ready    etcd,master   116s    v1.19.1-rc2+k3s1   172.16.137.44   1.2.3.4       Debian GNU/Linux 10 (buster)   4.19.0-11-amd64   containerd://1.4.0-k3s1
k3s-03   Ready    etcd,master   6s      v1.19.1-rc2+k3s1   172.16.137.45   <none>        Debian GNU/Linux 10 (buster)   4.19.0-11-amd64   containerd://1.4.0-k3s1

```

## installing the kubernetes dashboard

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.4/aio/deploy/recommended.yaml
```

now we create 2 markups to define the user access to the dashboard
#### **`dashboard.admin-user.yaml`**
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard"
```
#### **`dashboard.admin-user-role.yaml`**
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```
apply the configuration
```
kubectl apply -f dashboard.admin-user.yaml
kubectl apply -f dashboard.admin-user-role.yaml
```


next step is to get the bearer token which we need to login to the dashboard in the following step
```
k3s kubectl -n kubernetes-dashboard describe secret admin-user-token | grep ^token
```

since k3s version 1.19 we have to use the proxy to access the dashboard but this is not tricky at all:
```
kubectl proxy
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/clusterrole?namespace=default
```


create a deployment for nginx:
#### **`nginx-deployment.yaml`**
```
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.16.1
        ports:
        - containerPort: 80
```

```
kubectl apply -f nginx-deployment.yaml
```