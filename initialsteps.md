# Intial steps to be done on all nodes (master and worker)

### commands
#### Disable swapoff limit on all nodes
	$ swapoff -a
	$ vim /etc/fstab  # remove swap off entry from this file

#### Disable SELinux on all nodes
	$ setenforce 0
	$ sed -i 's/enforcing/disabled/g' /etc/selinux/config
	$ grep disabled /etc/selinux/config | grep -v '#' 
	SELINUX=disabled

#### Install docker
	
	$ yum update -y
	$ yum install -y docker

	$ systemctl start docker
	$ systemctl enable docker
	$ systemctl status docker

#### Add kubernetes repo

	$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
	[kubernetes]
	name=kubernetes
	baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-e17-x86_64
	enabled=1
	gpgcheck=1
	repo_gpgcheck=1
	gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
	https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
	exclude=kube*
	EOF

	$ yum install -y kubeadm kubelet kubectl --disableexcludes=kubernetes

	$ systemctl enable kubelet && systemctl start kubelet

##### This step is perform only on centos/RHEL

	$ cat <<EOF > /etc/sysctl.d/k8s.conf
	net.bridge.bridge-nf-call-ip6tables = 1
	net.bridge.bridge-nf-call-iptables = 1
	EOF

	$ sysctl --system
	$ echo '1' > /proc/sys/net/ipv4/ip_forward

### Perform only on master

	$ kubeadm init --pod-network-cidr=10.240.0.0/16

	<<<<<<output>>>>>

	To start using your cluster, you need to run the following as a regular user:
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/
Then you can join any number of worker nodes by running the following on each as root:
kubeadm join 10.128.15.216:6443 --token fh279b.j40uqu3mcsve3pna \
    --discovery-token-ca-cert-hash sha256:072a885ceb37d32a6e62a790de39b1a72377c2aa6d2abbd8553a498ca3585173 

    ## configuring pod network plugin

    $ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml

### perform this step on worker node

kubeadm join 10.128.15.216:6443 --token fh279b.j40uqu3mcsve3pna \
    --discovery-token-ca-cert-hash sha256:072a885ceb37d32a6e62a790de39b1a72377c2aa6d2abbd8553a498ca3585173 

#### tip
To create token again use below command

kubeadm token create --print-join-command