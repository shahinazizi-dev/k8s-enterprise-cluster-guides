# Change Master Nodes and Control Plane Endpoint IP (Loadbalancer/VIP) 

> Environment: 
>> 5 Master Node, 2 Haproxy with Keepalived as VIP 


### Create Backups 

01- PKI 
```
mkdir -p /root/kube-backup/pki/
```
```
cp -a /etc/kubernetes/pki/* /root/kube-backup/pki
```

02- ConfigMaps

```
kubectl get cm -n kube-system kube-proxy -o yaml > kube-proxy.yaml
```

```
kubectl get cm -n kube-system kubelet-conf -o yaml > kubelet-conf.yaml 
```

```
kubectl get cm -n kube-system kubeadm-conf -o yaml > kubeadm-conf.yaml
```
```
kubectl get cm -n kube-public cluster-info -o yaml > cluster-info.yaml
```

03- KubeConfig

```
mkdir /root/kube-backup/kubeconfig
```
```
cp /etc/kubernetes/controller-manager /etc/kubernetes/scheduler /root/kube-backup/kubeconfig /etc/kubernetes/admin.conf
```


04- Manifests

```
mkdir /root/kube-backup/manifests
```
```
cp /etc/kubernetes/manifests/* /root/kube-backup/manifests
```

## Change IP of Master Nodes

01- Write `kubeadm-config.yaml` file with cert SNI (old and new IP)
```
apiVersion: kubeadm.k8s.io/v1beta3
apiServer:
  timeoutForControlPlane: 4m0s
  certSANs:
  - "192.168.0.91"
  - "192.168.0.92"
  - "192.168.0.93"
  - "192.168.0.75"
  - "192.168.0.76"
  - "192.168.0.51"
  - "192.168.0.52"
  - "192.168.0.53"
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: 192.168.0.75:6443
controllerManager: {}
dns: {}
etcd:
  external:
    caFile: /etc/kubernetes/pki/etcd/ca.pem
    certFile: /etc/kubernetes/pki/etcd/etcd.pem
    endpoints:
    - https://192.168.0.81:2379
    - https://192.168.0.82:2379
    - https://192.168.0.83:2379
    keyFile: /etc/kubernetes/pki/etcd/etcd-key.pem
    #imageRepository: opkbhfpskdp0101.p.fnst/kubernetes
kind: ClusterConfiguration
kubernetesVersion: v1.27.13
networking:
  dnsDomain: cluster.local
  podSubnet: 192.168.16.0/20
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

02- Create new `apiserver` certificates

```
cd /etc/kubernetes/pki
```
```
rm -f apiserver.crt apiserver.key apiserver-kubelet-client.crt apiserver-kubelet-client.key
```
```
kubeadm init phase certs apiserver --config ../kubeadm-config.yaml
```
```
kubeadm init phase certs apiserver-kubelet-client --config ../kubeadm-config.yaml
```
03- update DNS records 
```
update /etc/hosts  OR  update DNS record in DnsServer
```

04- Drain Master Node 
```
kubectl drain master01
```

05- Change IP of Control Plane Endpoint 

06- Exit `kube-apiserver.yaml` from `manifests`and Edit that. 

07- Create New Kubeconfig Files 
      
```  
rm -f /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf
```
```
kubeadm init phase kubeconfig controller-manager --config kubeadm-config.yaml
```
```
kubeadm init phase kubeconfig scheduler --config kubeadm-config.yaml
```
08- move `kube-apiserver.yaml` to `manifests` again.

09- Reset  `containerd.service` and `kubelet.service` services ( after that controller-manager and scheduler pods must be restarted )

10- uncordon Master Node 

#### Do 6 steps on every master node 

>> continue IF cluster is in a stable mode

#### Configure HAproxy for new master IP

01- Split HAproxy cluster

> To decrease impact of changing IP on Kubernetes cluster, split HAproxy to two different balancer

1- Set VIP on HA-01 node (all traffic will pass to this node) 

2- change ip HA-02 and Keepalived with new IP range.

02- Backup all pki directory , man ifest directory , kubeconfigs anf configmaps. (on all control plane nodes )
 
03- create exact same `kubeadm-config.yaml` file with new `ControlPlaneEndpoint` IP ( On all master nodes )

```
apiVersion: kubeadm.k8s.io/v1beta3
apiServer:
  timeoutForControlPlane: 4m0s
  certSANs:
   
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: 192.168.0.76:6443
controllerManager: {}
dns: {}
etcd:
  external:
    caFile: /etc/kubernetes/pki/etcd/ca.pem
    certFile: /etc/kubernetes/pki/etcd/etcd.pem
    endpoints:
    - https://192.168.0.81:2379
    - https://192.168.0.82:2379
    - https://192.168.0.83:2379
    keyFile: /etc/kubernetes/pki/etcd/etcd-key.pem
    #imageRepository: opkbhfpskdp0101.p.fnst/kubernetes
kind: ClusterConfiguration
kubernetesVersion: v1.27.13
networking:
  dnsDomain: cluster.local
  podSubnet: 192.168.16.0/20
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

04- Delete all certificate in `pki` folder except `ca.crt` , `ca.key`, `etcd`  ( On all master nodes )

05- Recreate certificates with new config ( On all master nodes )

```
kubeadm init phase certs all --config kubeadm-config.yaml
```

04- Delete all configfiles "schedular.conf" "controller-manager.conf" "admin.conf" "kubelet.conf"  ( On all master nodes )
```
rm -f /etc/kubernetes/controller-manager /etc/kubernetes/scheduler  /etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf
```

06- Recreate kubeconfig files ( On all master nodes )

```
kubeadm init phase kubeconfig controller-manager --config kubeadm-config.yaml
```
```
kubeadm init phase kubeconfig scheduler --config kubeadm-config.yaml
```
```
kubeadm init phase kubeconfig admin --config kubeadm-config.yaml
```
```
kubeadm init phase kubeconfig kubelet --config kubeadm-config.yaml
```

07- Config all `ConfigMaps` with new IP and deploy 
```
cd /root/kube-backup/configmaps
```

08- Create `cluster-info` configmap
```
kubectl delete cm -n kube-public cluster-info 
```
```
kubeadm init phase upload-config all --config /etc/kubernetes/kubeadm-config.yaml
```

09- Restert containerd and kubelet (check for status of manifests) 

10- Rollout `kube-proxy` daemonset 

``` 
kubectl rollout restart daemonset -n kube-system kube-proxy
```
11- Copy new `admin.conf` 

12- On worker node copy `kubelet.conf` and restart kubelet service 

13- Make sure cluster is stable, then config HAproxy cluster 







