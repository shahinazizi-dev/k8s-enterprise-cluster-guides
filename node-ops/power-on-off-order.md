Power off order :

1 – Ceph nodes 

2 – Harbor Node 

3 – Etcd nodes 

4 – haproxy - keepalive Nodes 

5 – Master Nodes 

6 – Worker Nodes 

Note that : you should stop and disable etcd.service before powering off etcd nodes.  


Power on order :

1 – Ceph nodes 

2 – Harbor Node 

3 – Etcd nodes 

4 – haproxy - keepalive Nodes 

5 – Master Nodes 

6 – Worker Nodes 

Note that : After turning on etcd nodes, start and enable etcd.service on all etcd nodes. 
