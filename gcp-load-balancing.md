GCP (Google Cloud Platform) loadbalancing options are overwhelming, and a bit confusing. 

Here is my cheatsheet to burst jargons and decide which option to choose  
  
  
1. **HTTP(S) Global load balancing** - proximity based, content based, or both - Level 7 LB
1. TCP Loadbalancing
    1. **Multi Regional / Global load balancing**
        1. **SSL Proxy** - proximity based - terminate SSL sessions - Level 3 LB
        1. **TCP Proxy** - proximity based - Level 3 LB
    1. **Regional** 
        1. **External** Between instances spanning within same region across zones (does not SSL/TCP Proxy)
        1. **Internal** Between instances spanning within same region across zones
1. UDP load balancing


Each load-balancer will have one or more frontend through wich request comes in and backend to which requiest is routed.

Below are key ingredients required to cook GCP load balancer !

|  what | used in | Desc |
| -------------| ------------- | ------------- |
| FrontEnds | all |  one or more Host+ports that  client uses to connect |
| Host and path rules |HTTP(S) Global load balancing | used by HTTP(S) load balancer to route request based on path rules |
|Backend services| all  external | one or more backend services each backend service contains one or more instance groups with load balance configurations like health check  |
|Regional Backend services| Regional Internal TCP lb |contains one or more backend services from same region each backend service contains one or more instance groups with load balance configurations like health check, |
|Target pool| TCP LB |Target pool is a kind of backend used by tcp forwarding rules|
|global forwarding rule| HTTP(S) Global load balancing | forwards External IP:port to a target-proxy|
|target proxy| HTTP(S) Global load balancing |A target receives traffic from a forwarding rule and points to a URL map|
|url map|HTTP(S) Global load balancing | url map contains rules that are used by content based loadbal to route requests|
