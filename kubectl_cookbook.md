

```
#list all nodes with all pods all container

kubectl get pods --all-namespaces -o=custom-columns=NODE:.spec.nodeName,POD:metadata.name,CONTAINER:.spec.containers[*].name

#same with json output
kubectl get pods --all-namespaces -o=json|jq '[.items[]|{node: .spec.nodeName , pod:.metadata.name, container:.spec.containers[].name }]|group_by(.node)|map({(.[0].node):(group_by(.pod)|map({(.[0].pod):map(.container) }) )})'

```
