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

/// install ETCD with binary file ( Do these on all etcd nodes ) :

Download etcd binary file:

#wget https://github.com/etcd-io/etcd/releases/download/v3.5.16/etcd-v3.5.16-linux-amd64.tar.gz .
#tar xvf etcd-v3.5.15-linux-amd64.tar.gz
#cd etcd-v3.5.15-linux-amd64/
#cp etcd* /usr/local/bin/
#etcdctl version

====================

/// create etcd.service ( Do on all etcd nodes )

# vim etcd.service
Note: you can copy this file from current etcd node in other etcd cluster.

Note : you must customise etcd.service with your own configuration.

====================
/// start etcd.service
Note : please perform the steps below starting from the first node and continuing node by node, separately.

# systemctl daemon-relod
# systemctl enable etcd.service
# systemctl start etcd.service

====================

**** Generate SSLs for ETCD nodes ****

openssl genrsa -out ca.key 2048

====================

openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr

====================

openssl x509 -req -in ca.csr -signkey ca.key -CAcreateserial -out ca.crt -days 1000

====================

openssl genrsa -out etcd-server.key 2048

====================

vim openssl-etcd.cnf
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
IP.1 = 192.168.100.100
IP.2 = 192.168.100.100
IP.3 = 192.168.100.100
IP.4 = 127.0.0.1

====================

openssl req -new -key etcd-server.key -subj "/CN=etcd-server" -out etcd-server.csr -config openssl-etcd.cnf

====================

openssl x509 -req -in etcd-server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out etcd-server.crt -extensions v3_req -extfile openssl-etcd.cnf -days 1000

====================

# mkdir /etc/etcd/pki
# mv ca.key ca.crt etcd-key.key etcd.crt /etc/etcd/pki/

====================

/// copy all certificates from etcd node01 to all etcd nodes.

#scp /etc/etcd/pki/  user@0.0.0.0.:/etc/etcd/pki/

====================

**** Generate SSLs for Master nodes ****

openssl genrsa -out apiserver-server.key 2048

====================

openssl req -new -key apiserver-server.key -subj "/CN=apiserver-server" -out apiserver-server.csr

====================

openssl x509 -req -in apiserver-server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out apiserver-server.crt -days 1000

====================  

/// copy api server certificates from etcd node01 to all master nodes.
Do this for each master node

#scp ca.crt apiserver.crt apiserver-key.key user@0.0.0.0.:/etc/kubernetes/pki/

====================

/// copy etcd server certificates from etcd node01 to all master nodes.
Do this for each master node

#scp ca.crt etcd.crt etcd-key.key user@0.0.0.0.:/etc/kubernetes/pki/etcd/


====================
/// Checking the status : 

ETCDCTL_API=3 /usr/local/bin/etcdctl --endpoints=https://10.200.200.200:2379,https://10.200.200.200:2379,https://10.200.200.200:2379 --cacert=/etc/etcd/pki/ca.crt --cert=/etc/etcd/pki/etcd-server.crt --key=/etc/etcd/pki/etcd-server.key  -w table endpoint status

====================


