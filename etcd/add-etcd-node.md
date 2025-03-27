
/// disable firewall

-----------------------------------------------------------------------------------------------------------------------------------

/// disable selinux

 -----------------------------------------------------------------------------------------------------------------------------------
/// add /usr/local/bin/ to default paths

 # vim /etc/profile

 insert line at the end of file :  export PATH=$PATH:/usr/local/bin/

-----------------------------------------------------------------------------------------------------------------------------------

# mkdir -p /etc/etcd/pki/

-----------------------------------------------------------------------------------------------------------------------------------

///copy etcd keys from curent etcd nodes to new ones.

 # scp etcd.pem etcd-key.pem ca.pem to shazizi@192.168.0.0:/home/username/ ( from one of curent etcd nodes )
 
 # chown root. /home/username/*
 
 # mv /home/username/* /etc/etcd/pki/
 
 -----------------------------------------------------------------------------------------------------------------------------------
 
 /// install etcd
 
  # scp /usr/local/bin/etcd /usr/local/bin/etcdctl  shazizi@192.168.0.0:/home/username/ ( from one of curent etcd nodes )
  
  # chown root. /home/username/*
 
  # vim 
 
  # mv /home/username/* /usr/local/bin/
  
  OR  
  
  install etcd binary file.
 
 -----------------------------------------------------------------------------------------------------------------------------------
 
 /// create etcd.service unit
 
 # scp /etc/systemd/system/etcd.service/  shazizi@192.168.0.0:/home/username/ ( from one of curent etcd nodes )
 
 # vim /home/username/etcd.service/ 
 
  - edit IP & Hostname of selected node
  - add only selected node data in "initial_cluster"
  - change "initial_cluster_status" from new to existing
 
 # chown root. /home/username/etcd.service
 
 # mv /home/username/etcd.service /etc/systemd/system/
 
 -----------------------------------------------------------------------------------------------------------------------------------

  /// add new npde to cluster  ( from one of curent etcd nodes )
  
  # ETCDCTL_API=3 etcdctl member add opkb... --peer-urls=https://192.168.0.0:2380 --endpoints=https://192.168.0.0:2379,https://192.168.0.0:2379
   --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/pki/etcd.pem --key=/etc/etcd/pki/etcd-key.pem
   
 -----------------------------------------------------------------------------------------------------------------------------------
   
  /// on selected node
  
  #systemctl daemon reload
  #systemctl restart etcd.service
  #systemctl status etcd.service 
  
  -----------------------------------------------------------------------------------------------------------------------------------
  
  
 /// ckecking status with ETCD member lists : 

   #ETCDCTL_API=3 etcdctl --endpoints=https://0.0.0.0:2379,https://1.1.1.1:2379,https://2.2.2.2:2379 
    --cacert=/etc/etcd/pki/ca.pem  --cert=/etc/etcd/pki/ca.pem --key=/etc/etcd/pki/ca.pem  member list
    
  -----------------------------------------------------------------------------------------------------------------------------------
  
   /// ckecking status with ETCD member lists : 
  
  #ETCDCTL_API=3 etcdctl endpoint status --write-out=table --endpoints=https://192.168.0.0:2379,https://192.168.0.0:2379 --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/pki/etcd-cluster.pem --key=/etc/etcd/pki/etcd-cluster-key.pem
  
  -----------------------------------------------------------------------------------------------------------------------------------    
  
  
  
  
  
  
  
  
  
  
  
  
  
 
