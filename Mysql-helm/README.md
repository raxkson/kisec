# Mysql Helm

1. Set persistent volume for mysql

In worker node
```bash
mkdir mysql-volmue
chown -R 1000:1000 mysql-volmue
```

In master node
```bash
kubectl apply -f mysql-volume.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  labels:
          type: local
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /home/kisec/mysql-volume/
```

2. Helm install mysql

Before you install mysql you should fix some values in mysql-values.yaml
1. Same capacity with persistent volume capacity if it's diffrent it doesn't work
2. Fix Service type as NodePort
3. Fix nodePort as 30000 - 350000(?)
4. Fix mysqlRootPassword, mysqlUser, mysqlPassword, mysqlDatabase

```bash
helm inspect values stable/mysql > mysql-values.yaml
helm install mysql -f mysql-value.yaml stable/mysql
```