
///  Creating a Certificate Authority (CA) = Root CA on ETCD node.

cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "expiry": "8760h",
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "My own CA",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IR",
      "O": "Kubernetes",
      "OU": "ETCD-CA",
      "ST": "Teh",
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca
}

-----------------------------------------------------------------------------------------------------------------------------------

///  Generating Certificates for etcd Nodes on ETCD node.

Note : add both old and new ips 

ETCD_HOSTNAMES=*.etcd.local
ETCD_IP=127.0.0.1,192.168.100.100,192.168.100.100,192.168.100.100,192.168.100.100,192.168.100.100,192.168.100.100

cat > etcd-csr.json <<EOF
{
  "CN": "*.etcd.local",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IR",
      "O": "Kubernetes",
      "OU": "ETCD",
      "ST": "Teh"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${ETCD_IP},${ETCD_HOSTNAMES} \
  -profile=kubernetes \
  etcd-csr.json | cfssljson -bare etcd-cluster
  
-----------------------------------------------------------------------------------------------------------------------------------

/// Examining the IP addresses in the certificate.

    openssl x509 -in etcd.pem -text
    
-----------------------------------------------------------------------------------------------------------------------------------    

/// create backup

  create a backup from all etcd kyes ( pki derectory ) in etcd nodes and control plane nodes.

-----------------------------------------------------------------------------------------------------------------------------------

/// copy new ca.*(4 files) , etcd.*(4 files) to all etcd nodes.

 repetable scp command : 

 # for i in 'seq 71 73' ; do scp ca.pem root@192.168.0.$i:/home/mydoc/; done
 
  Note : Do chown root:root for these files.
    
  Note : With this copy and without reloading the service, we will not encounter any problems.
-----------------------------------------------------------------------------------------------------------------------------------    

/// copy new ca.pem , etcd.pem , etcd-key.pem to all control plane nodes.

 repetable scp command : 

 # for i in 'seq 51 53' ; do scp ca.pem root@192.168.0.$i:/home/mydoc/; done
 
  Note : Do chown root:root for these files.
 
  Note : With this copy and without reloading the service, we will not encounter any problems.
  
  Note : until havenot been hanshake between masters and etcd nodes, everything is good.
  
   
 -----------------------------------------------------------------------------------------------------------------------------------

 /// edit etcd unit on all etcd nodes.
 
 #vim /etc/systemd/system/etcd.service
 replace new cetfificates and keys with old ones.
 until havenot been hanshake between masters and etcd nodes, everything is good.
  
  ----------------------------------------------------------------------------------------------------------------------------------
 
  /// replace new etcd keys with old ones to all etcd nodes.
  
  # cp ca.pem etcd.pem etcd-key.pem /etc/etcd/pki/
 
 -----------------------------------------------------------------------------------------------------------------------------------
 
 /// reload etcd.service on all etcd nodes after replacing
 
 # systemctl restart etcd.service
 
 and then check cluster status
 
 -----------------------------------------------------------------------------------------------------------------------------------
  
  /// replace new keys with old ones to all control pane nodes.
  
  # cp ca.pem etcd.pem etcd-key.pem /etc/kubernetes/pki/etcd/
 
  replace new cetfificates and keys with old ones.
  
  and then check cluster status

 
 ------------------------------------------------------------------------------------------------------------------------------------



 
                                                                 (((((((( start changing ip add  ))))))))))


Note : do not start with Leader Node


------------------------------------------------------------------------------------------------------------------------------------

 #systemctl stop etcd.service ( on selected etcd nodes )
 
 -----------------------------------------------------------------------------------------------------------------------------------
 
//remove one etcd node (from another node)

 # ETCDCTL_API=3 etcdctl member remove d21739fce68087be --endpoints=https://192.168.100.100:2379,https://192.168.100.100:2379 --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/
   pki/etcd-cluster.pem --key=/etc/etcd/pki/etcd-cluster-key.pem

 # rm -rf /data/*

-----------------------------------------------------------------------------------------------------------------------------------
 
/// changing etcd node ip 

 # nmtui ...
 # systemctl restart NetworkManager
 # nmcli device reapply ens06
 
-----------------------------------------------------------------------------------------------------------------------------------

 /// update etcd.service on has changed etcd node

 # vim /etc/systemd/system/etcd.service
 
 1 - change old ip to new one
 
 2 - change "initial-cluster-state" from "new" to "existing"
 
 Note : without reloading the service, we will not encounter any problems.
 
-----------------------------------------------------------------------------------------------------------------------------------

 /// update etcd.service on the other etcd nodes.

 # vim /etc/systemd/system/etcd.service

 1 - "initial-cluster-state" section (changing old ip to new one)

 2 - change "initial-cluster-state" from "new" to "existing"
 
 Note : without reloading the service, we will not encounter any problems.

-----------------------------------------------------------------------------------------------------------------------------------

 ///  add changed ip etcd node to etcd cluster ( from another etcd nodes )

  Note : Utilize the following command from other nodes

  ETCDCTL_API=3 etcdctl member add etcd3 --peer-urls=https://192.168.100.100:2380 --endpoints=https://192.168.100.100:2379,https://192.168.100.100:2379
  --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/pki/etcd-cluster.pem --key=/etc/etcd/pki/etcd-cluster-key.pem

-----------------------------------------------------------------------------------------------------------------------------------

/// start etcd.service on selected etcd node

# systemctl daemon reload
# systemctl start etcd.service  
# systemctl restart etcd.service

-----------------------------------------------------------------------------------------------------------------------------------

/// applay changes in etcd unit

Do these on all the nodes ( by one by )

# systemctl daemon reload 
# systemctl restart etcd.service

After each node, test 'node health,' and then proceed to the next node

ETCDCTL_API=3 etcdctl endpoint health --write-out=table --endpoints=https://192.168.100.100:2379,https://192.168.100.100:2379,https://192.168.100.100:2379 --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/pki/etcd.pem --key=/etc/etcd/pki/etcd-key.pem

-----------------------------------------------------------------------------------------------------------------------------------

/// checking nodes satate

Use the 'endpoint status' to verify that the node has been added

ETCDCTL_API=3 etcdctl endpoint status --write-out=table --endpoints=https://192.168.100.100:2379 --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/pki/etcd-cluster.pem --key=/etc/etcd/pki/etcd-cluster-key.pem

-----------------------------------------------------------------------------------------------------------------------------------

/// update kube-apiserver.yaml on all control planes

 Update the node's information with the changed IP addresses, starting from the last master node to the first node.

 # kube-apiserver.yaml

-----------------------------------------------------------------------------------------------------------------------------------

 /// verify remove and add apiserver's pod after edit in "kube-apiserver.yaml" ( specially for a single node)

After editing the 'kube-apiserver.yaml' file, verify the removal and addition of the apiserver pod, particularly for a single node.

# watch kebectl get pods -n kube-system

-----------------------------------------------------------------------------------------------------------------------------------

   - Repeat the last two previous steps for the other master nodes.

-----------------------------------------------------------------------------------------------------------------------------------


































