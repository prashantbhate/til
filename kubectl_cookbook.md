
# Node Pod Container names


```
#list all nodes with all pods all container

kubectl get pods --all-namespaces -o=custom-columns=NODE:.spec.nodeName,POD:metadata.name,CONTAINER:.spec.containers[*].name

#same with json output
kubectl get pods --all-namespaces -o=json|jq '[.items[]|{node: .spec.nodeName , pod:.metadata.name, container:.spec.containers[].name }]|group_by(.node)|map({(.[0].node):(group_by(.pod)|map({(.[0].pod):map(.container) }) )})'

```

# IP Addresses

```
#list internal and external ip addresses of node
gcloud compute instances list

# cluster/master ip address
kubectl cluster-info

# node & pod IP addresses
kubectl get pods --all-namespaces -o=custom-columns=NODE:.spec.nodeName,NODE_IP:status.hostIP,POD:metadata.name,POD_IP:status.podIP --sort-by=.spec.nodeName


```


# Complete details

```
#dumps complete details into a directory 
kubectl cluster-info dump --all-namespaces --output-directory=kudump
```


# Wait for pods to be terminated

```
until [ $(kubectl get po|grep Terminating|wc -l ) -eq 0 ] ;do \
	echo "Wait for pods to be terminated:"; \
	kubectl get po | grep Terminating; \
	sleep 1; \
	echo .....; \
	done
```

# Wait for pods to be Ready

```
while [ $(kubectl get po --no-headers | grep -v -E '(\d+)/\1' | wc -l ) -ne 0 ] ;do \
	echo "Wait for pods to be ready:"; \
	kubectl get po | grep -vE '(\d+)/\1'; \
	sleep 1; \
	echo .....; \
	done
```

# Get open ports without netstat

```
awk 'function hextodec(str,ret,n,i,k,c){
    ret = 0
    n = length(str)
    for (i = 1; i <= n; i++) {
        c = tolower(substr(str, i, 1))
        k = index("123456789abcdef", c)
        ret = ret * 16 + k
    }
    return ret
}
function getIP(str,ret){
    ret=hextodec(substr(str,index(str,":")-2,2)); 
    for (i=5; i>0; i-=2) {
        ret = ret"."hextodec(substr(str,i,2))
    }
    ret = ret":"hextodec(substr(str,index(str,":")+1,4))
    return ret
} 
NR > 1 {{if(NR==2)print "Local - Remote";local=getIP($2);remote=getIP($3)}{print local" - "remote}}' /proc/net/tcp 
```
