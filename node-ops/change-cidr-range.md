
/// create n new ippool. CRD

Note : When we want to create a new CRD for Calico, we should first install the namespace `calico-apiserver`.  it will install by caloco "custome-resorce.yaml" by default. 

# k describe ippools. default-ipv4-ippool  -o yaml > newIppool

# vim newIppool

Note : edit newIppool according your desire configuration.

-------------------------------------------------------------------

Note: The important point is that when you create a new IP pool and disable the old one, it will not impact your running pods and services; they will continue to function properly as they did before.
Now, if you run a new container in the cluster, it will be assigned an IP from the new IP pool.

-------------------------------------------------------------------

/// disable old ippool

# k edit ippool. oldippool

  then insert " disabled: true " under " cidr: " line
  
-------------------------------------------------------------------

/// Change node cidr  ( Please perform the following actions on all master and worker nodes. ) 

# k get nodes opkbmfpskst0101 -o yaml > masternode01.yaml
# vim masternode01.yaml
 Note : edit "PodCIDR:" and "PodCIDRs:" according your desire configuration.

#k delete nodes opkbwfpskst0101 && k create -f masternode01.yaml

Note : we will not have any problem with this action because the current pods will work properly with the old IPs.

-------------------------------------------------------------------

/// edit configMaps :

# k edit cm -n kube-system kubeadm-config
  Node : edit " podSubnet "
  
# k edit cm -n kube-system kube-proxy
  Node : edit " clusterCIDR "
  
-------------------------------------------------------------------

/// edit contreller manager manifest ( on all master nodes )

# vim /etc/kubernetes/manifest/kube-controler-manager.yaml
  Node : edit " cluster-cidr "

-------------------------------------------------------------------
/// restart "controler-manager" & "kube-proxy" pod 

# systemctl restart containerd.service
  
-------------------------------------------------------------------

/// delete metricserver and coredns pods in namespace " kube-system " to run with new ip add.

-------------------------------------------------------------------
