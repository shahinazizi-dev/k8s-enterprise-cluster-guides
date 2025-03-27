
**** ON ALL MASTER AND WORKER NODES ****


/// Download and insert below files to every host :
    containerd - runc - cni plugin

====================

/// Configure Kernel Modules

cat << EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

    OR   

modprobe overlay
modprobe br_netfilter

### checking status :

lsmod | grep br_netfilter
lsmod | grep overlay

====================

/// Configure Kernel parameters


# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
fs.inotify.max_user_watches = 1048576
EOF

# Apply sysctl params without reboot
sudo sysctl --system

### checking status :
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward fs.inotify.max_user_watches

====================

/// disable swap

vim /etc/fstab and and Delete Swap line
swapoff -a

====================

/// disable firewall

systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl status firewalld.service

====================

/// disable selinux 

setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=Disabled/' /etc/selinux/config

# Check Result
getenforce

====================

/// harbor certs

mkdir /root/harbor-certs/

copy harbor keys from harbor machine (/etc/docker/cert.d/nodeX/ ) to each master and worker node in  dir /root/harbor-certs/
               ca.cert
	       nodeX .crt
               nodeX .cert
               nodeX .key

====================

/// adding default path

adding Dir /usr/local/bin/ to default paths  >> export PATH="/usr/local/bin/:$PATH"  &  add line    export PATH=$PATH:/usr/local/bin/   in .bashrc file

====================

///  Install and Configure containerd

download contained binary file

tar -xzvf containerd-1.6.2-linux-amd64.tar.gz -C /usr/local 

Download or create file "containerd.services" (From other master or worker nodes in another cluster) in Dir /etc/systemd/system/containerd.service 

mkdir -p /etc/containerd

containerd config default | sudo tee /etc/containerd/config.toml

edit /etc/containerd/config.toml aconding to another config files at the same enviroment
 

             - disabled_plugins = []  >  #disabled_plugins = []
             - SystemdCgroup = true
             
             - correct pause's image version > sandbox_image = "registry.local/kubernetes/pause:3.9"
             
             - insert Repository (harbor) address - registry.local/kubernetes 
             - insert harbor's auth key from opkbhfpskup0101.p.fnst/root/.docker/config.json
             - adding harbor's Kyes ( Certificates ) to /root/harbor-certs/
               ca.cert
               nodeX .cert
               nodeX .key

systemctl daemon-reload

systemctl enable --now containerd.service

systemctl restart containerd

systemctl status containerd


scp /etc/containerd/config.toml to all control planes

====================

 /// set the default  endpoint  for containerd :
 
  vim /etc/crictl.yaml  
        runtime-endpoint: unix:///run/containerd/containerd.sock
        
====================

///  Installing runc

download runc file

install -m 755 runc.amd64 /usr/local/sbin/runc

====================

/// Installing CNI plugins

download CNI binary file

mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.t

====================

/// adding kubernetes repo : 

cat <<EOF > /etc/yum.repos.d/kubernetes.repo	
[kubernetes]
name=Kubernetes
baseurl=http://pkgs.k8s.io/core:/stable:/v1.28/rpm/
enabled=1
gpgcheck=1
gpgkey=http://pkgs.k8s.io/core:/stable:/v1.28/rpm/repodata/repomd.xml.key
#exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

====================

/// update repository

 yum makecache

====================

/// copy etcd cluster keys to master nodes only

mkdir -P /etc/kubernetes/pki/etcd/

Copy the following files ( etcd cluster keys ) from any etcd node in the cluster to the first control plane node:

scp /etc/etcd/pki/ca.pem         CONTROL_PLANE_IP:/etc/kubernetes/pki/etcd/
scp /etc/etcd/pki/etcd.pem       CONTROL_PLANE_IP:/etc/kubernetes/pki/etcd/
scp /etc/etcd/pki/etcd-key.pem   CONTROL_PLANE_IP:/etc/kubernetes/pki/etcd/

after copy , change file owner to root user

=======================================================================================================================================================

sudo yum install  kubelet kubeadm kubectl  (in  appropriate version )

=======================================================================================================================================================

sudo systemctl enable --now kubelet.service

====================
   
   **** ONLY ON FIRST MASTER NODE ****
   
====================

pull kubernetes images from repository (harbor)

registry.k8s.io/coredns/coredns           
registry.k8s.io/coredns                 
registry.k8s.io/conformance               
registry.k8s.io/kube-apiserver            
registry.k8s.io/kube-controller-manager   
registry.k8s.io/kube-proxy                
registry.k8s.io/kube-scheduler            
registry.k8s.io/pause  

/// prepare VIPs

====================

yum install iproute-tc

Note : if this package does not install, kubelet will not run

====================

Create "kubeadm-config.yaml" file

====================

kubeadm init --config kubeadm-config.yaml --upload-certs

====================

/// install and configure calico and tigera operator

after install kubernetes on the first master node,before join next master node , you should install calico network.

download calico images an insert in repository ( harbor )

instal calico on first control palne node

====================

edit controll-managetr for promerthious

====================

/// joining Second Master Node and go on

the control-plane node joining command :

  kubeadm join 10.200.200.200:6443 --token 8kc9in.zd83ms9jngjvugcu \
        --discovery-token-ca-cert-hash sha256:ad5067cedc2a4a3b06f30917c568b17535626eaac14020144495a4d2b061a888 \
        --control-plane --certificate-key d46969e6a87b67b6a41d74436fbf791c1614c41920fa4dc5bcd839f9c634b0a6

the worker node joining command :

kubeadm join 10.200.200.200:6443 --token 8kc9in.zd83ms9jngjvugcu \
        --discovery-token-ca-cert-hash sha256:ad5067cedc2a4a3b06f30917c568b17535626eaac14020144495a4d2b061a888


====================

/// joining Workes Nodes

====================
\@
/// install metric server

====================
