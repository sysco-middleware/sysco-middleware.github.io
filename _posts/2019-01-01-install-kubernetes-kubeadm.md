---
layout: post
title: Installing on-premise Kubernetes cluster using kubeadm
categories: kubernetes, containers, orchestration
tags: [kubernetes, containers, orchestration]
author: PrakharSrivastav
---
# Motivation and target audience

This blog post is geared towards developers and administrators who want to setup an on-premise Kubernetes cluster. This post will guide you towards the basics of setting up a multi-node kubernetes cluster on linux servers and will address some of the key concerns during the setup.

Kubernetes is a huge area in itself and this post does not intend to cover all the nitty-gritty setup details. Instead, this blog post aims to provide a convenient starting point. 

# Introduction

Kubernetes (K8s) is a container orchestration technology that focuses on easing the efforts required to build, deploy, scale and manage containerized applications. It also provides means to manage complicated and dynamic life cycle of containerized applications. It was first developed at Google, but later on open sourced as a seed technology to Cloud Native Computing Foundation (CNCF). 

From Kubernetes  website:

> Kubernetes (k8s) is an open-source system for automating deployment, scaling, and management of containerized applications. It groups containers that make up an application into logical units for easy management and discovery. Kubernetes builds upon 15 years of experience of running production workloads at Google, combined with best-of-breed ideas and practices from the community.

Some of the distinguished kubernetes features are:
- **Service discovery and load balancing**. 
- **Automatic binpacking**.
- **Storage orchestration**. 
- **Self-healing**. 
- **Automated rollouts and rollbacks**.
- **Secret and configuration management**.
- **Batch execution**.
- **Horizontal scaling**.

