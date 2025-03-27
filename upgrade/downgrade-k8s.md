**** Do the following steps on first control plane node. ****

------------------------------------------------------------------------------------------------

  /// create backup  

 create backup from  " /etc/kubernetes/ "
 ------------------------------------------------------------------------------------------------
 
/// pull images to harbor

------------------------------------------------------------------------------------------------

/// download and insert below files to every host :
    containerd - runc - cni

------------------------------------------------------------------------------------------------

 /// kordon node
 
------------------------------------------------------------------------------------------------
 
/// Checking the compatibility between containerd, runc and shim with the new version of Kubernetes.
If necessary, you must downgrade them.

 /// upgrade conntainerd to new version

 stop & disable conntainerd.service

 install new version of conntainerd.service

 prepare containerd config file ( config.toml )
 
 chown -R root. /etc/containerd

------------------------------------------------------------------------------------------------

 /// upgrade runc

 download and transfer new version

 install -m 755 runc.amd64 /usr/local/sbin/runc

------------------------------------------------------------------------------------------------

 /// upgrade cni plugin
 
  mkdir -p /opt/cni/bin
  tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.t
------------------------------------------------------------------------------------------------

 systemctl daemon-reload

 systemctl enable --now containerd.service

 systemctl restart containerd
 
------------------------------------------------------------------------------------------------

  IMPORTANT :  edit pause image version in containerd configuration file to new version of pause image version.
  
-----------------------------------------------------------------------------------------------
/// Downgrade kubeadm , kubelet and kubectl to the desired version:

Note : update kubernetes repository if necessary.
# yum install kubelet=1.27.2  kubeadm=1.27.2  kubectl=1.27.2

----------------------------------------
/// Downgrade Cluster :

# kubeadm upgrade plan

# kubeadm upgrade apply v1.27.2

----------------------------------------
# systemctl restart containerd
# systemctl restart kubelet.service

----------------------------------------
Note: Perform all the steps for the other nodes one by one.

Note: in the "Downgrade Cluster" step , use following command:

# kubeadm upgrade node

----------------------------------------
