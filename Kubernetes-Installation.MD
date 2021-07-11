# Kubernetes Cluster installation using kubeadm
Follow this documentation to set up a Kubernetes cluster on __CentOS 7__ Virtual machines.

This documentation guides you in setting up a cluster with one master node and two worker nodes.

Master: t2.medium
Worker Nodes: t2.micro

# Ports require for kubernetes master
```sh
6443
32750
10250
4443
443
8080
179 --> Should be allowed for Master and Worker nodes. 
```

## On both Kmaster and Kworker Nodes
Perform all the commands as root user unless otherwise specified

##### Install, enable and start docker service
Use the Docker repository to install docker.
> If you use docker from CentOS OS repository, the docker version might be old to work with Kubernetes v1.13.0 and above
```sh
yum install -y -q yum-utils device-mapper-persistent-data lvm2 > /dev/null 2>&1
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo > /dev/null 2>&1
yum install -y -q docker-ce >/dev/null 2>&1
```

##### Docker start

```sh
systemctl enable docker
systemctl start docker && systemctl status docker
```
##### Disable SELinux
```sh
setenforce 0
sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
```
##### Disable Firewall
```sh
systemctl disable firewalld
systemctl stop firewalld
```
##### Disable swap
```sh
sed -i '/swap/d' /etc/fstab
swapoff -a
```
##### Update sysctl settings for Kubernetes networking
```sh
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
### Kubernetes Setup
##### Add yum repository
```sh
cat >>/etc/yum.repos.d/kubernetes.repo<<EOF
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
##### Install Kubernetes
```sh
yum install -y kubeadm-1.15.6-0.x86_64 kubelet-1.15.6-0.x86_64 kubectl-1.15.6-0.x86_64
```
##### Enable and Start kubelet service
```sh
systemctl enable kubelet
systemctl start kubelet
```
## On kmaster
##### Initialize Kubernetes Cluster
```sh
kubeadm init --apiserver-advertise-address=<MasterServerIP> --pod-network-cidr=192.168.0.0/16
```
##### Copy kube config
To be able to use kubectl command to connect and interact with the cluster, the user needs kube config file.

### Add user
```sh
useradd kubeadmin
passwd kubeadmin
```


```sh
mkdir /home/kubeadmin/.kube
cp /etc/kubernetes/admin.conf /home/kubeadmin/.kube/config
chown -R kubeadmin:kubeadmin /home/kubeadmin/.kube
chmod -R 755 /home/kubeadmin/.kube/
```

```sh
yum -y install wget
```


##### Deploy Calico network / establish networking
This has to be done as the user in the above step (in my case it is __kubeadmin__)
```sh
wget https://docs.projectcalico.org/v3.9/manifests/calico.yaml
kubectl create -f calico.yaml
```

##### Cluster join command
```sh
kubeadm token create --print-join-command
```
## On Kworker
##### Join the cluster
Use the output from __kubeadm token create__ command in previous step from the master server and run here.

## Verifying the cluster
##### Get Nodes status
```sh
kubectl get nodes
```
##### Get component status
```sh
kubectl get cs
```