You can read more about Kubernetes features [here](https://kubernetes.io/). If you are interested in some case studies you can find them [here](https://kubernetes.io/case-studies/). In the next section we will start with installing the kubernetes cluster on linux.

## Content:

- Prerequisites.
    - Hardware and OS specifications
    - Hostname configurations
    - Configure host files
    - Verify MAC and product_uuid
    - Disable SELinux and Swap
- Install software and other dependencies.
    - Install docker
    - Install Kubernetes components
- Configure kubernetes master and initialize cluster.
    - Initialize master
    - Create kube configuration in home directory
    - Install flannel network
- Add nodes to the cluster.
- Sources and References.

## 1. Prerequisites

In this example we will set up a kubernetes cluster with one master and 2 nodes. We will use the below setup for our kubernetes cluster.

### Hardware and OS specifications.
We will use 3 CentOS 7 servers with minimum 2 CPU and 2 GB RAM. You should have root privileges on the servers to install the required software packages.

**Note** : The hardware mentioned is to setup basic minimum configuration for kubernetes. Kubernetes comes with lots of bells and whistles and if you are installing all bells and whistles, please refer to this documentation for more details.
For this example we have provisioned 3 server with below IPs
- 192.168.37.48 - This will act as our Master
- 192.168.37.49 - This will be first node of our cluster
- 192.168.37.50 - This will be the second node of the cluster

It is important to mention here that kubernetes works in accordance with master slave architecture. The pods will never be scheduled on the master, instead master will act as a coordinator to manage different kubernetes services, manage traffic between the pods and manage workload scheduling on the nodes.

### Hostname configurations
Login to each of the server via terminal and change the hostname by following below 2 steps. For example on server 192.168.37.48 we have setup the corresponding host name as `kubernetes1.oslo.sysco.no`
- run `hostnamectl set-hostname kubernetes1.oslo.sysco.no`
- update host name in the file /etc/hostname to `kubernetes1.oslo.sysco.no`

Follow same steps for other two servers as well. 

### Configure host files

In order for each of our hosts to communicate with others hosts by hostname, we should modify the host file configurations. We will add the below content to `/etc/host` file on **each server**. 
```
192.168.37.48 kubernetes1.oslo.sysco.no
192.168.37.49 kubernetes2.oslo.sysco.no
192.168.37.50 kubernetes3.oslo.sysco.no
```
**Note** : Its important that you replace above settings with your IP and server alias.

After applying above settings, you should be able to ping other servers from each host. For example, in our case, we can ping `kubernetes2.oslo.sysco.no` from the host `kubernetes1.oslo.sysco.no`
```
[root@kubernetes1 ~]# ping kubernetes2.oslo.sysco.no
PING kubernetes2.oslo.sysco.no (192.168.37.49) 56(84) bytes of data.
64 bytes from kubernetes2.oslo.sysco.no (192.168.37.49): icmp_seq=1 ttl=64 time=0.460 ms
64 bytes from kubernetes2.oslo.sysco.no (192.168.37.49): icmp_seq=2 ttl=64 time=0.229 ms
64 bytes from kubernetes2.oslo.sysco.no (192.168.37.49): icmp_seq=3 ttl=64 time=0.205 ms
64 bytes from kubernetes2.oslo.sysco.no (192.168.37.49): icmp_seq=4 ttl=64 time=0.199 ms
64 bytes from kubernetes2.oslo.sysco.no (192.168.37.49): icmp_seq=5 ttl=64 time=0.262 ms
--- kubernetes2.oslo.sysco.no ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4000ms
rtt min/avg/max/mdev = 0.199/0.271/0.460/0.097 ms
```

References:
- [Hardware and OS requirements](https://kubernetes.io/docs/setup/independent/install-kubeadm/#before-you-begin)

### Verify MAC and product_uuid
 We need to verifiy that each host in the cluster has unique MAC and product_uuid. Kubernetes uses these values to uniquely identify the nodes in the cluster. If these values are not unique to each node, the installation process may [fail](https://github.com/kubernetes/kubeadm/issues/31).

- You can get the MAC address of the network interfaces using the command `ip link` or `ifconfig -a`. For example the MAC address in our example are: 
    - `kubernetes1.oslo.sysco.no` - `00:21:f6:8d:df:0b`
    - `kubernetes2.oslo.sysco.no` - `00:21:f6:6a:26:77`
    - `kubernetes3.oslo.sysco.no` - `00:21:f6:07:83:83`

- The product_uuid can be checked by using the command `sudo cat /sys/class/dmi/id/product_uuid`. For example the unique product ids in our example are 
    - `kubernetes1.oslo.sysco.no` - `1234EEE2-0002-1000-9481-2C9B3620A4FF`
    - `kubernetes2.oslo.sysco.no` - `1234EEE2-0002-1000-A7D1-ACE30E89E55F`
    - `kubernetes3.oslo.sysco.no` - `1234EEE2-0002-1000-0ED0-F6750310F8CC`

**Note** : The product ids are changed in the above example. You should see the similar pattern for your setup. Its important that these value are unique for each node that forms a cluster.

References:
- [MAC and UUID Spec](https://kubernetes.io/docs/setup/independent/install-kubeadm/#verify-the-mac-address-and-product-uuid-are-unique-for-every-node)

### Configure OS level settings
So far we have verified our hardware, now its time to optimize some OS level settings in order to install kubernetes successfully.

- **Enable br_netfilter Kernel Module**: The br_netfilter module is required for kubernetes installation. This module is required to enable transparent masquerading and to facilitate VxLAN traffic for communication between Kubernetes pods across the cluster. Run the command below to enable the br_netfilter kernel module.
    ```
    modprobe br_netfilter
    echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
    ```
    If you are interested in why it is required, checkout [this](https://github.com/kubernetes/kubernetes/issues/12459) kubernetes issue.


- **Disable SWAP** : We need to disable swap on the servers in order for kubernetes installation to work properly. One of the important kubernetes component that we will install in the next few steps is `kubelet`. In order for kubelet to work properly, the swap should be disabled. Disable the swap on each of the servers by using the following steps.
    - Login via ssh and run ```swapoff -a``` on each host.
    - Edit the `/etc/fstab` file. Run ```vim /etc/fstab``` and comment the swap line UUID
    ```
    # /etc/fstab: static file system information.
    #
    # Use 'blkid' to print the universally unique identifier for a
    # device; this may be used with UUID= as a more robust way to name devices
    # that works even if disks are added and removed. See fstab(5).
    #
    # <file system> <mount point>   <type>  <options>       <dump>  <pass>
    # / was on /dev/nvme0n1p2 during installation
    UUID=47371a4c-3cdf-4ecc-b573-f09a029c74f4 /               ext4    errors=remount-ro 0       1
    # /boot/efi was on /dev/nvme0n1p1 during installation
    UUID=C176-4947  /boot/efi       vfat    umask=0077      0       1
    # swap was on /dev/nvme0n1p3 during installation
    # COMMENT BELOW LINE
    # UUID=78aeec44-b1af-4720-9316-6af68385b23f none            swap    sw              0       0
    ```
    References
    - [why-disable-swap-on-kubernetes](https://serverfault.com/questions/881517/why-disable-swap-on-kubernetes)
    - [kubernetes-issue-7294](https://github.com/kubernetes/kubernetes/issues/7294)


## 2. Install docker, kubernetes and other dependencies.
Now that we have successfully completed Hardware and OS level checks. Its time now to install the required software on our servers. Please go through the instructions below in order to complete software installation.

### Install Docker
We will install docker-ce (community version) for this example. You can check for the various available versions of docker [here](https://docs.docker.com/install/overview/). Also the steps to install docker on different operations systems are described in details [here](https://docs.docker.com/install/linux/docker-ce/centos/). You need to run the below scripts on each of the servers participating in the kubernetes cluster.

- **Uninstall old version**: Older versions of Docker were called docker or docker-engine. If these are installed, uninstall them, along with associated dependencies.
    ```
    sudo yum remove docker \
            docker-client \
            docker-client-latest \
            docker-common \
            docker-latest \
            docker-latest-logrotate \
            docker-logrotate \
            docker-selinux \
            docker-engine-selinux \
            docker-engine
    ```
- **Install docker-ce**: Run the below commands in sequence to install the docker-ce
    ```
    sudo yum install -y yum-utils device-mapper-persistent-data lvm2
    sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    sudo yum install docker-ce
    ```
- **Start docker daemon**: This command will start the docker daemon if it is stopped.
    ```
    sudo systemctl start docker
    ```

### Install Kubernetes components
We will use [kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/) to bootstrap kubernetes cluster for our example. We will start with installing basic kubernetes components namely kubectl, kubeadm and kubelet. We need to run the below scripts on each of our servers.

- **Add PPA repository for kubernetes**: We will first add the ppa repository on each of our servers by running below command
    ```
    cat <<EOF > /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    EOF
    ```
- **Install kubernetes components** : Install the kubernetes binaries kubeadm, kubectl and kubelet using the yum command.
    ```
    yum update -y
    yum install -y kubelet kubeadm kubectl
    ```

- **Restart servers** : Reboot the servers by issuing the below command. This is required to apply and persist all the settings we have done so far.
    ```
    reboot
    ```
- **Restart services**: Login to each of the server and start docker and kubelet services by running the below commands.
    ```
    systemctl start docker && systemctl enable docker
    systemctl start kubelet && systemctl enable kubelet
    ```
- **Change cgroup drivers**: It is important that docker-ce and kubernetes belong to the same cgroup. We can check the docker c-group by issuing the  command `docker info | grep -i cgroup`.

    ```
    kubernetes1 ~]# docker info | grep -i cgroup
    Cgroup Driver: cgroupfs
    ```
    And change the kubernetes cgroup to cgroupfs by issuing the command below
    ```
    sed -i 's/cgroup-driver=systemd/cgroup-driver=cgroupfs/g' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
    ```
Once these steps are complete, reload the systemd service and restart the kubelet service by using the command below. After these services restarted, we are ready to initialize our cluster.
```
systemctl daemon-reload
systemctl restart kubelet
```
## 3. Configure kubernetes master and initialize cluster.

### Initialize master
Login to the server which you have decided to make the master. In our case we want kubernetes1.oslo.sysco.no to act as master so we do an ssh login and run the below command.
```
kubeadm init --apiserver-advertise-address=<server_ip> --pod-network-cidr=10.244.0.0/16
```
Replace the **server_ip** above with the IP of your master. The other supporting cluster services/nodes will connect to this IP while forming the cluster. For example in our case the it looks like 
```
[root@kubernetes1 ~]# kubeadm init --apiserver-advertise-address=192.168.37.48 --pod-network-cidr=10.244.0.0/16
```

Once the command is successful, you will get a token which other nodes will use to join to the cluster. This will be displayed on your terminal and will look something like
```
kubeadm join 192.168.37.48:6443 --token 8hp10q.i4ln2b1ogof374aj --discovery-token-ca-cert-hash sha256:6a59c9b03fa971aef94d61a4f4c1a6b085308f88aab4db1a4affda8d65987867
```
You should carefully save this token. Your Kubernetes master should be initialized successfully by now. 

### Create kube configuration in home directory
To start using your cluster, you need to run the following as a regular user.
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Install flannel network
You must deploy a pod network before anything will actually function properly. We will next, deploy the flannel network to the kubernetes cluster using the kubectl command. To check other networking options that can be used with kubernetes, please have a look [here](https://kubernetes.io/docs/concepts/cluster-administration/networking/).
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
Check if all the components are deployed properly
```
[root@kubernetes1 ~]# kubectl get nodes
NAME                        STATUS     ROLES    AGE     VERSION
kubernetes1.oslo.sysco.no   Ready      master   7m18s   v1.13.1
```
Also check if all required pods are running properly.
```
[root@kubernetes1 ~]# kubectl get pods --all-namespaces
NAMESPACE     NAME                                                READY   STATUS    RESTARTS   AGE
kube-system   coredns-86c58d9df4-kjmxn                            1/1     Running   0          63m
kube-system   coredns-86c58d9df4-lfx4p                            1/1     Running   0          63m
kube-system   etcd-kubernetes1.oslo.sysco.no                      1/1     Running   0          62m
kube-system   kube-apiserver-kubernetes1.oslo.sysco.no            1/1     Running   0          62m
kube-system   kube-controller-manager-kubernetes1.oslo.sysco.no   1/1     Running   0          63m
kube-system   kube-flannel-ds-amd64-mjn4j                         1/1     Running   0          55m
kube-system   kube-proxy-w8rfz                                    1/1     Running   0          55m
kube-system   kube-scheduler-kubernetes1.oslo.sysco.no            1/1     Running   0          62m
```
## 4. Add nodes to the cluster
We have successfully initialized our cluster and in the previous section we have verified that the master node is up and running. It time now to add the nodes `kubernetes2.oslo.sysco.no` and `kubernetes3.oslo.sysco.no` to our cluster. In order to do that, ssh to each of the nodes and run the `kubeadm join` command that we copied earlier.
```
[root@kubernetes2 ~]#  kubeadm join 192.168.37.48:6443 --token 8hp10q.i4ln2b1ogof374aj --discovery-token-ca-cert-hash sha256:6a59c9b03fa971aef94d61a4f4c1a6b085308f88aab4db1a4affda8d65987867
[root@kubernetes3 ~]#  kubeadm join 192.168.37.48:6443 --token 8hp10q.i4ln2b1ogof374aj --discovery-token-ca-cert-hash sha256:6a59c9b03fa971aef94d61a4f4c1a6b085308f88aab4db1a4affda8d65987867
```
Wait for some minutes and login to the master node. Run the below command again to verify if the nodes have joined the cluster.
```
kubectl get nodes
kubectl get pods --all-namespaces

[root@kubernetes1 ~]# kubectl get nodes
NAME                        STATUS   ROLES    AGE     VERSION
kubernetes1.oslo.sysco.no   Ready    master   12m     v1.13.1
kubernetes2.oslo.sysco.no   Ready    <none>   5m37s   v1.13.1
kubernetes3.oslo.sysco.no   Ready    <none>   4m4s    v1.13.1

[root@kubernetes1 ~]# kubectl get pods --all-namespaces
NAMESPACE     NAME                                                READY   STATUS    RESTARTS   AGE
kube-system   coredns-86c58d9df4-kjmxn                            1/1     Running   0          63m
kube-system   coredns-86c58d9df4-lfx4p                            1/1     Running   0          63m
kube-system   etcd-kubernetes1.oslo.sysco.no                      1/1     Running   0          62m
kube-system   kube-apiserver-kubernetes1.oslo.sysco.no            1/1     Running   0          62m
kube-system   kube-controller-manager-kubernetes1.oslo.sysco.no   1/1     Running   0          63m
kube-system   kube-flannel-ds-amd64-gm72r                         1/1     Running   0          56m
kube-system   kube-flannel-ds-amd64-mjn4j                         1/1     Running   0          55m
kube-system   kube-flannel-ds-amd64-nhz9b                         1/1     Running   0          58m
kube-system   kube-proxy-2qqjc                                    1/1     Running   0          56m
kube-system   kube-proxy-l69ld                                    1/1     Running   0          63m
kube-system   kube-proxy-w8rfz                                    1/1     Running   0          55m
kube-system   kube-scheduler-kubernetes1.oslo.sysco.no            1/1     Running   0          62m

```
kubernetes2 and kubernetes3 have been added to the kubernetes cluster.

## 5. Sources and References.

- kubernetes website : [kubernetes.io](https://kubernetes.io/)
- kubernetes with ansible : [k8s-ansible](https://www.digitalocean.com/community/tutorials/how-to-create-a-kubernetes-1-10-cluster-using-kubeadm-on-centos-7)
- kubernetes configuration best practices : [best-practices](https://kubernetes.io/docs/concepts/configuration/overview/)
- using flannel with kubernetes: [k8s-flannel](https://coreos.com/flannel/docs/latest/kubernetes.html)
- this blog post : [centos-k8s](https://www.howtoforge.com/tutorial/centos-kubernetes-docker-cluster/#step-kubernetes-cluster-initialization)