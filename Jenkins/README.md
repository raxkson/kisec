# Jenkins

## Install jenkins with helm
Get jenkins-volume.yaml for create volume on worker node
```bash
wget https://raw.githubusercontent.com/jenkins-infra/jenkins.io/master/content/doc/tutorials/kubernetes/installing-jenkins-on-kubernetes/jenkins-volume.yaml
```

Give permission to volume on woker node (if no permission master cannot pull docker containers)
```bash
mkdir jenkins-volume
sudo chown -R 1000:1000 jenkins-volume
```

Get value from stable/jenkins (You must edit some values!!! I put the comment in jenkins-value.yaml `#edit by raxkson`)
- admin password
- nodeport
- liveness time
- readiness time
```bash
helm inspect values stable/jenkins > jenkins-value.yaml
```

Install Jenkins with Helm
```bash
helm install jenkins -f jenkins-value.yaml stable/jenkins
```

For Jenkins UnkownHostException...(This make me crazy)
First, you have to login jenkins docker with root
```bash
docker exec -u 0 -it <Container ID> bash
```
Second, you should be add nameserver 8.8.8.8 /etc/resolv.conf, but we do not have editor and put with redirection

Then you can finally use plugin...!
```bash
echo 'nameserver 8.8.8.8' >> /etc/resolv.conf
```



### When master node give error with liveness & readiness (In worker node)
```bash
sudo systemctl restart kubelet.service
```
Also something wrong in master or worker
```bash
sudo docker stop $(sudo docker ps -a -q); sudo docker rm $(sudo docker ps -a -q)
```
Only for master
```bash
kubectl rollout restart deployment jenkins 
```

when you want to use git with jenkins give this command to workernode (you must use this on user not root!)
```bash
git config --global --unset http.proxy
```


Last thing is... when you restart jenkins-docker you should do this thing again
```bash
docker exec -u 0 -it <Container ID> bash
echo 'nameserver 8.8.8.8' >> /etc/resolv.conf
docker exec -it <Container ID> bash
git config --global --unset http.proxy
```

When you want to use kubernetes plugin you have to apply jankins-sa.yaml
```bash
kubectl apply -f jenkins-sa.yaml
```


There are so many errors... Good luck.. If you need help contact to raxkson@gmail.com
