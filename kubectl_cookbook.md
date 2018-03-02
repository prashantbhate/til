

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
