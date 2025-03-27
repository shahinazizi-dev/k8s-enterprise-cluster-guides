 /// create backup  

 create backup from  " /etc/kubernetes/ "
 
 ------------------------------------------------------------------------------------------------

/// pull images to harbor

------------------------------------------------------------------------------------------------

/// download and insert below files to every host :
    containerd - runc - cni

------------------------------------------------------------------------------------------------

/// kordon nodes ( one by one ) 

------------------------------------------------------------------------------------------------

 /// upgrade conntainerd to new version

 stop & disable conntainerd.service

 install new version of conntainerd.service ( with no drain or cordon node ??? ) in all nodes.

 tar Cxzvf /usr/local containerd

 prepare containerd config file ( config.toml )
 
 chown -R root. /etc/containerd
   
------------------------------------------------------------------------------------------------

  IMPORTANT :  edit pause image version in containerd configuration file to new version of pause image version.
  
-----------------------------------------------------------------------------------------------

 /// upgrade runc

 install -m 755 runc.amd64 /usr/local/sbin/runc

------------------------------------------------------------------------------------------------

 /// upgrade cni plugin
 
  mkdir -p /opt/cni/bin
  tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.t
------------------------------------------------------------------------------------------------
   
 /// transfer harbor keys to all nodes (if needed)
 
 copy harbor keys from harbor machine to all nodes in  dir /root/harbor-certs/
 
 mkdir /root/harbor-certs/
 
 ------------------------------------------------------------------------------------------------

   systemctl daemon-reload

   systemctl enable --now containerd.service

   systemctl restart containerd

   systemctl status containerd

------------------------------------------------------------------------------------------------   

 /// update repository with kubernetes new version on first master node : 
	
cat <<EOF > /etc/yum.repos.d/kubernetes.repo	
[kubernetes]
name=Kubernetes
baseurl=http://pkgs.k8s.io/core:/stable:/v1.29/rpm/
enabled=1
gpgcheck=1
gpgkey=http://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
#exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF
------------------------------------------------------------------------------------------------
 ///upgrade kubelet & kubectl & kubeadm
 
	# yum install kubectl-1.29.7 kubelet-1.29.7 kubeadm-1.29.7  ( with no drain or cordon node ??? )
------------------------------------------------------------------------------------------------
 ###### only in  master nodes ####
 
 vim /etc/kuberneters/kubeadminit.yaml (for hint only )
 
 edit kubernetes current version with desired version
 
------------------------------------------------------------------------------------------------
	
  # kubeadm upgrade node  ( for worker node or the second or further master node )
  Note : check the pods in ns kube-system for create new pods for components
	
 	# systemctl restart kubelet
	
	# systemctl status kubelet
  
  # systemctl restart containerd
  	
  # systemctl status containerd

------------------------------------------------------------------------------------------------
 6 - in all master nodes ( for prometiuos monitoring )   
        
       vim  kube-controller-manager.yaml  :  --bind-address=127.0.0.1 >> 0.0.0.0
       vim  kube-scheduler.yaml           : --bind-address=127.0.0.1 >> 0.0.0.0
       
------------------------------------------------------------------------------------------------
         
         
         
         
         
         
         
         
         
         
         
         
         
        
        
