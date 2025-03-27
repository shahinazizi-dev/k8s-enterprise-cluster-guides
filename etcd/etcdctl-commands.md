
ETCDCTL_API=3 etcdctl member list --write-out=table --endpoints=https://192.168.0.81:2379 --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/pki/etcd-cluster.pem --key=/etc/etcd/pki/etcd-cluster-key.pem

-------------------------------------------------------------------------------------------------------------------

ETCDCTL_API=3 etcdctl endpoint status --write-out=table --endpoints=https://192.168.0.81:2379 --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/pki/etcd-cluster.pem --key=/etc/etcd/pki/etcd-cluster-key.pem

-------------------------------------------------------------------------------------------------------------------

ETCDCTL_API=3 etcdctl endpoint health --write-out=table --endpoints=https://192.168.0.71:2379,https://192.168.0.72:2379,https://192.168.0.73:2379 --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/pki/etcd.pem --key=/etc/etcd/pki/etcd-key.pem

-------------------------------------------------------------------------------------------------------------------

ETCDCTL_API=3 etcdctl member add etcd3 --peer-urls=https://192.168.0.83:2380 --endpoints=https://192.168.0.81:2379,https://192.168.0.83:2379 --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/pki/etcd-cluster.pem --key=/etc/etcd/pki/etcd-cluster-key.pem

-------------------------------------------------------------------------------------------------------------------

ETCDCTL_API=3 etcdctl member remove d21739fce68087be --endpoints=https://192.168.0.81:2379,https://192.168.0.82:2379 --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/pki/etcd-cluster.pem --key=/etc/etcd/pki/etcd-cluster-key.pem

-------------------------------------------------------------------------------------------------------------------

/// Reset ETCD nodes: (execute on first etcd node )

   #ETCDCTL_API=3 etcdctl --endpoints=https://0.0.0.0:2379,https://1.1.1.1:2379,https://2.2.2.2:2379
    --cacert=/etc/etcd/pki/ca.pem  --cert=/etc/etcd/pki/ca.pem --key=/etc/etcd/pki/ca.pem dell "" --prefix
   
-------------------------------------------------------------------------------------------------------------------
   
/// ETCD member lists : 

   #ETCDCTL_API=3 etcdctl --endpoints=https://0.0.0.0:2379,https://1.1.1.1:2379,https://2.2.2.2:2379 
    --cacert=/etc/etcd/pki/ca.pem  --cert=/etc/etcd/pki/ca.pem --key=/etc/etcd/pki/ca.pem  member list
   
------------------------------------------------------------------------------------------------------------------


    



 
