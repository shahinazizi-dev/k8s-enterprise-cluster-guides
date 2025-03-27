
/// etcd keys on master nodes

ls  /etc/kubernetes/pki/etcd/

====================

/// to see inserted kernel Modules

cat/etc/modules-load.d/k8s.conf


/// checking current state of kernel Modules 

lsmod | grep br_netfilter
lsmod | grep overlay


====================

/// kernel parameters status

cat /etc/sysctl.d/k8s.conf  (net bridge)

====================

/// adding kubernetes repo : 

ll /etc/yum.repos.d/

====================

/// cheking default paths 

adding Dir /usr/local/bin/ to default paths 

====================

/// cheking containerd configfile

 cat /etc/containerd/config.toml

====================






























