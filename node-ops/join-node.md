**** FOR JOINING NEW MASTER NODE ****

/// copy Certificates

Note : Do from any Kubernetes master nodes (Eg: kmaster1)

Copy harbor keys:
scp -r /root/harbor-certs/ root@new-master:/root/harbor-certs
scp /root/harbor-certs/opkbhfpspsp0101.fns.cert  root@new-master:/etc/pki/tls/certs/ ( Old Approach )

Copy etcd keys:
scp -r /etc/kubernetes/pki/etcd root@new-master:/etc/kubernetes/pki/etcd

----------------------------------------------------------------------------------------
/// Create join command 

Note : Do from any Kubernetes master nodes (Eg: kmaster1)

# kubeadm init phase uploa d-certs --upload-certs --config  kubeadminit.yaml
Note : with this command, you will build a certificate key for hoining new master node.


# kubeadm  token create --certificate-key XXXXXXXXXXXXXXXXXXXX --print-join-command

Note: Use the respective kubeadm join commands you copied from the output of kubeadm init command on the first master.

# kubeadm join ...............     --apiserver-advertise-address "NEW-MASTER-IP" 

----------------------------------------------------------------------------------------

**** FOR JOINING NEW WORKER NODE ****

Copy harbor keys:
scp -r /root/harbor-certs/ root@new-master:/root/harbor-certs
scp /root/harbor-certs/opkbhfpspsp0101.fns.cert  root@new-master:/etc/pki/tls/certs/ ( Old Approach )
 
# kubeadm token create --print-join-command

----------------------------------------------------------------------------------------

/// Register access request

Register access request to git
Register access request to previous cluster accessed domains







