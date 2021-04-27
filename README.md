# kisec  


## When kubernetes is not work after reboot
1. Check swap off
```bash
sudo -i
swapoff -a
strace -eopenat kubectl version
```

2. Remove all dockers
```bash
sudo docker stop $(sudo docker ps -a -q); sudo docker rm $(sudo docker ps -a -q)
```

3. Check docker is work ok
```bash
sudo dockerd --debug
```
