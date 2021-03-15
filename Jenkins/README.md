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

Get value from stable/jenkins (You must edit some values!!!)
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

### When master node give error with liveness & readiness (In worker node)
```bash
sudo systemctl restart kubelet.service
```
Also something worng
```bash
sudo docker stop $(sudo docker ps -a -q); sudo docker rm $(sudo docker ps -a -q)
```

There are so many errors... Good luck.. If you need help contact to raxkson@gmail.com
