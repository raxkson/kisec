# Docker Registry - docker hub for local


## Install Docker-Registry
 
First, install docker-registry with helm
```bash
helm install docker-registry stable/docker-registry -n registry --set service.nodePort=30500,service.type=NodePort
```

Second, add the address where you want to login /etc/hosts
```bash
sudo echo "192.168.55.2 mydockers" >> /etc/hosts
```

I do not use SSL because it works only in my local, so we have to config insecure-registries (if you do not have daemon.json, just create it!)
YOU SHOULD ADD THIS TO ALL NODES
```bash
vi /etc/docker/daemon.json 
echo { "insecure-registries":["192.168.55.2:30500", "mydocker:30500"] } >> /etc/docker/daemon.json // if you have file, add it
```


Before you login, create ingress
```bash
kubectl apply -f ingress.yaml
```

Finally, you can upload your docker image...
```bash
docker login mydocker:30500
docker tag my-img mydocker:30500/my-img
docker push mydocker:30500/my-img
```