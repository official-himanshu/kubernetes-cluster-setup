# Intial steps to be done on all nodes (master and worker)

### commands
#### Disable swapoff limit on all nodes
	$ swapoff -a
	$ vim /etc/fstab  # remove swap off entry from this file

#### Update the hostname
	$ vim /etc/hostname

#### Note the IP address
	$ ifconfig

#### Update the host files
	$ vim /etc/hosts

#### set static ip address
	$ vim /etc/network/interfaces

	# add following lines
	auto eth1
	iface eth1 inet static
	      address 192.168.33.10
	      netmask 255.255.255.0


#### install openssl-server
	$ sudo apt-get install openssl-server

#### install docker
	$ sudo su 
	$ apt-get update
	$ apt-get install docker.io
	$ systemctl start docker
	$ systemctl enable docker
	$ systemctl status docker

#### Now add the kubernetes apt keys for creating the kubernetes environment
	$ apt-get update && apt-get install -y apt-transport-https curl
	$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
	$ cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
	deb http://apt.kubernetes.io/ kubernetes-xenial main
	EOF
	$ apt-get update

#### Install kubeadm, kubectl and kubelet
	$ apt-get install -y kubeadm kubectl kubelet

#### Update the kubernetes configuration
	$ vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

	# Add the following line
	Environment="cgroup-driver=systemd/cgroup-driver=cgroupfs"

## To be done on master only
#### Initiate the cluster
	#### Using calico pod network here so cidr block is 192.168.0.0/16
	#### If you are using flunnel pod network then cidr block is 10.240.0.0/16
	$ sudo kubeadm init --pod-network-cidr= --apiserver-advertise-address=192.168.33.10

### Bring up the pod network using calico CNI
	$ kubectl apply -f https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/kubeadm/1.7/calico.yml



## To be perform on nodes

	$ kubeadm join 192.168.33.10:6443 --token w195lj.5kkn83dmhai0gl8d \
>     --discovery-token-ca-cert-hash sha256:e42dfaba10674f4bfd88feb8dbd05f9cfde2ddef0bf5e626fb783e85478925f4
