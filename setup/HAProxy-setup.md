
                                     ***** YOU CAN DO ALL STEPS IN PARALLEL ON ALL HA PROXY SERVERS WITH MOBAXTERM.*****

/// install HA Proxy ( on all haroxy nodes )

 # yum install haproxy
 
 -------------------------------
 
 /// install keepalived  ( on all haroxy nodes )

 # yum install keepalived
 
 -------------------------------
 /// configure HA Proxy ( acconding to other ha proxy nodes )
 
  #vim /etc/haroxy/haproxy.cfg
  
- default
    mode            http >>>> tcp
     
- frontend main ( remove all lines execp blow lines )
    bind *:5000 >>> *:6443
    default_backend app  >>> kubernetes
 
 
- backend     app >>> kubernetes  ( change information of servers according your servers )
    balance roundrobin
    server server1 opkbm...0101:6443 check
    server server2 opkbm...0103:6443 check
    server server3 opkbm...0105:6443 check
  
  -------------------------------
  
  # systemctl  enable --now hrproxy.serive
   
 --------------------------------
 
  /// configure keepalived ( acconding to other ha proxy nodes )
  
    #vim /etc/keepalived/keepalived.conf
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
