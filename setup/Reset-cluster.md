
  /// on master & worker node:
  
 # systemctl stop containerd.service
 
 # systemctl stop kubelet.service

 # kubeadm reset :

 # rm -rf $HOME/.kube/

 # rm -rf /var/lib/kubelet/
 
 # rm -rf /etcd/cni/net.d

   Note: the etcd keys will removed, for install kubernetes again , we must insert etcd keys on the master node again 
   
 -------------------------------------------------------------------------------------------------------------------------
 
   /// on ETDC node:
   
   - Reset ETCD nodes: (execute on first etcd node )

   #ETCDCTL_API=3 etcdctl --endpoints=https://0.0.0.0:2379,https://1.1.1.1:2379,https://2.2.2.2:2379  --cacert=/etc/etcd/pki/ca.pem  --cert=/etc/etcd/pki/ca.pem 
    --key=/etc/etcd/pki/ca.pem dell "" --prefix
    
Note: when you have a etcd cluster and reset master node with " #kubeadm reset" , any object from etcd cluster will note removed automatically an you must reset etcd nodes manually.
if you dont, when you want to reinitialize kubenrnetes cluster and use " #kubeadm init" command, etcd nodes dont identify that you your request in new and then you will have some problem. 
so after reset master node with " #kubeadm reset" , you must reset all etcd nodes too.

---------------------------------------------------------------------------------------------------------------------------

